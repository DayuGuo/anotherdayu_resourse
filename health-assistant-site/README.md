# Personal Health Assistant — 说明网站

纯 HTML/CSS，无需构建、数据库或 Node.js。网站源文件在 `site/health-assistant/`。

## 部署

将 `site/health-assistant/` 整个目录复制到 **anotherdayu.com 已有网站的文档根目录**，保留目录层级。不要替换现有博客根目录。若主域名由其他平台托管，须先安排 `/health-assistant/` 路径转发至 VPS；仅把文件放到 VPS 不会改变域名的现有路由。

用于已有 Nginx server 块的配置示例见 `nginx-location.conf.example`，须根据真实目录调整，再由你测试配置并 reload。本仓库没有自动部署 workflow，不启用 GitHub Pages。

## 本地预览

在项目目录运行 `python3 -m http.server 8080 --directory site`，打开 `http://localhost:8080/health-assistant/`。

## 上线后填写 Google OAuth

- Application home page: https://anotherdayu.com/health-assistant/
- Application privacy policy link: https://anotherdayu.com/health-assistant/privacy/
- Application Terms of Service link: https://anotherdayu.com/health-assistant/terms/
- Authorized domain: anotherdayu.com

先确认三个真实 HTTPS 页面均公开可访问、内容正确，并按 Google 要求在 Search Console 验证域名所有权。然后保存 Branding，再检查能否切换 Production。页面上传和部署本身不代表 Google 审核或 Production 已完成。切换后重新完成健康授权并运行诊断。

现有 Apps Script `/usercallback` 回调保持不变；此静态网站不处理授权回调。

## 数据边界

只包含公开应用说明和政策，不包含健康记录、Apps Script 源码、Client secret、令牌或授权链接。隐私政策描述当前 v3 脚本行为；如以后接入 AI 或新增用途，应先更新披露并处理必要的授权。

服务器日志策略取决于你的部署设置；请审阅政策后再发布。日期为草稿完成日期，实际发布时如需调整，统一修改政策和条款。
