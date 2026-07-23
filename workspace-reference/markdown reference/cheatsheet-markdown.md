### Markdown Cheat Sheet

Quick reference for essential Markdown syntax. Use it to format docs, notes, or READMEs efficiently.

#### Headers
Create hierarchy with `#` symbols (1-6 levels).
```
# H1 (largest)
## H2
### H3
#### H4
##### H5
###### H6
```
Renders as: # H1, ## H2, etc.

#### Text Formatting
```
*italic* or _italic_
**bold** or __bold__
***bold italic*** or ___bold italic___
~~strikethrough~~
`inline code`
```
Example: **Bold text** with *italics* and `code`.

#### Lists
**Unordered** (bullets):
```
- Item 1
- Item 2
  - Subitem (indent 2 spaces or 4)
```
**Ordered** (numbers):
```
1. First
2. Second
   1. Nested
```
**Task lists**:
```
- [x] Done
- [ ] Todo
```

#### Links & Images
**Links**:
```
[Link text](https://example.com)  
[Link with title](https://example.com "Tooltip")
```
**Images**:
```
![Alt text](image.jpg)  
![Alt text](https://example.com/image.jpg "Tooltip")
```

#### Code Blocks
**Inline**: `` `code` ``  
**Block** (fenced, with optional language):
```
```python
def hello():
    print("Hello, Markdown!")
```
```
Indent 4 spaces also works for plain blocks.

#### Quotes & Horizontal Rules
**Blockquote**:
```
> Quote block
> > Nested quote
```
**HR** (separator):
```
---
***
___
```

#### Tables
```
| Header 1 | Header 2 |
|----------|----------|
| Cell 1   | Cell 2   |
| Right-aligned | $100    |  (use `:-:` for center, `---:` for right)
```
Align: `|:---|` left, `|---:|` right, `|:---:|` center.

#### Extras
- **Escaping**: Backslash `\` before `*`, `#`, etc.: `\*\*bold\*\*`
- **Line breaks**: Two spaces at line end or `<br>`
- **Emoji**: `:smile:` (GitHub) or native 😀
- **Math** (with extensions like MathJax): `$E=mc^2$` or `$$ \int f(x) dx $$`

Test in a Markdown viewer like GitHub or Dillinger. Need examples for a specific tool like VSCode?