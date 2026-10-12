# HTTP y APIs

En esta carpeta aprendes **el idioma que habla tu backend con el mundo**: cómo es una petición y una respuesta HTTP por dentro, qué significan los códigos de estado y los headers, cómo funcionan las cookies y CORS, cómo diseñar una API REST que no rompa a sus clientes, cómo paginar y filtrar, y cómo documentarla con OpenAPI.

-----

## Antes de empezar

No necesitas mucho: conviene haber leído [Arquitectura hexagonal](../12-arquitectura/02-arquitectura-hexagonal.md), porque la API es un adapter primario y sus DTOs no son el dominio. Esta etapa es la **teoría** que [Ktor y Koin](../14-ktor-y-koin/README.md) pone en práctica, así que puedes leerla antes o después.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. HTTP a fondo](01-http-a-fondo.md) | Anatomía de petición y respuesta, métodos, códigos de estado, headers, cookies y CORS | Nada |
| [2. Diseño REST](02-diseno-rest.md) | Recursos, acciones, `PUT` vs `PATCH`, formato JSON, versionado y cambios incompatibles | HTTP a fondo |
| [3. Paginación y filtros](03-paginacion-y-filtros.md) | Offset vs cursor, filtros tipados, orden seguro y su relación con los índices | Diseño REST y SQL |
| [4. OpenAPI y Swagger](04-openapi-y-swagger.md) | Describir la API en un spec, contract-first vs code-first, generar código y mantenerlo sincronizado | Diseño REST |

-----

## El mapa completo en una mirada

```
HTTP            ->  "petición y respuesta, en texto"
   POST /api/v1/pedidos HTTP/1.1        HTTP/1.1 201 Created
   Content-Type: application/json       Location: /api/v1/pedidos/p-9
   Authorization: Bearer eyJ...
   2xx éxito · 4xx culpa del cliente · 5xx culpa del servidor

Cookies y CORS  ->  "lo que el navegador hace por su cuenta"
   Set-Cookie: rt=...; HttpOnly; Secure; SameSite=Strict
   OPTIONS (preflight) -> Access-Control-Allow-Origin: https://app.tienda.com

REST            ->  "recursos con sustantivos, métodos con significado"
   GET /empresas/{id}/pedidos     POST /pedidos/{id}/cancelar
   agregar un campo: compatible · renombrar o quitar: /api/v2

Paginación      ->  "de a pedazos, sin perderse"
   ?limite=20&cursor=eyJ...    WHERE (creado_en, id) < (?, ?) ORDER BY ... LIMIT 21

OpenAPI         ->  "el contrato, en un archivo"
   openapi.yaml -> Swagger UI, clientes generados, tests de contrato
```

-----

## Cómo está armada cada lección

1. **En una frase:** qué es, sin jerga.
2. **Antes de empezar:** qué tienes que saber y las palabras nuevas.
3. **El contenido:** la explicación con ejemplos.
4. **Resumen en 5 líneas:** para repasar.
5. **Siguiente lección:** para seguir el orden.

-----

## Recursos

* [MDN: HTTP](https://developer.mozilla.org/es/docs/Web/HTTP): la mejor referencia de métodos, códigos, headers, cookies y CORS.
* [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
* [Especificación OpenAPI](https://spec.openapis.org/oas/latest.html)
* [Zalando RESTful API Guidelines](https://opensource.zalando.com/restful-api-guidelines/): una guía de diseño de APIs muy completa.

-----

## Después de esta carpeta

Ya sabes diseñar el contrato de tu API. Si todavía no la leíste, sigue con [Ktor y Koin](../14-ktor-y-koin/README.md). Vuelve al índice: [README principal](../README.md).
