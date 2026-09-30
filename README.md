# dsh-web-launcher

Windows 下 DSH Web 的启动 / 停止脚本，双击即用。

## 技术选型

| 维度 | 选型 |
| --- | --- |
| 脚本 | VBScript（.vbs，双击运行） |
| 进程识别 | WMI 查 node.exe 命令行，按 `bin.js` + `" web"` 精准匹配 |
| 停止方式 | `taskkill /F /T`（连同子进程） |

## 核心流程

### 启动（start-dsh-web.vbs）

1. 读配置：node、入口 `apps/cli/lib/bin.js`、工作目录、日志 `dsh-web.log`、地址 `127.0.0.1:3080`
2. 用 WMI 查 node.exe 命令行，看是否已含 `bin.js` + `" web"`
3. 已在跑 → 弹窗「无需重复启动」，退出
4. 没在跑 → 拼命令 `cmd /c node "bin.js" web >> 日志 2>&1`
5. 切到工作目录，无窗口后台启动（`Run` 参数 0, False）
6. 轮询 20 次 × 1 秒，探测进程是否起来
7. 起来了 → 弹「启动成功 + 访问地址 + 日志路径」
8. 没起来 → 弹「启动失败，看日志」

### 停止（stop-dsh-web.vbs）

1. 用 WMI 查 node.exe，挑出命令行含 `bin.js` + `" web"` 的所有 PID
2. 没有 → 弹「没有正在运行的 DSH Web」
3. 有 → 逐个 `taskkill /F /T /PID`（连同子进程）
4. 睡 1.5 秒，复查是否还有残留
5. 0 残留 → 弹「已停止 N 个进程」
6. 有残留 → 弹「权限不足，建议管理员运行」

## 关键设计点

- **按命令行精准识别进程**：只看命令行含 `bin.js` + `" web"` 的 node，不影响其他 node 程序。
- **无窗口后台启动**：`Run` 参数 0, False，不弹黑窗。
- **日志落盘**：输出重定向到脚本同目录 `dsh-web.log`。
- **启动自检**：启动后轮询 20 秒探测，成功 / 失败都弹窗告知。
- **停止连坐 + 复查**：`taskkill /F /T` 连同子进程，1.5 秒后复查残留，有残留提示管理员运行。

## 文件清单

| 文件 | 作用 |
| --- | --- |
| start-dsh-web.vbs | 启动（双击） |
| stop-dsh-web.vbs | 停止（双击） |
| dsh-web.log | 运行日志 |
