# python-mammoth - 增强版

本仓库是 [python-mammoth](https://github.com/mwilliamson/python-mammoth) 的 fork 版本，添加了 OMML 公式 Base64 编码功能，防止 LaTeX 代码在转换过程中被转义。

## 新增功能

### OMML 公式 Base64 编码

在将 OMML (Office MathML) 转换为 LaTeX 后，使用 Base64 编码保护 LaTeX 代码，防止在后续的 HTML 到 Markdown 转换过程中被 markdownify 转义。

**问题背景**：
- Word 文档中的数学公式使用 OMML 格式存储
- mammoth 使用 docxlatex 将 OMML 转换为 LaTeX
- LaTeX 代码包含大量反斜杠 `\`（如 `\alpha`, `\partial`, `\frac` 等）
- markdownify 在转换 HTML 时会转义这些反斜杠，导致 LaTeX 代码损坏

**解决方案**：
1. 将 LaTeX 代码编码为 Base64
2. 使用特殊占位符格式：`⟨OMML:{delimiter}:{base64_data}⟩`
3. 在最后的 Markdown 处理阶段解码恢复 LaTeX

### 占位符格式

```
⟨OMML:$:base64_encoded_latex⟩    # 行内公式
⟨OMML:$$:base64_encoded_latex⟩   # 行间公式
```

**delimiter** 保存公式的定界符信息（`$` 或 `$$`），用于在解码时正确还原。

### 技术实现

**编码阶段** (mammoth/docx/body_xml.py):
```python
import base64
latex_b64 = base64.b64encode(latex.encode('utf-8')).decode('ascii')
latex_placeholder = f"⟨OMML:{delimiter}:{latex_b64}⟩"
```

**解码阶段** (在 markitdown 中):
```python
latex = base64.b64decode(latex_b64).decode('utf-8')
result = f"{delimiter}{latex}{delimiter}"
```

### 修改的文件

- `mammoth/docx/body_xml.py` - 添加 Base64 编码逻辑
- `mammoth/docx/office_xml.py` - 相关的 OMML 处理增强

## 示例

**Word 文档中的公式**:
```
E = mc²
```

**OMML 转 LaTeX**:
```latex
E = mc^{2}
```

**Base64 编码保护**:
```
⟨OMML:$:RT0gbWNeezyyfQ==⟩
```

**最终 Markdown 输出**:
```markdown
$E = mc^{2}$
```

## 安装

从 GitHub 安装：

```bash
pip install git+https://github.com/shiyuanpei/python-mammoth.git@master
```

## 用途

本增强版主要用于 [markitdown](https://github.com/shiyuanpei/markitdown) 项目，
实现包含数学公式的 Office 文档到 Markdown 的准确转换。

与 [docxlatex](https://github.com/shiyuanpei/docxlatex) 增强版配合使用，
提供完整的 Office 公式到 Markdown LaTeX 的转换管道。

## 原始项目

原始 python-mammoth 项目: https://github.com/mwilliamson/python-mammoth

## 许可证

与原项目保持一致的 BSD 2-Clause 许可证。
