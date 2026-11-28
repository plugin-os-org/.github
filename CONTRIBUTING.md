# Contributing to Plugin OS

欢迎参与 Plugin OS 的开发！为了保证这个微内核系统的工业化质量，所有代码提交必须严格遵守以下规范和自动化校验机制。请仔细阅读本文档，任何不符合规范的 PR 将被 CI 流水线自动拒绝。

## 一、 分支与提交规范 (Git & Commits)

我们采用 **GitHub Flow** 分支模型，所有修改必须通过 Pull Request 合并到 `main` 分支。

### 1. 基础提交格式

提交信息必须遵循 **Conventional Commits** 规范。格式如下：

```text
<type>(<scope>): <subject>

[optional body]

[optional AUDIT footer]
```

* **type**: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`。
* **scope**: 模块名（如 `sdk`, `transport`, `manager`, `storage`, `web_ui`）。
* **subject**: 简明扼要的变更描述（首字母小写，不加句号）。

### 2. AUDIT 审计强校验

凡涉及以下核心领域的变更，**必须**在 Commit Message 结尾附加 `AUDIT:` footer，否则 CI 的 Commit Lint 将直接报错阻断 PR：

1. IPC 协议 / 帧格式变更。
2. 沙箱策略 / 权限映射变更。
3. 数据库 Schema 变更。
4. 破坏性 API 变更。

**AUDIT Footer 格式**：

```text
AUDIT: reason=<reason>; reviewer=<name>; impact=<impact_area>; ticket=<ticket_id>
```

**示例**：

```text
feat(transport): support msgpack serialization

Added content-type flag in frame header to allow msgpack instead of JSON.

AUDIT: reason=performance; reviewer=alice; impact=ipc-protocol; ticket=PERF-1024
```

### 3. 禁止事项

* 禁止在代码中遗留 `# TODO`, `# FIXME`, `# AUDIT` 等注释。如有待办事项，请使用 GitHub Issues 或项目看板。
* 禁止包含无意义的 commit（如 `update`, `fix bug`）。

## 二、 代码规范与类型安全 (Code Quality)

### 1. 类型安全 (`mypy --strict`)

Plugin OS 的内核（SDK、Transport、Manager）强制要求 `mypy --strict` 零报错。

* 禁止滥用 `Any` 类型。
* 若遇到第三方库（如 `asdict`）类型推断失败的问题，请使用带类型的局部变量进行中转，**禁止使用 `# type: ignore` 来掩耳盗铃**。

### 2. 异常处理铁律

**禁止静默吞噬任何异常**。严禁使用 `except Exception: pass` 或捕获 `BaseException`。

* **精准捕获**：必须捕获具体的异常类型（如 `OSError`）。
* **降级处理**：如果该异常不影响主流程，必须记录日志（或写入降级提示），并在代码中显式说明降级逻辑。

```python
# 正确示例
try:
    with open(log_file, "a") as f:
        f.write(data)
except OSError as e:
    logger.error(f"Failed to write to log file: {e}")
    # 降级逻辑：写入内存队列
```

### 3. 安全与隔离红线

* **禁止污染标准输出**：插件子进程的 `stdout` 专用于输出 JSON Lines 日志流。严禁在插件代码中使用原生的 `print()`。
* **禁止子进程逃逸**：插件代码（除 TUI 外）不允许使用 `subprocess.Popen` 或 `os.system` 创建新进程。
* **TUI 插件特例**：TUI 插件可通过 `--tui-mode` 接管终端，但必须使用 SDK 提供的 `TuiLocalLogger`，禁止破坏 Textual 的 UI 渲染。

## 三、 文档与注释规范 (Documentation)

Plugin OS 使用 **Doxygen** 生成 API 文档。所有公开的模块、类和函数必须使用 **Google 风格 Docstring**。CI 中开启了 `WARN_AS_ERROR=YES`，任何注释缺失或格式错误都将导致构建失败。

> ⚠️ **风格铁律**：绝对禁止混用 Google 风格（如 `Args:`）与 Doxygen 原生指令（如 `\param`、`\return`）。统一采用 Google 风格，由 `doxypypy` 过滤器自动完成转换。

### 1. 模块级注释

在文件开头使用 `@file` 和 `@brief` 描述模块职责。

```python
"""!
@file ctx.py
@brief 定义了 SDK 向插件端暴露的协议接口。
@details 包含插件上下文、能力代理等定义。
"""
```

### 2. 函数与方法级注释

必须包含：简述、参数 (`Args`)、返回值 (`Returns`)、可能抛出的异常 (`Raises`)。参数名称必须与函数签名完全一致。即使返回 `None`，也建议显式声明。

```python
async def resolve_capability(
    self, capability: str, domain: str | None = None
) -> CapabilityProxy | None:
    """解析能力名称，返回对应的代理对象。

    Args:
        capability (str): 能力名称。
        domain (str | None, optional): 负责该能力的插件名称。Defaults to None.

    Returns:
        CapabilityProxy | None: 对应的能力代理对象或 None。

    Raises:
        AmbiguousCapability: 当省略 domain 且匹配到多个提供者时抛出。
    """
```

### 3. 类与变量注释

类级别的注释使用 `Attributes:` 描述公开属性。

```python
class LRUCache:
    """基于 OrderedDict 实现的异步安全 LRU 缓存。

    Attributes:
        capacity (int): 缓存的最大容量。
    """
```

## 四、 CI 自动化流水线 (CI Pipeline)

你的 PR 提交后，GitHub Actions 会执行以下强校验：

1. **Commit Lint**：检查 Conventional Commits 格式，并拦截未按要求附带 `AUDIT:` footer 的核心变更。
2. **Mypy Strict**：对所有核心模块进行全量类型检查。
3. **Doxygen 构建**：强校验所有公开 API 的注释规范。
4. **Pytest**：运行全量单元测试与混沌工程测试用例。

感谢你为维护 Plugin OS 的工业级标准所做的贡献！
