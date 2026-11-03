# yt-bmx · YouTube 抓取器（境内无需翻墙）

在 GitHub 的海外机器上跑 `yt-dlp`，把视频打包成 **Release 资产**。你只需从网页下载——
`objects.githubusercontent.com` 在国内可达（实测 200 / 0.8s）。

---

## 1. 建仓库

浏览器打开 <https://github.com/new>

- Repository name：`yt-bmx`
- 勾 **Public**（Release 资产才能匿名下载）或 Private 都行，Public 更省事
- 不要勾 README / .gitignore / license
- 点 **Create repository**

## 2. 上传文件

仓库页 → **uploading an existing file** → 把本目录里的 `.github` 文件夹整个拖进去
（拖文件夹会自动保留 `.github/workflows/grab.yml` 路径）→ Commit changes。

> 如果拖拽没认出 `.github`（Windows 有时隐藏点开头的文件夹）：
> 仓库页 → **Add file → Create new file** → 文件名直接输入
> `.github/workflows/grab.yml`（输入 `/` 会自动建目录）→ 把 `grab.yml` 内容粘进去 → Commit。

## 3. 跑第一次

仓库页 → **Actions** 标签 → 左侧 **Grab YouTube BMX Flatland** → 右侧 **Run workflow**

填参数：

| 参数 | 含义 | 建议首发值 |
|---|---|---|
| `keywords` | 关键词，英文逗号分隔 | 默认 5 个词够用 |
| `days_back` | 只抓最近 N 天发布 | `62` |
| `max_per_keyword` | 每个词取多少条搜索结果 | `40` |
| `max_height` | 最高分辨率 | `1080`（想快就 `720`）|
| `dry_run` | `true` = 只列清单不下载 | **首发先用 `true` 试水** |

**首跑务必先 `dry_run=true`**：几十秒就能看到解析出多少条、标题日期对不对，
确认没问题再改 `false` 正式下。

## 4. 如果撞上 "Sign in to confirm you're not a bot"

GitHub 的 runner 是机房 IP，YouTube 对这类 IP 有机器人校验。症状是日志里出现
`Sign in to confirm you're not a bot` 或大片 `403`。解法是带浏览器 Cookie：

1. 在你自己的电脑上装浏览器扩展 **Get cookies.txt LOCALLY**（Chrome/Edge 商店）
2. 打开 <https://www.youtube.com>（需要你能访问，或用朋友/手机热点协助）
3. 扩展点 **Export** → 存成 `cookies.txt`（Netscape 格式）
4. 仓库 → **Settings → Secrets and variables → Actions → New repository secret**
   - Name：`YT_COOKIES`
   - Secret：把 `cookies.txt` 全文粘进去
5. 重跑 workflow，会自动带上登录态

> Cookie 有效期通常几天到几周，过期就重导一次。Secret 只有 Actions 能读，日志里不会回显。

## 5. 把视频拿回本机

跑完后两种取法：

**A. Release（推荐，永久保留、匿名可下）**
仓库页 → **Releases** → 点最新那条 → 直接点文件名下载。
批量下用命令行（免登录）：

```bash
curl -L -O https://github.com/<你的用户名>/yt-bmx/releases/download/<tag>/<文件名>
```

**B. Artifact（保留 30 天，需登录）**
Actions → 点开那次运行 → 页面底部 **Artifacts** → 下载 zip。

## 6. 自动化

`grab.yml` 里已内置每月定时（UTC 20:00 → 北京时间次日 04:00）自动抓一次。
`.archive.txt` 会回写到仓库，实现跨次去重——下过的不会再下一遍。

想改成每周，把 `cron: '0 20 2 * *'` 改成 `'0 20 * * 1'`（每周一）。

---

## 文件说明

| 文件 | 作用 |
|---|---|
| `.github/workflows/grab.yml` | 全部逻辑：搜索 → 按日期过滤 → 下载 → 打包 → 发 Release → 回写去重 |
| `.archive.txt` | 跨次去重的已下载 ID 记录（首次运行自动生成）|

## 已知限制

- **单文件 2GB 上限**：Release 资产限制，超长片（如 1.5 小时全场）会被跳过并在日志里告警。
  需要的话改成分段或走 artifact。
- **抓的是搜索结果，不是频道全集**：`ytsearchdate` 按发布时间排序，冷门频道可能排不进前 N 条。
  要盯特定频道，把 `keywords` 换成频道 URL 也能用（yt-dlp 支持直接喂频道地址）。
- **私享/会员视频**下不了。
