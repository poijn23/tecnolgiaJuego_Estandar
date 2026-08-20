# tecnolgiaJuego_Estandar



Documento LaTeX colaborativo.

## Estructura

```
main.tex               Documento principal (incluye las secciones)
secciones/              Contenido dividido por sección
referencias.bib          Bibliografía
.github/workflows/       CI: compila el documento en cada push/PR y publica el PDF como artefacto
```

## Compilar en local

```bash
latexmk -pdf main.tex
```

## Colaborar

Este repositorio está preparado para trabajo en equipo: la rama `main` está protegida y los cambios se integran mediante Pull Requests con al menos una revisión aprobada. Consulta [CONTRIBUTING.md](CONTRIBUTING.md) para el flujo de trabajo completo.

