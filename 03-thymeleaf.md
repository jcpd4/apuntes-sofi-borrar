# Thymeleaf — chuleta

Plantillas en `src/main/resources/templates/`. Nombre devuelto por el controller = nombre del `.html`.

## Estructura de una página nueva (copia de `listaUsuarios.html`)

```html
<head th:replace="fragments :: head (titulo='Título')"></head>
<body>
  <th:block th:if="${usuario != null}">  <div th:replace="fragments :: navbar"></div> </th:block>
  <th:block th:unless="${usuario != null}"> <div th:replace="fragments :: navbarNoLogeado"></div> </th:block>
  ... contenido ...
  <div th:replace="fragments::javascript"/>
</body>
```

## Expresiones que más se usan

| Para... | Sintaxis |
|---|---|
| Texto | `th:text="${u.email}"` |
| Texto condicional | `th:text="${u.bloqueado} ? 'Bloqueado' : 'Activo'"` |
| Concatenar | `th:text="'Cerrar sesión ' + ${usuario.nombre}"` |
| Bucle | `<tr th:each="u : ${usuarios}">` |
| Mostrar si | `th:if="${usuario.admin}"` / `th:unless="${u.admin}"` |
| Enlace con id | `th:href="@{/registrados/{id}(id=${u.id})}"` |
| Clase condicional | `th:classappend="${u.bloqueado} ? 'btn-success' : 'btn-danger'"` |
| Mensaje de error | `<div th:if="${error}" th:text="${error}"></div>` |

## Botón que cambia estado (POST, patrón PRG)

```html
<form method="post" th:action="@{/registrados/{id}/bloqueo(id=${u.id})}" class="d-inline" th:unless="${u.admin}">
    <button type="submit" class="btn btn-sm" th:text="...">...</button>
</form>
```
Controller: `@PostMapping` → hace el cambio → `return "redirect:/registrados";`
¿Por qué? Si recargas (F5) no se repite el POST.

## Barra de menú — `fragments.html`

- Fragmento `navbar` usa `${usuario.id}`, `${usuario.nombre}`, `${usuario.admin}` → **el controller tiene que meter `usuario` en el model SIEMPRE**.
- Enlace solo para admin: `<li class="nav-item" th:if="${usuario.admin}"> ... </li>`.

## Formulario con checkbox / campos (registro)

- El objeto del formulario: `th:object="${registroData}"`, campos con `th:field="*{admin}"`.
- Ocultar checkbox si ya hay admin: `th:unless="${existeAdmin}"` (línea 42 de `formRegistro.html`) → por eso el controller mete `existeAdmin`.

## Versión en "Acerca de" — `about.html`
Ahí se cambia el número de versión y la fecha de release cuando sacas versión.
