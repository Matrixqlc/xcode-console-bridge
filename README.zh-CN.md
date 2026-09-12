# Xcode Console Bridge

[English](README.md)

将 Xcode Debug Console（调试控制台）当前显示的输出，转成 AI 编码代理可安全读取的本地证据包。

Xcode 很适合展示运行时输出，但没有提供用于读取已填充 Debug Console 的公开 API（应用程序接口）。这让 macOS 上的 iOS 开发流程缺了一环：AI 可以阅读源代码、修改代码，但开发者复现问题后仍要手工导出运行时证据。

Xcode Console Bridge 补上这条链，同时不会把“剪贴板里有内容”误判为“已经读到了控制台”。

## 它做什么

1. 激活一个由你明确匹配的 Xcode 窗口。
2. 调用 Xcode 的 Activate Console（激活控制台）快捷键。
3. 通过 macOS 辅助功能确认焦点元素确实是 `Console`。
4. 复制控制台文本，恢复此前的剪贴板与前台 App，并拒绝 HTML（网页内容）或不可信的载荷。
5. 写入本地证据包：

```text
xcode-console-evidence/
  <run-label>/<timestamp>/
    xcode-console.raw.log       # 完整的本地事实源
    xcode-console.digest.md     # 为 AI 限定体积的导航摘要
    xcode-console.metrics.tsv   # 可比较的轻量计数
    xcode-console.compare.md    # 与可选基线的对比
  latest -> <run-label>/<timestamp>
```

它不会远程读取设备、启动构建、运行 App、上传日志或持续监听控制台。它只在你明确调用时，对 Xcode 当前展示内容做一次本地快照。

## 前提条件

- macOS
- 正在运行的 Xcode，且 Debug Console（调试控制台）可用
- `bash`、`osascript`、`pbcopy`、`pbpaste` 与 [ripgrep](https://github.com/BurntSushi/ripgrep)
- 执行命令的终端或 AI 代理宿主已取得 macOS 辅助功能权限

抓取日志不需要网络连接。

## 安装

克隆仓库后直接执行：

```bash
git clone https://github.com/Matrixqlc/xcode-console-bridge.git
cd xcode-console-bridge
./bin/xcode-console-bridge --help
```

## 抓取一次运行证据

窗口匹配词必须足够精确，只命中一个已打开的 Xcode 窗口。尽量加入一个预期会出现在自家 App 日志中的身份词，它能加强对剪贴板内容的校验。

```bash
./bin/xcode-console-bridge \
  --window-token MyApp \
  --app-name MyApp \
  --identity-token MyApp \
  --run compression-regression
```

不确定该使用什么窗口匹配词时：

```bash
./bin/xcode-console-bridge --list-windows
```

匹配到零个或多个窗口时，工具会停止；它不会猜测你想读取哪个工程。

### 已知正常基线

对一个已确认正常的运行增加 `--set-baseline`。同一证据根目录之后的抓取会生成简要的指标对比。

```bash
./bin/xcode-console-bridge --window-token MyApp --run known-good --set-baseline
```

### 手动复制兜底

若没有辅助功能权限，或 Xcode 更新后改变了控制台的辅助功能树，先手动复制 Console（控制台）文本，再把它规范化为同样的证据格式：

```bash
./bin/xcode-console-bridge --import-clipboard --run manual-copy
```

若原始日志已经保存到文件：

```bash
./bin/xcode-console-bridge --raw-log /path/to/xcode-console.raw.log --run imported-log
```

## 交给 AI 编码代理分析

让 AI 先阅读摘要和对比报告；只有需要证实具体结论时，才针对原始日志做局部、限行的检索。不要把整个原始日志直接粘贴进对话。

```text
先读取 xcode-console-evidence/latest/xcode-console.digest.md。
再检查 xcode-console-evidence/latest/xcode-console.compare.md。
把 xcode-console.raw.log 视为事实源；诊断前使用有针对性、限定行数的检索确认每个运行时结论。
```

可复用的 AI 指令见 [AI integration guidance](docs/AI-INTEGRATION.md)。

## 安全与隐私

- 证据默认保留在本机；除非你主动分享，否则不会上传。
- 控制台内容可能含有路径、标识符、请求数据或其他敏感信息。上传到 issue（问题）或发送给托管模型前，请先审阅与脱敏。
- 自动抓取会恢复其替换掉的原剪贴板内容。
- 自动抓取由 12 秒进程看门狗限制；AppleScript（苹果脚本）卡住时会被终止，不会遗留后台进程。
- 复制成功本身不算成功：工具必须验证 Console 焦点，并要求载荷具有控制台形态或匹配指定身份词。
- 第一版不自动清理历史证据；是否以及何时删除由你决定。

## 限制

这是一座跨越 Xcode 控制台导出 API 缺口的实用桥接工具，不是受支持的 Xcode 自动化 API。Xcode 或 macOS 辅助功能变化后可能需要适配；当工具无法证明焦点元素是 `Console` 时，会选择失败而不是冒险抓取。

`os_log` 与 macOS `log` 命令是互补的系统日志工具；它们不能替代“复制当前 Xcode Debug Console 显示状态”这一需求。后者可能包含当前复现过程里重要的调试器与进程输出。

## 许可证

[MIT](LICENSE)
