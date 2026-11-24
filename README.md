# Chrome / Edge 文献检索与下载

这是一个 Codex Skill。它在已连接的 Google Chrome 或 Microsoft Edge 中查找论文，沿用该浏览器配置文件已有的机构或出版社访问权限，下载并核验全文 PDF。支持按主题、题名、DOI、文章链接或论文列表发起任务。

## 安装

需要能连接 Chrome 或 Edge 的 Codex，以及 Node.js/npm。PDF 核验需要 Python 3.10 或更高版本和 [`requirements.txt`](requirements.txt) 中的依赖。

```sh
npx --yes skills add https://github.com/xyj0727/edge-literature-download -g -a codex -y --copy
```

重启 Codex 后，调用 `$edge-literature-download`。如需为 PDF 核验准备独立环境，可在仓库目录运行下列 PowerShell 命令，并让 Codex 使用该环境中的 Python。

```powershell
python -m venv .venv
& .\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

该安装方式按需手动更新。可运行 `npx skills update -g`，或重新运行安装命令以安装仓库当前版本。

## 用法

> 用 $edge-literature-download 在 Chrome 中下载 DOI 为 10.48550/arXiv.1706.03762 的论文，把 PDF 保存到项目的 literature 文件夹，并核对题名和页数。

> 用 $edge-literature-download 在 Edge 中查找 2020 年以来关于带翼单桩抗拔承载力的论文。下载能访问的全文，报告每篇的 DOI、版本和下载结果。

在提示中指定 Chrome 或 Edge，或直接指定一个已连接的标签页。机构订阅和 WebVPN 取决于所选配置文件当前拥有的访问权限。浏览器扩展弹窗、验证码或另存为对话框可能需要接管，交回控制后可从原页面继续。批量下载会逐篇核对文件归属。

## PDF 核验

[`verify_pdf.py`](scripts/verify_pdf.py) 也可独立执行。它检查 PDF 是否可解析，报告页数、SHA-256 和论文身份匹配结果。题名或 DOI 证据不足时，需核对文章首页。

```powershell
python scripts/verify_pdf.py "C:/path/to/paper.pdf" --title "Attention Is All You Need"
```

## 许可

采用 [MIT 许可证](LICENSE)。工作流改编自 [ego-literature-download](https://github.com/deathcats4/ego-literature-download)，并沿用其 PDF 核验脚本。
