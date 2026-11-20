# Organización del equipo y metodología — NOMADA (TFG)

Este documento es común a los dos repositorios del proyecto:
[TFG_Client](https://github.com/j-novillo/TFG_Client) (app Android) y
[TFG_Server](https://github.com/j-novillo/TFG_Server) (API REST).

## Contexto

NOMADA es el **Trabajo de Fin de Grado del Ciclo Formativo de Grado Superior en
Desarrollo de Aplicaciones Multiplataforma (DAM)**, cursado en el IES Francisco de Quevedo (Madrid), curso 2024‑2025.
Es un proyecto **colaborativo de cinco personas**, desarrollado entre **marzo y junio de 2025**
(primer commit el 22/03/2025, entrega el 09/06/2025).

| Integrante | GitHub |
|---|---|
| Jorge Novillo | [@j-novillo](https://github.com/j-novillo) (antes `jorjitoNM`) |
| Adrián Saavedra | [@cronoxxxx](https://github.com/cronoxxxx) |
| Marcos Sánchez | [@Marcossanchez149](https://github.com/Marcossanchez149) |
| Mario Valiente | [@MarioValiente2](https://github.com/MarioValiente2) |
| Aitor García | [@aitor345674](https://github.com/aitor345674) |

## Scrum

El equipo se organizó siguiendo **Scrum**, adaptado a un equipo de estudiantes con un plazo
de entrega cerrado:

- **Product Backlog**: las funcionalidades de la aplicación (registro y login, notas
  geolocalizadas, mapa, perfiles, seguidores, likes y guardados, imágenes, filtros…) se
  escribieron como historias de usuario y se priorizaron según su valor para el producto mínimo.
- **Sprints cortos**: el trabajo se dividió en iteraciones. Al final de cada una había que tener
  un incremento funcional en la rama `develop` que se pudiera enseñar.
- **Sprint Planning**: al empezar cada sprint se escogían las historias del backlog y se
  repartían entre los miembros. Cada historia solía implicar cambios en cliente y servidor a la vez.
- **Seguimiento diario**: reuniones breves para poner en común avances, bloqueos y
  dependencias entre cliente y API (por ejemplo, acordar el contrato de un endpoint antes
  de consumirlo desde la app).
- **Sprint Review**: revisión del incremento al cerrar cada sprint para validar lo construido
  y ajustar las prioridades del backlog.
- **Retrospectiva**: qué mejorar en la forma de trabajar (convenciones de ramas, tamaño de las
  PR, cómo resolver conflictos de merge…).

## Flujo de trabajo con Git (GitFlow)

Los dos repositorios siguen un modelo **GitFlow**:

```
main  ←─ versiones estables / entrega
  └── develop  ←─ integración continua del sprint
        ├── feature/<funcionalidad>   (una rama por historia de usuario)
        ├── fix / hotfix-<problema>   (correcciones)
        └── ...
```

- Cada historia de usuario se desarrollaba en su propia rama `feature-*`
  (por ejemplo `feature-register-user`, `feature-like-note`, `feature-note-images`,
  `feature-DarkMode`, `featureFiltrarPorTipoDeNote`…).
- La integración se hacía mediante **Pull Requests** contra `develop`, revisadas por el resto del equipo.
- `develop` se pasaba a `main` al cerrar hitos y para la entrega final.
- En el repositorio del servidor, cada push a `main` lanza un **workflow de GitHub Actions** que
  construye la imagen Docker y la publica en Docker Hub, listo para desplegarse.

Algunas cifras del histórico:

| | Cliente | Servidor |
|---|---|---|
| Commits | ~250 | ~150 |
| Pull Requests | 42 | 33 |
| Ramas de funcionalidad | 30+ | 25+ |

## Reparto de trabajo

Las historias se repartían por funcionalidad y no por capa, así que la mayoría de los miembros
trabajó tanto en la app Android como en la API. La contribución de **Jorge Novillo**
(uno de los principales contribuidores en ambos repositorios) se centró en:

- **Arranque del proyecto**: esqueleto inicial de cliente y servidor (estructura de capas,
  modelo, dependencias y `.gitignore`).
- **Seguridad y sesión** (servidor): Spring Security con **JWT de acceso y de refresco**, login,
  registro, endpoint de refresh, filtro de tokens y obtención del usuario desde el `SecurityContext`.
- **Verificación de cuenta por correo electrónico** (servidor): envío de un email con enlace de
  confirmación a través de SMTP.
- **Gestión de errores de la API**: `ApiError` y manejadores de excepciones para devolver
  respuestas de error coherentes.
- **Autenticación en el cliente**: pantallas de Start, Login y Registro con Jetpack Compose,
  gestión de tokens, interceptor y autenticador de OkHttp e integración con Firebase Auth.
- **Imágenes**: subida y descarga de imágenes de notas y de foto de perfil con Firebase Storage,
  y su visualización en el mapa y en el detalle.
- **Likes**, **modo oscuro**, tests de login y registro, resolución de conflictos de integración
  y actualización de dependencias.
