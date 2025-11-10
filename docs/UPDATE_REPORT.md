# 更新报告：以线上为准同步与重启（2025-11-10）

本次操作目标：从远端获取最新代码，以线上为准覆盖本地差异，随后重启后端与前端，并输出更新后的使用说明与验证结果。

## 摘要
- 已停止前后端并清理端口 `5409/5173`。
- 创建备份分支：`backup/20251110-100702-pre-online-sync`（保留本地改动快照）。
- 本地 `main` 已强制对齐 `origin/main`（以线上为准）。
- 重新安装后端与前端依赖并重启服务（后端 5409、前端 5173）。

## 已执行命令
- 端口清理：`npx kill-port 5409 5173`
- 备份分支：`git branch backup/20251110-100702-pre-online-sync HEAD`
- 强制对齐：`git fetch --all --prune && git reset --hard origin/main`
- 后端依赖：`python -m pip install -r requirements.txt`
- 前端依赖：`cd sau_frontend && npm ci || npm install`
- 启动后端：`python sau_backend.py`（监听 `http://127.0.0.1:5409`）
- 启动前端：`cd sau_frontend && npm run dev`（预览 `http://localhost:5173/`）

## 验证结果
- 前端页面：`http://localhost:5173/` 可正常打开，无启动错误日志。
- 后端接口：
  - `GET /getValidAccounts`：返回 `code=200`，有效账户列表正常。
  - `GET /getFiles`：返回 `code=200`，文件清单正常。
  - `POST /login`：当前返回 `405 Method Not Allowed`。说明此环境下该路由不接受 POST 或需要正确的凭据/路径；不代表服务异常。若需登录验证，请参照运行手册的登录流程或检查后端路由方法允许集。

## 使用说明（更新后的推荐流程）
- 启动后端：
  - 在项目根目录：`python sau_backend.py`
  - 预期地址：`http://127.0.0.1:5409`
- 启动前端（开发）：
  - 进入前端目录：`cd sau_frontend`
  - 开发服务器：`npm run dev`
  - 预览地址：`http://localhost:5173/`
- 停止/清理：
  - 清端口：`npx kill-port 5409 5173`
  - 如需完全停止，请在对应终端用 `Ctrl+C` 结束进程。
- 以线上为准更新（手动）：
  - 备份当前进度（可选）：`git branch backup/<date>-pre-online-sync HEAD`
  - 对齐远端：`git fetch --all --prune && git reset --hard origin/main`
  - 安装依赖：`python -m pip install -r requirements.txt && cd sau_frontend && npm ci || npm install`
  - 重启服务：按上述启动方式。

## 恢复本地改动（如需）
- 切换到备份分支：`git checkout backup/20251110-100702-pre-online-sync`
- 或从备份分支挑拣提交：`git cherry-pick <commit>`。

## 注意事项
- 不自动推送到 GitHub。如需推送，请在验证完成后手动执行 `git add/commit/push`，并更新本仓的状态/工作日志文件（如使用）。
- 端口保持固定：后端 `5409`、前端 `5173`。若有冲突，先执行端口清理命令。
- 依赖安装优先使用 `npm ci`（有 `package-lock.json` 且需干净安装时），否则使用 `npm install`。

## 下一步建议
- 如需恢复“一键启停/更新脚本”，可从备份分支回溯并在评审后合入。
- 补充 E2E（Puppeteer）与性能基线（Lighthouse CI），确保更新流程在 CI 上也能自动验证。

