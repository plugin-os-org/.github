# Contributing to Plugin OS

作为开源/内源项目，我们欢迎一切形式的贡献。但由于本系统对稳定性和安全性要求极高，请遵守以下准则：

## 开发规范

1. **类型检查**：所有核心模块必须通过 `mypy --strict`。禁止滥用 `Any` 类型。
2. **日志规范**：禁止在插件代码中使用 `print()`。请使用 `ctx.get_service("logger").info()`。
3. **无阻塞**：禁止在宿主进程内写同步阻塞代码。所有 IO 操作必须使用 `asyncio`。
4. **测试覆盖**：新功能必须包含测试用例。涉及 IPC 协议修改的，必须包含混沌测试用例。

## 分支与提交

* 采用 GitHub Flow。所有修改需提交 PR 到 `main` 分支。
* 提交信息请遵循 [Conventional Commits](https://www.conventionalcommits.org/) 规范（如 `feat: add telemetry heatmap`）。

## 安全底线

在 PR 中请明确声明：您的修改是否涉及沙箱配置、IPC 协议或 Manifest 校验。未声明且破坏安全边界的 PR 将被直接关闭。
