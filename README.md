# Manual de Git (resumen y comandos útiles)

Este documento es una versión organizada y ajustada del Manual.txt original. Contiene los comandos más usados, explicaciones breves, y pasos para casos concretos: cómo eliminar un commit que ya está subido y cómo cambiar el mensaje de un commit antiguo. También incluye un diagrama de los "workspaces" (espacios) de Git.

---

## Índice
- Inicializar repositorio
- Añadir / Eliminar archivos en staging
- Commits
- Estado / Logs / Dif
- Tags
- Ramas (branches)
- Checkout / Switch
- Merge / Rebase
- Stash
- Cherry-pick
- Remotos (remote) / Clonar / Fetch / Pull / Push
- Eliminar un commit ya subido (dos métodos)
- Cambiar nombre/mensaje de un commit viejo
- Diagrama de los tipos de workspace (espacios)
- Comandos de referencia rápida
- Buenas prácticas y advertencias

---

## Inicializar repositorio
```bash
git init                    # Inicializa repo en el directorio actual
git init NombreRepo         # Crea un nuevo repo en carpeta NombreRepo
```

## Añadir / Eliminar archivos en staging (Index)
```bash
git add -A                  # agrega todos los cambios (incluye eliminaciones)
git add .                   # agrega cambios del directorio actual
git add file                # agrega sólo un archivo específico
git add -n file             # "dry-run" (muestra lo que haría)

git rm -f file              # elimina archivo del working tree y del index
git rm --cached file        # elimina archivo del index (lo deja en working tree)
```

## Commits
```bash
git commit -m "Comentario"              # crea commit con mensaje
git commit --amend -m "Nuevo mensaje"   # modifica el último commit (cambios + mensaje)
```

## Estado / Logs / Diff
```bash
git status                     # ver estado del repo
git log                        # ver historial
git log --oneline              # resumen de commits por línea
git log --oneline --graph      # gráfico simplificado
git diff                       # ver diferencias no staged
git diff --staged              # ver diferencias staged vs último commit
git diff <sha1>                # diferencias desde commit (con working tree)
git diff <sha1> <sha2>         # diferencias entre dos commits
```

## Tags
```bash
git tag v1.0
git tag -a 0.0 -m "Comentario de la etiqueta"
git tag -l                     # listar tags
git tag <tag> <commit>         # asignar tag a commit
git tag -d 0.0                 # eliminar tag local
git tag -f -a 0.2 -m "Mensaje" <commit>  # forzar/renombrar
git push origin --tags         # subir tags al remoto
```

## Ramas (branches)
```bash
git branch nueva-rama          # crea nueva rama
git branch                     # lista ramas locales (la actual con *)
git branch -l                  # listar ramas
git branch -a                  # listar ramas (locales y remotas)
git branch -d rama             # eliminar rama (segura)
git branch -D rama             # eliminar rama (forzado)
git branch -m viejo nuevo      # renombrar rama
```

## Checkout / Switch
```bash
git checkout rama              # cambiar a rama
git checkout -b rama           # crear e ingresar a rama
git checkout <commit-id>       # mover HEAD a commit (detached HEAD)
git checkout -- file           # descartar cambios en file (volver a HEAD)
# alternativa moderna:
git switch <branch>            # cambiar de rama
git switch -c <branch>        # crear e ingresar a rama
```

## Merge / Rebase
```bash
git merge rama-que-quiero      # desde rama destino: fusiona cambios de la otra rama
git rebase rama-a-mesclar      # reescribe historia, mueve commits sobre otra rama
git rebase -i <base>           # rebase interactivo (editar, reordenar commits)
```

## Stash
```bash
git stash                      # guarda cambios temporales
git stash list                 # listar stashes
git stash apply                # aplicar último stash
git stash apply stash@{2}      # aplicar stash específico
git stash drop stash@{n}       # borrar stash específico
git stash pop                  # aplicar y eliminar el stash aplicado
```

## Cherry-pick
```bash
git cherry-pick <commit-hash>  # aplicar commit específico en la rama actual
```

## Remotos / Clonar / Fetch / Pull / Push
```bash
git remote add origin <url>     # agregar remoto
git remote -v                   # ver remotos
git remote remove origin        # eliminar enlace remoto

git clone <url>                 # clonar repo

git fetch origin master         # traer objetos y refs desde remoto
git pull origin master          # fetch + merge (o fetch + rebase si está configurado)

git push origin master          # subir commits al remoto
git push origin master -f       # push forzado (Peligro)
git push --force-with-lease     # push forzado más seguro (recomendado sobre -f)

# subir tags:
git push origin v1.0
git push origin --tags
```

