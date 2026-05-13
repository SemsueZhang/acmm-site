# 贡献指南（CONTRIBUTING）

感谢你愿意为本项目贡献内容！以下是推荐的工作流程和注意事项，帮助你快速上手并提高合并通过率。

## 工作流程

1. Fork 本仓库到你的 GitHub 账号。
2. 在本地克隆你 Fork 后的仓库：

```bash
git clone https://github.com/<yourname>/acmm-site.git
cd acmm-site
```

3. 新建分支，分支名请短且语义化，例如：

```bash
git checkout -b feat/kmp-page
```

4. 在 `docs/` 下修改或新增页面，必要时更新 `mkdocs.yml` 的 `nav`。

5. 在本地预览你修改的页面：

```bash
python -m venv .venv
.\.venv\Scripts\Activate.ps1   # Windows
source .venv/bin/activate        # macOS / Linux
pip install -r requirements.txt
mkdocs serve
```

6. 本地确认无误后提交并推送分支：

```bash
git add .
git commit -m "feat(docs): 补充 KMP 教学页面"
git push origin feat/kmp-page
```

7. 在 GitHub 上创建 Pull Request，填写清晰的变更说明并标注相关人员或 reviewer（如有）。

## 文档与代码规范

- 内容语言：简体中文为主，技术术语可保留英文。  
- 算法页面风格建议：背景 -> 思路 -> 步骤 -> 示例题 -> 伪代码 -> 复杂度（LaTeX）-> C++17 代码 -> 常见错误。  
- 代码示例请使用 C++17，统一使用 STL，添加中文注释以便阅读。  
- 数学公式请用 LaTeX（`$...$` 或 `$$...$$`），站点已配置 MathJax。  
- 图片或资源请放到 `docs/assets/` 或其他子目录，并在 Markdown 中使用相对路径引用。

## 提交信息建议

- 使用简短前缀，例如 `feat(docs):`、`fix(docs):`、`chore:`，并在描述中说明主要改动。

示例：

```
feat(docs): 新增 Trie 教学页面，包含示例与 C++ 实现
```

## PR 评审注意点

- 确认导航 `mkdocs.yml` 是否正确更新，避免死链。  
- 本地用 `mkdocs serve` 预览并检查公式、代码块与图片显示是否正常。  
- 保持单个 PR 聚焦于一个主题（例如只改一页或只改导航），便于审阅。  

## 维护者合并策略（参考）

- CI / 构建通过且内容符合规范 → 允许合并。  
- 对教学内容的重大结构性修改建议先打开 Issue 讨论再提交 PR。  

## 其他说明

- 若你希望贡献题库（`docs/problems/`），请注明题目来源或标注“示例题”。  
- 如果对站点样式、主题有改动建议，请先在 Issue 里讨论，避免无意间影响全站样式。  

再次感谢你的贡献！如需我帮你把某个页面改成合规范的格式，也可以直接发变更请求说明。  
