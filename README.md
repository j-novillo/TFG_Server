# NOMADA · Servidor (API REST)

> **Trabajo de Fin de Grado (TFG)** del CFGS en Desarrollo de Aplicaciones Multiplataforma (DAM),
> IES Francisco de Quevedo, Madrid, curso 2024‑2025.
> **Proyecto colaborativo** de 5 personas organizado con **Scrum**.
>
> Este repositorio contiene el **backend**. La app Android está en
> **[TFG_Client](https://github.com/j-novillo/TFG_Client)**.

![Java](https://img.shields.io/badge/Java_21-ED8B00?logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot_3.4-6DB33F?logo=springboot&logoColor=white)
![Spring Security](https://img.shields.io/badge/Spring_Security_+_JWT-6DB33F?logo=springsecurity&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?logo=mysql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?logo=githubactions&logoColor=white)

---

## ¿Qué es NOMADA?

NOMADA es una **red social de notas geolocalizadas**. Cada usuario puede dejar notas asociadas a un
punto del mapa (un restaurante, un paisaje, un monumento, un evento…), valorarlas, hacerlas
públicas, privadas o visibles solo para sus seguidores, y descubrir las notas que otras personas
han dejado a su alrededor.

La idea surge de una pregunta sencilla: *¿y si los recuerdos y recomendaciones de la gente
estuvieran ligados al lugar donde ocurrieron?* NOMADA lo resuelve combinando un mapa interactivo
con funciones sociales (seguir usuarios, likes, guardados).

El sistema tiene dos piezas:

```
┌─────────────────────────┐   HTTPS + JWT   ┌──────────────────────────┐      ┌─────────┐
│  App Android (Kotlin)   │ ──────────────► │  API REST (Spring Boot)  │ ───► │  MySQL  │
│  Jetpack Compose · MVVM │ ◄────────────── │  este repositorio        │      └─────────┘
└──────────┬──────────────┘      JSON       └────────────┬─────────────┘
           │                                             │ SMTP
           ▼                                             ▼
   Firebase (Auth, Storage,                    Email de verificación
   Crashlytics) · Google Maps                  de la cuenta
```

## Qué hace el servidor

- **Autenticación y autorización**: registro, login y renovación de sesión con **JWT**
  (token de acceso de 30 min y token de refresco de 7 días), contraseñas hasheadas con BCrypt y un filtro de
  seguridad propio sobre Spring Security.
- **Verificación de cuenta por email**: al registrarse, el usuario recibe un correo con un
  enlace de confirmación. La cuenta no se activa hasta que lo pulsa.
- **Notas geolocalizadas** con herencia de tipos (*clásica, evento, histórica, gastronómica,
  paisaje, cultural*): crear, borrar, valorar (1‑10), consultar por área del mapa, por tipo,
  por antigüedad u ordenadas.
- **Privacidad por nota**: `PUBLIC`, `FOLLOWERS` o `PRIVATE`.
- **Red social**: seguir y dejar de seguir, seguidores y seguidos, likes y notas guardadas.
- **Perfiles de usuario** y búsqueda de otros usuarios.
- **Errores de dominio tipados** (nota no encontrada, nota ajena, valoración fuera de rango,
  intento de seguirse a uno mismo…) que se traducen a respuestas HTTP coherentes con un `ApiError` común.

## Arquitectura

Arquitectura **en capas**, separando presentación, dominio y acceso a datos:

```
src/main/java/org/server
├── ui/            # Controladores REST y DTOs (contrato con el cliente)
│   ├── controllers/   Authentication, Notas, Social, Follow, User
│   └── model/         NoteDTO, EventNoteDTO, NoteMapDTO, UserDTO...
├── domain/        # Lógica de negocio
│   ├── service/       AuthenticationService, NoteService, SocialService, FollowService, UserService
│   └── errors/        Excepciones de dominio + ExceptionHandlers (@ControllerAdvice)
├── dao/           # Persistencia con Spring Data JPA
│   ├── model/         Entidades: User, Note (+ subtipos), likes, guardados, follows
│   └── repositories/
├── security/      # Spring Security: configuración, filtro de tokens y servicio JWT
├── common/        # Configuración y mapeo entidad ↔ DTO
└── utils/         # Envío de emails (MailComponent)
```

**Decisiones técnicas destacables**

- **Herencia JPA (single table)** para los tipos de nota: una sola tabla `notes` con
  discriminador `note_type`. Los eventos añaden fechas de inicio y fin.
- **El usuario autenticado se obtiene siempre del `SecurityContext`**, nunca de un parámetro
  de la petición, para que nadie pueda actuar en nombre de otro usuario.
- **Los DTOs no exponen datos sensibles**: la contraseña se eliminó del `UserDTO`.
- **Identificadores UUID** para los usuarios.

## API (resumen)

| Recurso | Endpoints principales |
|---|---|
| Autenticación | `POST /login` · `POST /register` · `GET /refresh` · `GET /confirm` · `GET /firebase_id` |
| Notas `/notes` | `GET /all` · `GET /{noteId}` · `GET /area` · `GET /type` · `GET /sorted` · `GET /antiquity` · `POST /addNota` · `PATCH /{id}/rate` · `DELETE /{id}` |
| Social `/social` | `POST /like` · `DELETE /liked` · `GET /likes` · `GET /saves` · `POST /saves` · `DELETE /saves` |
| Seguidores `/users` | `POST /follow` · `DELETE /unfollow` · `GET /followers` · `GET /following` · `GET /is-following` · `GET /{username}/followers` · `GET /{username}/following` |
| Usuarios `/user` | `GET /info/{username}` · `GET /notes` · `GET /notes/{username}` |

## Calidad y despliegue

- **Tests unitarios** con **JUnit 5 + Mockito** para todos los servicios de dominio
  (`AuthenticationService`, `NoteService`, `SocialService`, `FollowService`, `UserService`),
  más colecciones `.http` para probar los flujos de autenticación a mano.
- **Docker**: `Dockerfile` multi‑stage (compila con Maven + JDK 21 y ejecuta sobre un JRE 21 ligero).
- **CI/CD con GitHub Actions**: cada push a `main` construye la imagen y la publica en Docker Hub.
- En producción la API estaba desplegada en el servidor del centro, con la app consumiéndola por HTTPS.

## Cómo ejecutarlo

Requisitos: **JDK 21** y una base de datos **MySQL**. El esquema está en
`src/main/resources/schema.sql` y hay datos de ejemplo en `data.sql`.

1. Configura en `src/main/resources/application.properties` (o mediante variables de entorno)
   la conexión a MySQL, la clave de firma de los JWT y las credenciales SMTP.
2. Arranca la aplicación:

   ```bash
   ./mvnw spring-boot:run
   ```

   O con Docker:

   ```bash
   docker build -t nomada-server .
   docker run -p 8080:8080 nomada-server
   ```

3. Tests:

   ```bash
   ./mvnw test
   ```

## Equipo y metodología

Proyecto desarrollado por **Jorge Novillo**, **Adrián Saavedra**, **Marcos Sánchez**,
**Mario Valiente** y **Aitor García**, siguiendo **Scrum** (backlog de historias de usuario,
sprints, planning, reviews y retrospectivas) y un flujo **GitFlow** con *feature branches*
y *Pull Requests* revisadas por el equipo.

En **[docs/METODOLOGIA.md](docs/METODOLOGIA.md)** se explica cómo nos organizamos y qué parte
desarrolló cada uno.
