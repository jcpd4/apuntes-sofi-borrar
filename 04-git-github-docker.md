# Git, GitHub y Docker — comandos

Repo: `jcpd4/mads-practica-2`

## Issue → rama → PR → merge

```bash
git checkout main && git pull
gh issue create --title "Bloqueo de usuarios" --body "Descripción..."   # o desde la web
git checkout -b bloqueo-usuarios

# ... test + código ...
git add .
git commit -m "Añadido método de servicio para bloquear usuarios"
git push -u origin bloqueo-usuarios

gh pr create --title "Bloqueo de usuarios" --body "Closes #N"
gh pr checks              # esperar a que GitHub Actions esté en verde
gh pr merge --merge       # (o desde la web con "Merge pull request")
git checkout main && git pull
```

Mensajes de commit como los tuyos: "Añadido X", "Añadidos tests de Y", en presente/participio, uno por paso.

## Sacar versión (si lo pide el enunciado)

1. En una rama: cambiar `<version>` en `pom.xml` (ej. `1.1.0`) y la versión/fecha en `about.html`.
2. PR + merge.
3. Tag y release:
   ```bash
   git checkout main && git pull
   git tag -a v1.1.0 -m "Versión 1.1.0"
   git push --tags
   gh release create v1.1.0 --title "v1.1.0" --notes "Qué incluye"
   ```
4. Luego "Empezamos versión X.Y.Z" (subir a `-SNAPSHOT` / siguiente versión) — como hiciste en el commit `5b592a2`.

Ver tags: `git tag` → tienes `v1.0.1` y `v1.1.0`.
Ir a una versión concreta: `git checkout v1.1.0` (o `git checkout -b examen v1.1.0`).

## Docker

Tu `Dockerfile` copia `target/*.jar` → **primero hay que generar el jar**.

```bash
./mvnw package                                     # (añade -DskipTests si ya los has pasado)
docker build -t <usuario-dockerhub>/mads-todolist:1.1.0 .
docker run -p 8080:8080 <usuario-dockerhub>/mads-todolist:1.1.0
docker login
docker push <usuario-dockerhub>/mads-todolist:1.1.0
docker image ls     # ver imágenes
docker ps           # contenedores en marcha
```

## GitHub Actions

- `.github/workflows/developer-tests.yml` → lanza `./mvnw test` en cada push (contenedor `eclipse-temurin:8-jdk`).
- Si falla el check en el PR: abrir la pestaña "Actions", ver qué test falla, arreglar, nuevo commit y push (el PR se actualiza solo).
