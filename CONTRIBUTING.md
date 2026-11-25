# Contributing to Plugin OS

> 由 Deepseek 生成

欢迎参与 Plugin OS 的开发！为了保证这个微内核系统的工业化质量，所有代码提交必须严格遵守以下规范和自动化校验机制。

## 一、 代码注释规范 (Doxygen + Python Docstring)

Plugin OS 使用 **Doxygen** 自动生成 API 文档。我们拒绝在 Python 中使用 C 风格的 `/** */` 注释，也**禁止在代码中写任何审计日志或变更说明**（这属于 Git 的职责）。

所有对外暴露的模块、类、函数和方法，必须使用 **Google 风格** 的 Python Docstring。Doxygen 将直接解析这些 Docstring 并提取类型提示。

### 1. 模块级注释

在文件开头必须包含 `@file` 标签和模块简述。

```python
"""!
@file manager.py
@brief 负责宿主生命周期管理、子进程调度与心跳探活。
@details 包含 WatchDog 和 EdgeScheduler 的核心实现。
"""
```

### 2. 函数与方法级注释

必须包含：简述、参数 (`Args`)、返回值 (`Returns`)、可能抛出的异常 (`Raises`)。

```python
def resolve_capability(
    capability: str, 
    domain: str | None = None
) -> CapabilityProxy | None:
    """根据能力标签解析远程代理。

    Args:
        capability: 能力名称，如 "verify"。
        domain: 可选的能力域，用于解决跨域同名能力的歧义。

    Returns:
        解析成功时返回 CapabilityProxy，否则返回 None。

    Raises:
        AmbiguousCapability: 当省略 domain 且匹配到多个提供者时抛出。
    """
    ...
```

### 3. 类级注释

```python
class LRUCache:
    """基于 OrderedDict 实现的异步安全 LRU 缓存。

    Attributes:
        capacity (int): 缓存的最大容量。
    """
```

## 二、 审计与提交规范 (Commit Message)

我们使用 Git Commit 替代代码内的审计日志。所有提交信息必须遵循 **Conventional Commits** 规范，并在此基础上扩展 **`AUDIT:`** footer。

### 1. 基础格式

``` markdown
<type>(<scope>): <subject>

[optional body]

[optional AUDIT footer]
```

* **type**: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`。
* **scope**: 模块名（如 `sdk`, `transport`, `manager`, `storage`, `web_ui`）。
* **subject**: 简明扼要的变更描述。

### 2. AUDIT Footer (强制)

凡涉及以下核心领域的变更，**必须**在 Commit Message 结尾附加 `AUDIT:` 说明，否则 CI 将拒绝合并：

1. IPC 协议 / 帧格式变更。
2. 沙箱策略 / 权限映射变更。
3. 数据库 Schema 变更。
4. 破坏性 API 变更。

**格式**：

``` markdown
AUDIT: reason=<reason>; reviewer=<name>; impact=<impact_area>; ticket=<ticket_id>
```

**示例**：

``` markdown
feat(transport): support msgpack serialization

在帧头中添加了内容类型标志，以允许使用 msgpack 而不是 JSON。

AUDIT: reason=performance; reviewer=alice; impact=ipc-protocol; ticket=PERF-1024
```

### 3. 禁止事项

* 禁止在代码中写 `# TODO`, `# FIXME`, `# AUDIT` 等注释。如有待办事项，请使用 GitHub Issues 或项目看板。
* 禁止包含无意义的 commit（如 `update`, `fix bug`）。

## 三、 CI 自动化校验

你的 PR 提交后，GitHub Actions 会执行以下强校验：

1. **Docstring 校验**：运行 Doxygen 扫描，如果发现公开函数缺少参数说明，将报错并阻断合并。
2. **Commit Lint**：检查是否遵循了 `type(scope): subject` 格式。如果触及核心目录（如 `src/plugin_os_transport/` 或 `src/plugin_os_manager/sandbox/`），必须包含合法的 `AUDIT:` footer。
3. **Mypy Strict**：全量类型检查。

感谢你为维护 Plugin OS 的工业级标准所做的贡献！
