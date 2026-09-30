# Proyecto2627 · Documentación

## Puesta en marcha

```bash
python -m venv .venv
source .venv/bin/activate      # En Windows: .venv\Scripts\activate
pip install -r requirements.txt
properdocs serve               # http://127.0.0.1:8000
```

## Generar la web estática

```bash
properdocs build               # crea la carpeta site/
```

## Publicar en GitHub Pages

```bash
properdocs gh-deploy
```
