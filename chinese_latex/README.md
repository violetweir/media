# RouteSAM 中文稿

- 中文源码：`routesam_zh.tex`
- 中文预览：`routesam_zh.pdf`
- 对应英文投稿主稿：`../cas-dc-template.tex`
- 共享方法图：`../figs/figure1.png`
- 共享参考文献库：`../related_work_papers/references.bib`

中文稿用于梳理论文逻辑和讨论内容，采用单栏阅读版式；英文稿继续使用 Elsevier CAS 双栏模板。中文稿应使用 XeLaTeX 编译，例如：

```text
latexmk -xelatex routesam_zh.tex
```

当前两份稿件均保留最终实验结果占位符。阶段 2 的审核公式、阈值、参数高效适配模块和优化设置，需要在自动锚点闭环完成后统一冻结。
