# Examen práctica 2 — Checklist rápido

> Lee primero este fichero. Los demás son la chuleta por temas:
> - `01-donde-tocar-cada-capa.md` → qué fichero cambiar para cada cosa
> - `02-tests.md` → patrones de tests (repository, service, web)
> - `03-thymeleaf.md` → plantillas, barra de menú, formularios POST
> - `04-git-github-docker.md` → issues, ramas, PR, tags, Docker

---

## 0. Antes de empezar (2 min)

- [ ] Leer el enunciado ENTERO y subrayar: qué funcionalidad, desde qué versión se parte, qué se entrega (PR, tag, imagen Docker...).
- [ ] ¿Parto de `main` o de la versión de la práctica 2? Tus tags: `v1.0.1`, `v1.1.0` (la práctica 2 es **v1.1.0**). Ojo: tu `main` ya tiene cosas de la práctica 3 (equipos, PostgreSQL).
- [ ] `git status` limpio y `git pull` en `main`.
- [ ] `./mvnw test` → que todo esté en verde ANTES de tocar nada.

## 1. Flujo de trabajo que se evalúa (igual que en la práctica)

1. **Issue** en GitHub con la funcionalidad.
2. **Rama** con nombre descriptivo (`git checkout -b nombre-funcionalidad`).
3. **Commits pequeños** y con mensaje claro (idealmente TDD: test → código → commit).
4. **Push** y **Pull Request** que cierre el issue (`Closes #N` en la descripción).
5. Comprobar que **GitHub Actions** pasa (workflow `Developer tests`).
6. **Merge** del PR a `main`.
7. Si lo piden: **cambiar versión** (pom.xml + about.html), **tag** + **release**, **imagen Docker**.

## 2. Orden recomendado para implementar una funcionalidad (de abajo a arriba)

```
Modelo (entidad)  →  Repository  →  Service  →  Controller  →  Plantilla HTML
   test repo          test repo      test service   test web       test web
```

Por cada capa: escribir test → que falle → implementar → que pase → commit.

## 3. Lo que más suele caer (busca esto en tu código)

| Si te piden... | Mira en tu práctica | Fichero de referencia |
|---|---|---|
| Nuevo atributo en el usuario (p.ej. `bloqueado`) | Entidad + DTO + plantilla | `model/Usuario.java`, `dto/UsuarioData.java` |
| Nueva página solo para admin | `comprobarAdmin()` + excepción 401 | `controller/UsuarioController.java` |
| Página de detalle con `{id}` y 404 si no existe | `descripcionUsuario()` | `UsuarioController.java` + `UsuarioNotFoundException` |
| Botón que cambia un estado | Form POST + redirect (PRG) | `cambiarBloqueo()` + `listaUsuarios.html` |
| Regla de negocio ("solo un admin", "no bloquear admin") | Lanzar `UsuarioServiceException` en el service | `service/UsuarioService.java` |
| Consulta nueva en BD | *Derived query* en el repository | `UsuarioRepository.existsByAdminTrue()` |
| Cambiar el login (redirigir, nuevo error) | `LoginStatus` + `loginSubmit()` | `UsuarioService.login()`, `LoginController` |
| Mostrar algo en la barra de menú según rol | `th:if="${usuario.admin}"` | `templates/fragments.html` |
| Checkbox en formulario de registro | `RegistroData` + `existeAdmin` en el modelo | `formRegistro.html`, `LoginController.registroForm()` |

## 4. Errores típicos que te han pasado / que pasan

- **No añadir `usuario` al `Model`** → la barra de menú (`fragments :: navbar`) peta con `${usuario.id}` nulo.
- **Mockito devuelve `0L` y no `null`** para `managerUserSession.usuarioLogeado()` si no le dices nada → en el test "sin login" pon explícitamente `when(...).thenReturn(null)`.
- **Olvidar `@Sql(scripts = "/clean-db.sql")`** en tests web → datos de un test contaminan otro. Si añades una tabla nueva, añádela también a `clean-db.sql` (en orden: primero tablas hijas/join).
- **Usar `<a>` (GET) para cambiar estado** → tiene que ser `<form method="post">` + `redirect:`.
- **Usar `@GetMapping` donde el test hace `post(...)`** o al revés → 405 Method Not Allowed.
- **Atributo nuevo en la entidad pero no en el DTO** → ModelMapper no lo copia y en la vista sale vacío.
- **Olvidar `@Transactional`** en el método nuevo del service.
- **No hacer commit antes del merge / olvidar el push del tag** (`git push --tags`).

## 5. Comandos que vas a usar sí o sí

```bash
./mvnw test                               # todos los tests
./mvnw test -Dtest=UsuarioServiceTest     # una clase
./mvnw test -Dtest=UsuarioServiceTest#cambiarBloqueoAlternaEstado   # un test
./mvnw spring-boot:run                    # arrancar → http://localhost:8080/login
```
Consola H2 (en dev): http://localhost:8080/h2-console  (JDBC URL: `jdbc:h2:mem:dev`)
