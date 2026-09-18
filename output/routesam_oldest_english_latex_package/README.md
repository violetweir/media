# RouteSAM oldest English LaTeX package

This folder contains the oldest archived English LaTeX manuscript and the local files required to compile it.

Main source:

- `cas-dc-template.before_revision.tex`

Compile with:

```bash
latexmk -pdf -interaction=nonstopmode cas-dc-template.before_revision.tex
```

The manuscript source is copied unchanged from the historical backup. The bundled `cas-refs.bib` uses citation-key aliases compatible with that source so that the package can compile independently.
