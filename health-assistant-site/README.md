# Personal Health Assistant — health.anotherdayu.com

纯静态 HTML/CSS/JS，无需构建或数据库。仓库目录：`health-assistant-site/`；实际网站根目录：`site/health-assistant/`。本仓库没有自动部署工作流。

网站包含主页、隐私政策、条款，以及浏览器本地快照图表。图表不会自动连接 Drive，不上传或持久保存文件。不要把私人快照、CSV、令牌或客户端密钥放进公开网站目录。对从 Drive 下载的 v2 snapshot JSON，可在 `/visualize/` 中选择，按指标和来源查看。缺失日不连接、不补零，最多显示最近 90 个有记录日期，不计算医疗结论。

## 1. DNS

在 anotherdayu.com 的 DNS 管理中新增 A 记录：主机名 `health`，值为 VPS 公网 IPv4。仅在 VPS 确有可达 IPv6 时增加 AAAA。删除或纠正 health 的旧错误 AAAA。不要改主域名与 www 的记录。首次配置若使用 Cloudflare，先用 DNS only，HTTPS 成功后再按需启用代理。

云安全组及 VPS 防火墙允许 TCP 80、443，并保留已有 SSH 规则。不要为了本站停止或覆盖其他网站。

## 2. 拉取代码（在 VPS 执行）

```bash
git clone --filter=blob:none --sparse --single-branch --branch health-assistant-site https://github.com/DayuGuo/anotherdayu_resourse.git health-site-src
cd health-site-src
git sparse-checkout set health-assistant-site
```

仓库是公开资源仓库，此命令只取网站目录。后续更新在这个 checkout 中运行 `git pull --ff-only`，再复制网站文件。

## 3A. Ubuntu / Debian + Nginx（常见 VPS）

下方假定你使用 Ubuntu/Debian 且通过 sudo 管理；其他系统请用对应包管理器。已有反向代理面板（宝塔、1Panel 等）时，直接新增静态网站，域名填 health.anotherdayu.com，网站目录指向下述目录，再由面板申请证书；不要同时安装另一套代理抢占端口。

```bash
sudo apt update
sudo apt install -y nginx certbot python3-certbot-nginx git
sudo mkdir -p /var/www/health-assistant
sudo cp -a health-assistant-site/site/health-assistant/. /var/www/health-assistant/
sudo find /var/www/health-assistant -type d -exec chmod 755 {} +
sudo find /var/www/health-assistant -type f -exec chmod 644 {} +
```

先确认 `/etc/nginx/sites-available/health.anotherdayu.com` 不存在；若存在先审阅现有内容，不要覆盖。首次新增时：

```bash
sudo cp health-assistant-site/nginx-subdomain.conf.example /etc/nginx/sites-available/health.anotherdayu.com
sudo ln -s /etc/nginx/sites-available/health.anotherdayu.com /etc/nginx/sites-enabled/health.anotherdayu.com
sudo nginx -t
sudo systemctl reload nginx
```

DNS 生效后，先访问 `http://health.anotherdayu.com/`。确认看到正确页面，再运行：

```bash
sudo certbot --nginx -d health.anotherdayu.com --redirect
sudo certbot renew --dry-run
```

按 Certbot 提示输入你的证书通知邮箱并阅读条款。检查系统的证书续期 timer/cron 已启用。若 nginx -t 失败先修复配置，不要 reload。若证书申请失败，先检查 DNS（包括 AAAA）、80/443 安全组和端口占用，不要反复申请。

## 3B. 已有 Caddy

无需 Nginx/Certbot。把网站内容复制到 `/srv/health-assistant/`，使 Caddy 服务账户有读取权限，将 `Caddyfile.example` 的站点块追加到现有 Caddyfile（不要覆盖已有站点），validate 后 reload。域名解析到 VPS、80/443 可达时，Caddy 自动签发与续期 HTTPS。

## 4. 上线验证

```bash
curl -I https://health.anotherdayu.com/
curl -I https://health.anotherdayu.com/privacy/
curl -I https://health.anotherdayu.com/terms/
curl -I https://health.anotherdayu.com/visualize/
```

四个 URL 应返回 200，证书有效，网页互相链接正常。用浏览器检查手机宽度与政策内容。不要上传健康数据作为部署测试。

## 5. Google OAuth（上线检查后继续）

- Application home page: https://health.anotherdayu.com/
- Privacy policy: https://health.anotherdayu.com/privacy/
- Terms of Service: https://health.anotherdayu.com/terms/
- Authorized domain: anotherdayu.com（不是带 https 的 URL）

如 Google 要求域名验证，在 Search Console 验证本人所有权；DNS TXT 值必须使用 Google 实际提供的值。不要改变现有 Apps Script `/usercallback` 回调。网站部署不代表 Production 或 Google 审核已经完成。

部署后回到原 Codex 聊天回复“已部署”，再检查线上页面、保存 Branding、尝试切换 Production、重新运行 authorizeHealth 完成本人同意，最后检查两个 HTTP 200 与真实同步。仍可能需要本人处理 Google 验证或审核。

## 本地预览

```bash
python3 -m http.server 8080 --directory site/health-assistant
```

打开 `http://localhost:8080/`。日期为政策草稿完成日期，发布前请审阅，并按实际变更更新。
