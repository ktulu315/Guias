# Cheatsheet Rápido de Git

Referencia rápida en orden progresivo de necesidad. Para guía detallada, ver `Guia_Git_Avanzada_Profesional.md`.

## 1. Configuración Inicial

- `git config --global user.name "Tu Nombre"` → Identidad para commits
- `git config --global user.email "tu@email.com"` → Email para commits
- `git config --global init.defaultBranch main` → Rama por defecto 'main'
- `git config --global core.autocrlf true` → Normalizar saltos de línea (Windows)
- `git config --global --list` → Ver toda la configuración
- `git config --global alias.lg "log --oneline --graph --decorate --all"` → Alias para log visual

## 2. Inicio y Flujo Básico

- `git init` → Inicializar repositorio nuevo
- `git clone <url>` → Clonar repositorio remoto
- `git status` → Ver estado del working directory
- `git add <archivo>` → Agregar archivo al staging
- `git add .` → Agregar todos los cambios al staging
- `git commit -m "mensaje"` → Crear commit con mensaje
- `git log --oneline` → Historial resumido de commits

## 3. Ramas

- `git branch` → Listar ramas locales
- `git branch <nombre>` → Crear nueva rama
- `git switch <rama>` → Cambiar a otra rama
- `git switch -c <rama>` → Crear y cambiar a nueva rama
- `git merge <rama>` → Fusionar otra rama en la actual
- `git branch -d <rama>` → Eliminar rama

## 4. Remotos

- `git remote -v` → Ver repositorios remotos configurados
- `git remote add origin <url>` → Conectar remoto 'origin'
- `git push -u origin <rama>` → Subir rama y establecer upstream
- `git push` → Subir cambios al remoto
- `git pull` → Traer y fusionar cambios del remoto
- `git fetch` → Traer cambios del remoto sin fusionar

## 5. Deshacer Cambios

- `git restore <archivo>` → Descartar cambios locales de un archivo
- `git restore --staged <archivo>` → Sacar archivo del staging
- `git reset --soft HEAD~1` → Deshacer commit (cambios quedan en staging)
- `git reset --hard HEAD~1` → Deshacer commit (descarta cambios)
- `git revert <commit>` → Crear commit inverso (seguro para compartir)
- `git clean -fd` → Eliminar archivos no rastreados

## 6. Avanzado

- `git rebase <rama>` → Reubicar commits sobre otra base
- `git rebase -i HEAD~3` → Rebase interactivo (squash, reword, drop)
- `git cherry-pick <hash>` → Aplicar un commit específico en la rama actual
- `git stash` → Guardar cambios temporalmente
- `git stash pop` → Recuperar último stash y eliminarlo
- `git bisect start` → Iniciar búsqueda binaria de bugs
- `git bisect bad` → Marcar commit actual como malo
- `git bisect good <commit>` → Marcar commit como bueno
- `git reflog` → Historial completo de movimientos de HEAD

## 7. Diagnóstico

- `git log --oneline --graph --decorate --all` → Árbol visual completo del repo
- `git branch -vv` → Ramas con upstream y estado
- `git diff` → Cambios sin staging
- `git diff --staged` → Cambios en staging
- `git blame <archivo>` → Ver autor y commit de cada línea
- `git show-ref` → Listar todas las referencias (ramas, tags)
