
# ACMM 网站（acmm-site）

这是中山大学 ACMM 协会用于记录算法笔记、题库与社团信息的 MkDocs 网站源码。

## 快速开始（克隆到本地）

在终端中运行：

```bash
git clone https://github.com/SemsueZhang/acmm-site.git
cd acmm-site
```

## 本地预览与编辑（使用 MkDocs）

建议使用虚拟环境并安装依赖：

Windows (PowerShell):

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

macOS / Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

启动本地预览：

```bash
mkdocs serve
```

在浏览器打开 http://127.0.0.1:8000 即可查看实时预览，保存文件后会自动刷新。

构建静态站点：

```bash
mkdocs build
```

部署到 GitHub Pages（如果仓库配置了）：

```bash
mkdocs gh-deploy
```

## 在本地编辑文档

- 所有页面位于 `docs/` 目录下，修改或新增 Markdown 文件即可。  
- 修改导航请编辑 `mkdocs.yml` 中的 `nav` 字段以保证侧边栏与站点一致。  
- 静态资源（图片、CSS、JS）放在 `docs/` 下的合适子目录（例如 `docs/assets/`、`docs/stylesheets/`、`docs/javascripts/`）。

内容约定建议：
- 使用简体中文撰写说明与教程。  
- 算法页面建议包含：背景、核心思想、分步讲解、示例题、伪代码、复杂度公式（LaTeX）、C++17 参考实现、常见错误。  
- 代码块请使用标注语言，如 ```cpp 来包裹 C++ 示例。

## 常见命令速查

```bash
git status
git add <files>
git commit -m "描述性提交信息"
git push origin <branch>
```

## 联系与贡献

欢迎通过 Fork → 新建分支 → 提交 PR 的方式参与贡献。详细贡献流程请见 `CONTRIBUTING.md`。
