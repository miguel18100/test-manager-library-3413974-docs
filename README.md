# Manager Library · Documentación

Especificación y decisiones de *Manager Library*, un sistema para que una biblioteca pequeña lleve su
catálogo, sus socios y sus préstamos, y vea qué libros se prestan más.

| Documento | Contenido |
|---|---|
| [`spec-python/`](spec-python/) | El spec: constitución, historias de usuario, contrato de la API, modelo de datos, arquitectura, plan, tareas y ADR |
| [`spec-.net/`](spec-.net/) | La misma especificación con el vocabulario de .NET |
| [`adaptacion-react-laravel.md`](adaptacion-react-laravel.md) | Cómo se lleva el spec a Laravel y React, decisiones propias y trazabilidad historia → código → prueba |

El negocio (HU-01 a HU-10, RN-1 a RN-6, contrato de la API y modelo de datos) es el mismo para cualquier
lenguaje. El modelo de datos vive solo en `data-model.md` y el contrato HTTP solo en `api-contract.md`.
