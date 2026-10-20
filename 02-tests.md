# Tests — los 3 tipos que usas

Estructura **GIVEN / WHEN / THEN** en todos. Nombre del test = lo que comprueba
(ej. `cambiarBloqueoAdminLanzaExcepcion`).

---

## 1. Tests de repository/modelo — `repository/UsuarioTest.java`

- `@SpringBootTest` + `@Sql(scripts = "/clean-db.sql")`
- Inyectan `UsuarioRepository` y prueban entidad + consultas.
- Los que acceden a BD: `@Transactional` en el test.
- Ejemplo a imitar: `comprobarSiExisteAdmin()` (prueba la *derived query*).

## 2. Tests de servicio — `service/UsuarioServiceTest.java`

- Inyectan `UsuarioService`, trabajan con DTOs (`UsuarioData`).
- Comprobar resultado: `assertThat(x).isEqualTo(...)`, `.isTrue()`, `.isNull()`, `.hasSize(n)`.
- **Comprobar excepción:**
  ```java
  Assertions.assertThrows(UsuarioServiceException.class, () -> {
      usuarioService.metodo(...);
  });
  ```
  (mira cómo lo hiciste en `registrarSegundoAdminLanzaExcepcion` y `cambiarBloqueoAdminLanzaExcepcion`)

## 3. Tests web (controller + vista) — `controller/UsuariosRegistradosWebTest.java`

Cabecera de la clase:
```java
@SpringBootTest
@AutoConfigureMockMvc
@Sql(scripts = "/clean-db.sql")
public class XxxWebTest {
    @Autowired private MockMvc mockMvc;
    @Autowired private UsuarioService usuarioService;
    @MockBean  private ManagerUserSession managerUserSession;   // simula quién está logeado
```

### Simular login
```java
when(managerUserSession.usuarioLogeado()).thenReturn(usuario.getId());  // logeado
when(managerUserSession.usuarioLogeado()).thenReturn(null);             // NO logeado (¡explícito!)
```
Tienes el helper `logearAdmin()` al final de `UsuariosRegistradosWebTest` → cópialo si lo necesitas.

### Peticiones y comprobaciones (chuleta)

| Quiero comprobar... | Código |
|---|---|
| GET | `mockMvc.perform(get("/registrados"))` |
| POST | `mockMvc.perform(post("/registrados/" + id + "/bloqueo"))` |
| POST con parámetros de formulario | `post("/login").param("eMail", "a@ua").param("password", "123")` |
| HTML contiene texto | `.andExpect(content().string(containsString("Bloquear")))` |
| Varias cosas | `content().string(allOf(containsString("a"), containsString("b")))` |
| NO contiene | `content().string(not(containsString("claveSecreta")))` |
| Enlace concreto | `containsString("href=\"/registrados/" + id + "\"")` |
| 401 | `.andExpect(status().isUnauthorized())` |
| 404 | `.andExpect(status().isNotFound())` |
| 200 | `.andExpect(status().isOk())` |
| Redirección | `.andExpect(status().is3xxRedirection()).andExpect(redirectedUrl("/registrados"))` |
| Cambio en BD tras el POST | `assertThat(usuarioService.findById(id).isBloqueado()).isTrue();` |

Imports estáticos que necesitas (cópialos de tu test):
`MockMvcRequestBuilders.*`, `MockMvcResultMatchers.*`, `org.hamcrest.Matchers.*`,
`org.mockito.Mockito.when`, `org.assertj.core.api.Assertions.assertThat`.

## Qué tests escribir para una funcionalidad "solo admin" (plantilla mental)

1. Admin logeado → ve la página / el dato correcto (200 + `containsString`).
2. Sin login → 401.
3. Usuario normal → 401.
4. `{id}` que no existe → 404.
5. Si es acción POST → redirige + el cambio está en BD.
6. Si hay regla de negocio → test de servicio con `assertThrows`.
7. Si cambia la barra de menú → enlace aparece para admin / no aparece para normal.

## Lección que aprendiste (te puede salvar en el examen)

Si no configuras el mock, `usuarioLogeado()` devuelve **`0L`**, no `null`.
Por eso en el test sin login hay que poner `thenReturn(null)` y por eso `comprobarAdmin()`
comprueba también que `findById` no devuelva `null`.
