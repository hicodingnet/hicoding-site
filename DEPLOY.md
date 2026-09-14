# hicoding.net 官网部署（GitHub Pages 临时托管 → 备案后回迁腾讯云）

> 2026-09-01 建。目的：苹果组织账号注册需要「公开可访问 + 与组织关联」的官网，
> ICP 备案号下来前先用 GitHub Pages 境外托管顶上（境外托管无需备案，合规）。
> 本目录 = 官网仓库全部内容（index.html + CNAME）。
> **9/8 状态更新**：组织账号已到手（9/3），GitHub Pages 上线流程（§一/§二）已成历史；
> 当前等待=ICP/App 备案出号（预期 9 月中下旬，≤9/22 硬闸门）→ 出号日按 §三 回迁+挂号。

## ⚠️ 上线前必核对（苹果审核视角）

1. ✅ 页脚/关于我们已含公司全称「广州海码智能科技有限公司」+ 英文名
   「Guangzhou Haima Intelligent Technology Co., Ltd」（2026-09-02 按用户 DUNS 记录补齐）
2. ✅ 联系邮箱已定 `admin@hicoding.net`——**去邮箱服务商把这个地址建出来**
   （腾讯企业邮箱/域名邮箱均可），否则收不到 Apple 验证信

## 一、GitHub 上线（~10 分钟）

```bash
cd /Users/stronghuang/hicoding.net/website
git init && git add -A && git commit -m "hicoding.net 官网上线"
# GitHub 建一个名为 hicoding-site 的仓库（Public），然后：
git remote add origin git@github.com:<你的用户名>/hicoding-site.git
git push -u origin main
```

GitHub 仓库 → Settings → Pages：
- Source: Deploy from a branch → main / (root)
- Custom domain: 填 `hicoding.net` → Save
- 等 DNS 校验通过后勾选 **Enforce HTTPS**

## 二、DNS 添加记录（22.cn 的 DNS 管理面板）

| 类型 | 主机记录 | 记录值 |
|---|---|---|
| A | @ | 185.199.108.153 |
| A | @ | 185.199.109.153 |
| A | @ | 185.199.110.153 |
| A | @ | 185.199.111.153 |
| CNAME | www | <你的用户名>.github.io |

生效校验：`dig +short hicoding.net @8.8.8.8` 返回上面 4 个 IP；浏览器打开 https://hicoding.net

## 三、备案号下来后回迁腾讯云（官网正式版）

1. 官网页面加挂 ICP 备案号（index.html 页脚有占位注释）
2. 把页面部署到腾讯云轻量服务器（Nginx 静态站点或宝塔面板）
3. 22.cn 把上表 5 条记录删除，A @ 改为轻量服务器公网 IP
4. GitHub 仓库归档或保留作镜像（建议保留，作灾备）

## 时序

- ~~DUNS 查询 / 组织账号注册~~ ✅ 全部完成（DUNS 8/31 出码，组织账号 9/3 到手）
- **当前节点**：备案管局审核中（≤9/22）→ 出号日动作见 §三 + M2「出号日四连」
- ✅ **出号日官网三连已完成（9/11）**：① footer 挂「粤ICP备2026135341号」（链工信部）② 轻量（193.112.210.120，OpenCloudOS 9.6）装 nginx（dnf 需 `--disableexcludes=all`）+ 三页面上传 ③ DNS 已切（A @ / CNAME www）。**9/11 加 https**：certbot+Let's Encrypt（HTTP-01）签发 hicoding.net+www，nginx 443 配置 `/etc/nginx/conf.d/hicoding.conf`（80→301→https），`certbot-renew.timer` 已启用（90 天自动续）。**四连前置全部就绪**
- ✅ **公安联网备案网站(-1)过审（9/12 17:41:39 生效）：粤公网安备44011102485268号** → 9/14 三页页脚回挂（查询链 `beian.mps.gov.cn/#/query/webSearch?code=…` + 官方徽标 `assets/ghs.png` 自托管）并部署外网验证。**APP(-2A) 仍在审核**，出号后补挂 App 内备案展示行

## 五、多 App Universal Links 规范（9/11 定调）

AASA（`/.well-known/apple-app-site-association`）的 `details` 为数组，按 App 追加条目：
- 可期日历：appID `4Z3JZ7U43K.net.hicoding.hical`，路径 `/app/*`（已上线）
- 新 App 上线时：追加 `{"appIDs": ["4Z3JZ7U43K.net.hicoding.<appkey>"], "components": [{"/": "/<appkey>/*"}]}`，路径前缀与 App 一一对应、禁止重叠
- Bundle ID 规范 `net.hicoding.<appkey>`；自定义 Scheme 同理 `<appkey>://`
