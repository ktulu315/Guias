# 📘 Guía Profesional de Git (Nivel Avanzado)

------------------------------------------------------------------------

# 1️⃣ Modelo Interno de Git

## Arquitectura de Datos

Git almacena:

-   **Blob** → Contenido de archivo
-   **Tree** → Estructura de directorios
-   **Commit** → Snapshot + referencia al padre
-   **Ref** → Puntero a commit (branch, tag)

### Modelo DAG (Directed Acyclic Graph)

    A --- B --- C  (main)
           \
            D --- E (feature)

Cada commit apunta a su(s) padre(s). No existen ciclos.

------------------------------------------------------------------------

# 2️⃣ Estados Internos

    Working Directory → Staging Area → Repository

-   Working Directory → Archivos editados
-   Staging Area (Index) → Preparación del commit
-   Repository → Base de datos de objetos

------------------------------------------------------------------------

# 3️⃣ Flujo Profesional (Feature Branch Workflow)

1.  `git clone URL`
2.  `git switch -c feature-x`
3.  `git add .`
4.  `git commit -m "feature"`
5.  `git push -u origin feature-x`
6.  Crear Pull Request
7.  Merge vía plataforma
8.  `git pull`
9.  `git branch -d feature-x`

------------------------------------------------------------------------

# 4️⃣ Rebase Avanzado

## Rebase simple

`git rebase main`

Reaplica commits encima de otra base.

## Rebase interactivo

`git rebase -i HEAD~3`

Permite:

-   pick → mantener
-   squash → fusionar
-   reword → editar mensaje
-   drop → eliminar commit

------------------------------------------------------------------------

# 5️⃣ Cherry Pick

Aplicar commit específico:

`git cherry-pick <hash>`

Útil para traer un cambio puntual sin hacer merge completo.

------------------------------------------------------------------------

# 6️⃣ Git Bisect (Depuración)

Buscar commit que introdujo un bug:

    git bisect start
    git bisect bad
    git bisect good <commit-bueno>

Git usa búsqueda binaria para localizar el problema.

------------------------------------------------------------------------

# 7️⃣ Reset vs Revert

## Reset (reescribe historia)

`git reset --hard HEAD~1`

-   Mueve puntero de rama
-   Puede borrar commits

## Revert (seguro en producción)

`git revert <commit>`

-   Crea nuevo commit inverso
-   No reescribe historial

------------------------------------------------------------------------

# 8️⃣ HEAD y Referencias

-   HEAD → Puntero actual
-   refs/heads/main → Rama
-   refs/tags/v1.0 → Tag

Ver referencias:

`git show-ref`

------------------------------------------------------------------------

# 9️⃣ Diagnóstico Completo

    git status
    git branch -vv
    git log --oneline --graph --decorate --all

------------------------------------------------------------------------

# 🔟 Recuperación Avanzada

## Reflog

`git reflog`

Permite recuperar commits borrados accidentalmente.

Recuperar commit:

`git checkout <hash>`

------------------------------------------------------------------------

# 1️⃣1️⃣ Errores Comunes

  Error                    Causa
  ------------------------ ----------------------
  non-fast-forward         Remoto adelantado
  refspec does not match   Rama no existe
  detached HEAD            No estás en rama
  unrelated histories      Dos raíces distintas

------------------------------------------------------------------------

# 1️⃣2️⃣ Buenas Prácticas

-   No usar `--force` en ramas compartidas
-   Preferir `--force-with-lease`
-   Commits pequeños y descriptivos
-   Usar ramas para features
-   Pull frecuente antes de push

------------------------------------------------------------------------

# 🧠 Resumen Conceptual

Git es una base de datos distribuida de snapshots inmutables conectados
en un grafo acíclico dirigido (DAG).

Las ramas son simplemente punteros móviles.
