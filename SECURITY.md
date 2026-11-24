# Security Policy

## Supported Versions

Plugin OS 属于前沿工业级架构，我们仅支持最新的 Minor 版本和当前的大版本分支。

| Version | Supported          |
| ------- | ------------------ |
| 1.x     | :white_check_mark: |
| < 1.0   | :x:                |

## Reporting a Vulnerability

**请勿在公共 Issue 区讨论安全漏洞。**

如果您发现了以下类型的安全漏洞，请立即通过邮件发送至 `14h-18iw2a4d1r@dingtalk.com`：

1. **沙箱逃逸 (Sandbox Escape)**：插件子进程突破 AppContainer / bwrap / Seatbelt 限制，访问未授权的宿主文件或网络。
2. **越权 RPC 调用 (Unauthorized RPC)**：插件绕过了 `is_call_allowed` 鉴权，调用了未在 Manifest 中声明的其他插件能力。
3. **IPC 拒绝服务攻击 (DoS)**：发送超大帧、半包攻击导致宿主 OOM 或事件循环阻塞。
4. **路径穿越 (Path Traversal)**：通过恶意的 `plugin_id` 或存储 Key 读取宿主任意目录。

我们会在 2d~4d 内确认漏洞并给出修复计划。确认后，我们将公开致谢。
