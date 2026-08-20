# Guía de colaboración

Este repositorio contiene un documento LaTeX que se edita entre varias personas. Para mantener `main` siempre estable, seguimos este flujo:

## Flujo de trabajo

1. **No se trabaja directamente sobre `main`.** Crea una rama a partir de ella:
   ```bash
   git checkout main
   git pull
   git checkout -b nombre-descriptivo
   ```
2. Realiza tus cambios y compila localmente para comprobar que no hay errores (ver más abajo).
3. Haz commit con mensajes claros y en español, describiendo el "por qué" del cambio.
4. Sube la rama y abre un **Pull Request** contra `main`.
5. El PR necesita **al menos 1 revisión aprobada** y que el check de CI (compilación del PDF) pase en verde antes de poder fusionarse.
6. Una vez aprobado, fusiona el PR (se recomienda "Squash and merge" para mantener el historial limpio) y elimina la rama.

## Compilar el documento en local

Necesitas una distribución de LaTeX (TeX Live, MiKTeX, etc.) con `latexmk` y `biber`.

```bash
latexmk -pdf main.tex
```

También puedes revisar el PDF generado automáticamente como artefacto en cada Pull Request, en la pestaña "Checks" → workflow "Compilar documento LaTeX".

## Estructura del proyecto

```
main.tex               Documento principal
secciones/             Una sección por archivo, incluida desde main.tex
referencias.bib        Bibliografía
.github/workflows/     Compilación automática en cada push/PR
```

## Buenas prácticas

- No subas archivos generados (`.pdf`, `.aux`, `.log`, `.toc`, ...); ya están en `.gitignore`.
- Una rama y un PR por tema/sección para facilitar la revisión.
- Si hay conflictos, resuélvelos localmente y vuelve a subir la rama antes de pedir revisión.
