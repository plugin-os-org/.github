# Plugin OS

**A Industrial-Grade, Microkernel-based Plugin Operating System.**

Plugin OS 是一个基于 Python 与 FastAPI 构建的工业化微内核插件平台。我们将业务逻辑与基础设施彻底解耦，通过严格的进程隔离、OS级沙箱、DAG依赖调度与联邦生态，为不可信代码（如 AI 生成的 Python 脚本）提供安全的执行环境。

## 核心架构

* **Microkernel**：内核仅负责通信、调度与生命周期，不包含任何业务代码。
* **Strict Isolation**：插件运行在独立子进程，受 Windows AppContainer / Linux bwrap / macOS Seatbelt 保护。
* **High Performance IPC**：基于 JSON-RPC 2.0 的多路复用长连接，毫秒级通信延迟。
* **Blue-Green Deployment**：支持插件级别的无缝热更新与流量切换。

## 核心仓库矩阵

* `plugin-os-sdk`：契约中心（Pydantic 模型与插件上下）。
* `plugin-os-transport`：通信基座（帧解析与数据面）。
* `plugin-os-manager`：宿主内核（调度、WatchDog、Gateway）。
* `plugin-os-template-*`：插件开发脚手架。

**警告**：请勿在生产环境中使用尚未通过安全审计的第三方源插件。

## 架构文档

* [DESIGN.md (架构设计纲领)](../DESIGN.md)
* [ARCHITECTURE.md (系统架构详解)](../ARCHITECTURE.md)