---

## Cómo eliminar un commit que ya está subido al repo

Hay 2 enfoques principales: la forma segura (revertir) y la forma que reescribe la historia (reset + force). Elegir según si trabajas con otros colaboradores.

1) Forma segura — crear un nuevo commit que deshace el anterior (recomendado si ya fue compartido)
```bash
# Revertir un commit específico
git revert <commit-hash>

# Luego, empujar el nuevo commit "revert"
git push origin <branch>
```
- git revert crea un nuevo commit que invierte los cambios del commit dado. No reescribe la historia, es seguro para ramas compartidas.

2) Reescribir la historia — eliminar el commit y forzar el push (peligroso en repos compartidos)
```bash
# Mover la rama a un commit anterior (por ejemplo: 'good-commit' es el commit antes del malo)
git checkout <branch>
git reset --hard <good-commit-hash>

# Forzar el push (Usar --force-with-lease para mayor seguridad)
git push --force-with-lease origin <branch>
```
- Este método borra el commit del historial remoto. No usar si otros han basado trabajo en los commits que vas a eliminar, a menos que coordines con el equipo.

---

## Cómo cambiar el nombre (mensaje) de un commit viejo

Dependiendo de la localización del commit:

1) Cambiar el último commit (más reciente)
```bash
git commit --amend -m "Nuevo mensaje del último commit"
git push --force-with-lease origin <branch>
```
- Si el commit ya estaba en remoto, hay que forzar el push. Recomendar --force-with-lease.

2) Cambiar un commit antiguo (no necesariamente el más reciente) — rebase interactivo
```bash
# Por ejemplo, quieres editar un commit hace 3 commits:
git rebase -i HEAD~4
# En el editor, busca el commit a cambiar y reemplaza "pick" por "reword" (o "edit" si necesitas cambiar contenido).
# Guarda y cierra. Git te pedirá que edites el mensaje.
# Termina el rebase:
git rebase --continue

# Finalmente, empuja cambios reescribiendo la historia en remoto:
git push --force-with-lease origin <branch>
```
- También puedes especificar un commit base por hash: git rebase -i <commit-anterior>.

Advertencias:
- Reescribir commits ya compartidos obliga a los demás colaboradores a sincronizar (pull + rebase o reset). Comunica antes de forzar pushes.

---

## Diagrama: tipos de "workspace" (espacios) en Git

Diagrama ASCII del flujo básico:

```
Working Directory (Working Tree)
   [archivos editados, nuevos, eliminados]
             |
             |  git add <file>
             v
Staging Area (Index)  <-- archivos preparados para el commit
             |
             |  git commit
             v
Local Repository (.git)  <-- commits, objetos, referencias (HEAD, branches)
             |
             |  git push
             v
Remote Repository (origin)  <-- repositorio remoto (GitHub, GitLab...)
```

Con HEAD y ramas:

```
HEAD -> branch (e.g., master) -> commit -> commit -> commit
```

Flujo común:
- Edición -> git add -> git commit -> git push
- Para traer cambios: git fetch -> git merge (o git pull / git rebase)

---

## Comandos de referencia rápida (selección)
```bash
# Estado e historial
git status
git log --oneline --graph

# Añadir y commitear
git add <file>
git commit -m "mensaje"

# Branches
git branch -a
git checkout -b feature/x

# Traer y subir
git fetch
git pull
git push origin <branch>

# Revertir cambios
git revert <commit>
git reset --hard <commit>

# Reescribir mensaje de commit
git commit --amend -m "nuevo mensaje"
git rebase -i HEAD~N

# Forzar push (preferible con lease)
git push --force-with-lease origin <branch>
```

---

## Buenas prácticas y advertencias
- Para revertir cambios en repos compartidos, preferir `git revert` en lugar de reescribir la historia.
- Si debes forzar push, usa `--force-with-lease` para evitar sobreescribir cambios remotos inesperados.
- Comunica al equipo antes de reescribir la historia (reset/rebase + force).
- Haz backups o crea una rama temporal antes de operaciones destructivas: `git branch backup-before-reset`.
- Usa `git status` y `git log --oneline --graph` frecuentemente para entender la situación de tu repo.

---

Si quieres, puedo:
- Crear este archivo Manual.md en el repositorio (hacer commit/push).
- Incluir más ejemplos prácticos (con escenarios reales).
- Generar diagramas en formato imagen (si subes o permites) o en mermaid si prefieres.
