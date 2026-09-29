# RouteSAM English manuscript

`routesam_en_twocolumn.tex` is the English manuscript based on the current
`../chinese_latex/routesam_zh_twocolumn.tex`. It uses the Elsevier CAS
double-column format from `../cas-dc-template.tex`.

The folder includes local copies of the CAS class, style, bibliography style,
and reference database, so the English manuscript can be compiled here:

```text
latexmk -pdf -interaction=nonstopmode routesam_en_twocolumn.tex
```

`routesam_en_twocolumn.pdf` is the compiled preview. The original English
template is preserved at `template_reference/cas-dc-template.tex` for reference;
its older method description is superseded by the translated manuscript.

The author, affiliation, and final quantitative results remain placeholders,
matching the incomplete information in the current Chinese draft. The Chinese
draft also does not yet contain an experiments section.
