
> publish the documentation

```bash
cd ./my-docs
mkdocs build -c -d ../docs
```

> generating the markdown as presenation slides

```bash
pandoc index.md  -f markdown -t beamer -o xac.pdf
```
