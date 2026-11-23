# Description
<!-- 请简要说明此 PR 的变更目的和核心逻辑。 -->

## Type of Change
<!-- 请勾选适用的选项。 -->
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update
- [ ] Architecture/Refactor

## Impact Analysis
<!-- 工业化系统的核心是评估影响面。请勾选并说明。 -->
- [ ] **Manifest Change**: 是否修改了 `plugin.toml` 的 Schema？（如有，说明向后兼容策略）
- [ ] **IPC Protocol Change**: 是否修改了数据面/控制面的帧格式或 RPC 方法？
- [ ] **Sandbox Policy Change**: 是否修改了插件的权限映射规则？
- [ ] **Database Schema Change**: 是否涉及宿主的 SQLite 或插件存储表结构的变更？

## Checklist

- [ ] I have performed a self-review of my own code.
- [ ] I have added tests that prove my fix is effective or that my feature works.
- [ ] New and existing unit tests pass locally with my changes.
- [ ] I have run `mypy --strict` and `ruff check` locally.
- [ ] I have updated the `developer_guide.md` if the SDK API has changed.
