---
name: edge-literature-download
description: Search for academic papers in Google Chrome or Microsoft Edge, download full-text PDFs using the connected browser profile, and verify each saved paper. Use for a research topic, title, DOI, article link, or paper list, including authorized publisher and institutional access.
metadata:
  version: "0.2.0"
---

# Chrome / Edge 文献检索与下载

用 Codex 已连接的 Google Chrome 或 Microsoft Edge 检索论文、打开出版社或资料库页面并保存全文 PDF。先下载到用户指定目录，再以本地 PDF 为依据阅读和总结。已绑定项目且用户未指定目录时，保存到项目下的文献子目录。先确认目标路径是本机绝对路径。

## 选择浏览器

- 使用 `mcp__cua_repl`，以工具当前返回的文档为准。用户指定标签页时，绑定该标签页；用户指定浏览器时，用对应的 `"chrome"` 或 `"edge"` 创建新标签页。
- 用户未指定时，优先复用能访问目标资源的已连接配置文件。同等条件下沿用 Edge 作为默认浏览器。只有 Chrome 已连接时直接用 Chrome。使用连接的配置文件，不跨浏览器迁移登录状态。
- 首次创建标签页时传入简短的会话名，例如 `cua.createBrowserTab("chrome", url, { sessionName: "🔎 文献检索" })`；Edge 使用相同调用并将首个参数改为 `"edge"`。已经绑定用户标签页时沿用当前连接。
- 所需浏览器未接入或登录态在另一个配置文件时，请用户连接正确的浏览器配置文件。当前连接能打开页面但受浏览器弹窗或其他扩展阻挡时，按工具文档发起手动接管，请用户处理弹窗并交回控制，再从原页面继续。

## 定位论文

1. 有 DOI 或稳定文章 URL 时直接打开。仅有题名时先查精确题名。按主题检索时，依据用户给定的主题、年份、文献类型和数量拟定检索式，打开学术索引、数据库或出版社搜索页面，记录实际使用的检索式及纳入条件。根据题名、作者、年份和 DOI 去重，区分预印本与正式发表版。
2. 在来源页核对题名、DOI（如有）和版本。沿可见链接使用当前浏览器配置文件获准的出版社或机构登录态。需要 WebVPN 时从已知的机构入口进入并遵循实际导航。按浏览器安全规则处理登录、验证码、权限或访问限制。允许时可另查出版社开放获取版或可信仓储版本，并注明版本和来源。

## 下载并保存

1. 对页面上可见且已确认属于目标论文的直接 PDF 链接，优先使用当前浏览器 API 支持的 `locator.downloadMedia()`，保留它返回的本机文件路径。若需自定义超时，按当前文档选择参数名。不要把目录页、摘要页或签名链接当成已下载的 PDF。
2. 下载由页面按钮触发时，若当前 API 提供 `tab.playwright.waitForEvent("download")`，在点击前注册等待，再用下载对象的 `path()` 获取本机路径。控制若导航到 PDF 阅读器，就检查阅读器当前状态和可见的保存控件。阅读器可能只在截图中显示工具栏，可据当前截图点击。若另开标签页，先从可见的浏览器状态确认它对应本篇再操作。对同一个下载动作不要因未收到事件就重复点击。
3. 浏览器未返回本机路径时，只接受本次操作中可明确归属的保存证据，例如用户完成浏览器的「另存为」后提供的路径。读取本地候选文件需有当前动作返回的路径、明确的保存位置或用户确认；不要用「下载目录里最新的 PDF」猜测。若浏览器工具无法控制保存对话框或无法确认路径，按浏览器文档交由用户完成保存并告知文件路径。不要访问工具禁止的浏览器内部页面，也不要经其他接口绕过。不要导出登录状态或在结果中保存临时签名 URL。
4. 对批量任务，每篇保留自己的控制动作、实际来源和下载路径。对无法区分归属的阅读器操作，只尝试一篇，确认文件后再处理下一篇。某篇受阻时可继续其他独立条目。

## 核验并交付

用安装了 `requirements.txt` 中依赖的 Python 3.10+ 运行 [PDF 核验脚本](scripts/verify_pdf.py)（脚本路径相对本技能目录）：

```powershell
python scripts/verify_pdf.py "C:/absolute/path/paper.pdf" --doi "10.xxxx/example" --title "Expected article title"
```

有 DOI 和题名时都传入，缺 DOI 时去掉该选项。检查 `valid_pdf`、`identity_match`、页数和 DOI 冲突。读取得到的签名、哈希和页数仅用于文件验证，下载来源来自浏览器操作。预印本 DOI 与正式版 DOI 的差别须单独核对；显式 DOI 冲突、文字不足、封面或扫描件应检查实际文章首页，仍不明确则报告待核验。

验证通过后，将 PDF 复制到目标目录，使用英文文件名并避免覆盖同名文件。以目标副本再次运行核验，确认 SHA-256 一致。单篇报告题名、DOI（如有）、版本、保存路径和页数。多篇逐条列出已验证、待核验或受阻的结果和具体原因。用户要求阅读或综述时，在保存并核验后读本地 PDF。

本文献下载技能从 [ego-literature-download](https://github.com/deathcats4/ego-literature-download) 的工作流改编，沿用其 MIT 许可下的 PDF 核验脚本。
