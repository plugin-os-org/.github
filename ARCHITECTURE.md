# ARCHITECTURE — Plugin OS 系统架构

- 分支：`main`
- 日期：2026-11-25

本文档描述 Plugin OS 的物理仓库拓扑、运行时分层、核心组件交互与数据流向。

## 一、 仓库拓扑 (1 + X + Y + Z)

为了保持代码的纯净度，系统按职责拆分为多个独立仓库：

- **组织级规约 (1)**: `.github` (存放 CI 模板、Issues 模板、Doxygen 规范)
- **基础内核**:
  - `plugin-os-sdk` (契约中心)
  - `plugin-os-transport` (通信基座)
  - `plugin-os-manager` (调度与沙箱)
- **内部模块 (X)**: `plugin-os-internal-*` (Storage, Audit, Telemetry, Lifecycle, Store, CLI)
- **业务插件 (Y / Z)**: `plugin-os-template-*` (Core / Sub / Internal 脚手架)
- **控制台**: `web_ui` (Vue 3 前端)

## 二、 运行时分层架构

```text
[前端 Vue 3] <---(HTTP/SSE)---> [宿主 FastAPI 网关]
                                      |
      +-------------------------------+-------------------------------+
      |                               |                               |
[Internal CLI]           [Internal Telemetry/Audit]       [Internal Lifecycle/Store]
      |                               |                               |
      +-------------------------------+-------------------------------+
                                      | (宿主内存注入)
                           [Host Manager 引擎]
                           - EdgeScheduler (DAG 拓扑)
                           - WatchDog (生命周期)
                           - Gateway (动态 OpenAPI 路由)
                                      |
                           [Transport 基座]
                           - JSON-RPC 多路复用
                           - 取消帧 & 心跳探活
                                      |
 +---------------------------------+---------------------------------+
 |                                 |                                 |
[core_gui]  [gateway_core]   [auth_sub] [data_sub]    [core_tui] [chaos_sub]
 |               |                 |          |             |          |
 +----(控制面: stdin/stderr / 数据面: TCP Socket / 日志面: stdout)---+----------+
                                      |
                                [OS 级沙箱]
                     (AppContainer / bwrap / Seatbelt)
```

## 三、 核心组件交互机制

### 3.1 启动与 DAG 调度

1. `WatchDog` 扫描 `plugins/` 目录，解析 `plugin.toml`。
2. `EdgeScheduler` 构建依赖 DAG，Kahn 算法检测循环依赖。
3. 计算关键路径长度，使用并发优先级队列拉起子进程。
4. 启动期间若因依赖失败，自动级联标记下游节点为 `ERROR`。

### 3.2 握手与就绪判定

1. 子进程启动后，通过 TCP 连上宿主的 `TransportManager`。
2. 子进程发送首帧 `handshake`（携带 OpenAPI Schema）。
3. 同时子进程向 stderr 发送 `ready` 控制帧。
4. 宿主确认双重就绪后，通过 `Gateway` 将 OpenAPI 动态挂载至 FastAPI 路由。

### 3.3 路由与 RPC 拦截

- 子进程向宿主发起的 RPC 调用由 `TransportManager` 拦截分发：
  - `storage.*`：转发给 `PluginStoragePool` 获取对应的 Facade 执行。
  - `internal.*`：转发给其他 Internal 服务。
  - `audit.report`：转发给审计中心。
  - 业务调用：检查源插件的 Manifest 是否声明了对应的硬依赖或能力，鉴权通过后转发给目标子进程。

## 四、 高可用与热更新 (Blue-Green)

1. **状态自愈**：通过心跳（`ping/pong`）与 `process.wait()` 捕获崩溃。指数退避（1s/2s/4s/8s）重启，稳定运行 60s 后清零计数。
2. **蓝绿切流**：
   - `WatchDog` 拉起目标插件的第二个槽位（如 `green`）。
   - 新槽位握手成功并挂载路由后，原子修改 `TransportManager.routing_table`。
   - 等待存量请求处理完毕，销毁旧槽位（`blue`）。
   - 整个过程借助 `psutil` 进行资源监控。

## 五、 数据流向与安全审计

- **日志流向**：插件 `print()` -> 被 Runner 毒化拦截 -> 插件 `ctx.get_service("logger").info()` -> 子进程 stdout JSON Lines -> 宿主 WatchDog 解析 -> `AuditManager.ingest_log()` -> 分发给本地 loguru 与 SSE 推流前端。
- **权限审计**：插件越权访问文件 -> SDK 抛出 `PluginPermissionError` -> 子进程发出 RPC `audit.report` -> 宿主 `DataChannel` 拦截 -> `AuditManager.record_violation()` -> 前端生成红色告警。

## 六、 扩展性设计

- **配置热重载**：宿主的 `LifecycleManager` 校验 JSON Schema 后，向子进程发送 `reload_config` 帧。
- **动态日志级别**：宿主向子进程发送 `log_level` 帧，子进程内的 `StdoutLogger` 调整内部阈值。
- **联邦生态**：`StoreManager` 支持多源聚合，后端静默同步，支持 Git Clone 与 PyPI 双模安装。
