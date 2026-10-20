# Dónde tocar en cada capa

Estructura del proyecto (`src/main/java/madstodolist/`):

```
model/        Entidades JPA (Usuario, Tarea, Equipo)         → tablas de la BD
repository/   Interfaces CrudRepository                       → consultas
dto/          UsuarioData, RegistroData, LoginData...         → lo que viaja a la vista
service/      UsuarioService (lógica de negocio + reglas)     → @Transactional, lanza ServiceException
controller/   UsuarioController, LoginController...           → URLs, Model, comprobar permisos
controller/exception/  NotFound (404), NoAutorizado (401)...  → @ResponseStatus
authentication/ManagerUserSession                             → id del usuario logeado en la sesión
src/main/resources/templates/   Plantillas Thymeleaf
```

---

## 1. Modelo (entidad) — `model/Usuario.java`

Añadir un atributo nuevo = 3 cosas:
- el campo (con valor por defecto si es boolean: `private boolean bloqueado = false;`)
- getter (`isXxx()` si es boolean) y setter
- **lo mismo en el DTO** `dto/UsuarioData.java` (si no, ModelMapper no lo copia)
- y en `RegistroData` SOLO si se rellena desde el formulario de registro (como `admin`).

Con `ddl-auto=update/create` la columna se crea sola, no hay que tocar SQL.

## 2. Repository — `repository/UsuarioRepository.java`

*Derived queries*: Spring genera la consulta a partir del nombre del método.

| Método | Qué hace |
|---|---|
| `Optional<Usuario> findByEmail(String email)` | buscar por campo |
| `boolean existsByAdminTrue()` | ¿existe alguno con admin=true? |
| `List<Usuario> findByBloqueadoTrue()` | todos los bloqueados |
| `long countByAdminFalse()` | contar |
| `List<Usuario> findAllByOrderByEmailAsc()` | ordenar |

`findAll()` y `findById()` ya vienen de `CrudRepository`.

## 3. Service — `service/UsuarioService.java`

Patrón de un método de servicio:
1. `@Transactional` (o `@Transactional(readOnly = true)` si solo lee).
2. Buscar la entidad: `usuarioRepository.findById(id).orElse(null)`.
3. Comprobar reglas → si no se cumplen, `throw new UsuarioServiceException("mensaje")`.
4. Modificar y `save(...)`.
5. Devolver **DTO**, nunca la entidad: `modelMapper.map(usuario, UsuarioData.class)`.

Mira como ejemplo `cambiarBloqueo()` y la regla del admin único en `registrar()`.

Si cambia el login: añadir valor al enum `LoginStatus` (ej. `USER_BLOCKED`) y el `else if` en `login()`.
**Ojo al orden** de las comprobaciones en `login()` (primero existe, luego password, luego bloqueado).

## 4. Controller — `controller/UsuarioController.java`

Esqueleto de cualquier método que devuelve vista:
1. Comprobar permisos (`comprobarAdmin()` o comprobar logeado).
2. `model.addAttribute("usuario", usuario)` → **para la barra de menú**.
3. Llamar al service y añadir los datos al model.
4. `return "nombrePlantilla";`

Anotaciones clave:
- `@GetMapping("/ruta/{id}")` + `@PathVariable(value="id") Long id`
- `@PostMapping(...)` para acciones que cambian datos → terminar con `return "redirect:/ruta";`
- `@ModelAttribute` / `@Valid XxxData data, BindingResult result` para formularios.

### Excepciones → códigos HTTP (`controller/exception/`)

| Excepción | Código | Cuándo |
|---|---|---|
| `UsuarioNoLogeadoException` | 401 | no hay sesión |
| `UsuarioNoAutorizadoException` | 401 | hay sesión pero no tiene permiso |
| `UsuarioNotFoundException` | 404 | el `{id}` no existe |

Se definen con `@ResponseStatus(value = HttpStatus.XXX, reason = "...")` y `extends RuntimeException`.
Para que se vea el mensaje: `server.error.include-message=always` (ya lo tienes).

### Login — `LoginController`
- `loginSubmit()`: según `LoginStatus` → redirect o `model.addAttribute("error", "...")` + volver a `formLogin`.
- Admin al loguearse → `redirect:/registrados`; normal → `redirect:/usuarios/{id}/tareas`.
- `registroForm()` y `registroSubmit()` meten `existeAdmin` en el model (también en los caminos de error).

## 5. Vista — ver `03-thymeleaf.md`
