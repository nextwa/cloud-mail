# CloudMail 自动同步修复交接（2026-11-17）

生产部署源 nextwa/cloud-mail；上游 maillab/cloud-mail；Gitea www/cloud-mail。

11月12日起每日同步失败：merge-upstream 尝试导入官方 deploy-cloudflare.yml 的改动，GITHUB_TOKEN 无 workflows 权限被拒绝。官方此次工作流差异只是步骤名称翻译，没有需要移植的部署逻辑。

修复 sync-upstream.yml：基于 .upstream-revision 增量三方合并应用代码，排除 .github 和交接文档，保留部署配置；成功后显式 workflow_dispatch，因为 GITHUB_TOKEN 提交不会触发 push 工作流。冲突停止，不强推。每天10:17 UTC+8保持不变。

## 自动更新链路

每日10:17（UTC+8）检查上游 main，GitHub可能延迟执行。官方发布 release 不会直接触发 fork 更新，由此定时任务检查最新应用代码。

同步使用 .upstream-revision 记录已应用的上游版本；排除 .github、AGENTS.md、DEPLOYMENT.md。三方合并保留本地改动，冲突停止，不强制推送。同步后显式 dispatch 部署任务；没有更新时只有上次部署失败才重试。手动执行会显式部署。

上游工作流变更需单独审核。不要恢复 merge-upstream，也不要为此申请全仓库长期高权限令牌。

Gitea www/cloud-mail 是 GitHub 拉取镜像（间隔8小时），不能直接 git push。应推送 GitHub main 后通过 Gitea mirror-sync 触发并读回 main 验证。凭据使用系统已存 Git 凭据，不写入仓库或输出。

## 本次交付进度

2026-11-17：GitHub修复已发布；手动同步任务 44087923608 成功，应用提交 9e2f31ad4cb6f5ee0e15fa24e236cae73063e3f6 与上游应用代码一致。已触发部署任务 44087950277，最终部署验收结果将在本文件后续补录。

目前官方 release 为 v3.5.0，上游 main 页面版本为 v3.5.1。本任务跟随 main（与既有策略一致），不要将 main 版本误称为已发布 release。

生产 Worker cloud-mail，域名 https://maxmailx.com 。数据库、KV、R2沿用现有资源。只做部署和定点验证，不运行额外哈希及完整回归测试。

## 最终验收（2026-11-17 21:00 UTC+8）

- 同步任务 44087923608 成功；部署任务 44087950277 完成 success，部署步骤和数据库初始化步骤均 success。
- Cloudflare Worker 部署 374fe204-66d5-4333-9268-9ee107c0aa36，版本 2dab4ca5-d6f8-404a-9f6a-19e9598e6ea8，2026-11-17T12:58:10Z 创建，100%流量。
- 生产 https://maxmailx.com HTTP 200；生产系统设置资源 /assets/index-C8kmUjXs.js 实际包含 v3.5.1。应用目录与当前上游 main 一致。
- 浏览器当前已有的页面未完成渲染；没有宣称完成登录、收发信验收。本次验证范围是更新链路、部署/数据库初始化和生产版本读取。
- 维护文档提交不触发应用部署（原有路径过滤），更新工作流与文档已保留为本地维护文件。
