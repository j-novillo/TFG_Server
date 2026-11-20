# NOMADA · Server (REST API)

> **Final Degree Project (TFG)** of the Higher Vocational Training Program in Cross-Platform
> Application Development (DAM), IES Francisco de Quevedo, Madrid, 2024‑2025.
> **Collaborative project** by a team of 5, organized with **Scrum**.
>
> This repository contains the **backend**. The Android app lives in
> **[TFG_Client](https://github.com/j-novillo/TFG_Client)**.

![Java](https://img.shields.io/badge/Java_21-ED8B00?logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot_3.4-6DB33F?logo=springboot&logoColor=white)
![Spring Security](https://img.shields.io/badge/Spring_Security_+_JWT-6DB33F?logo=springsecurity&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?logo=mysql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?logo=githubactions&logoColor=white)

---

## What is NOMADA?

NOMADA is a **social network of geolocated notes**. Users can leave notes pinned to a spot on the
map (a restaurant, a landscape, a monument, an event…), rate them, make them public, private or
visible only to their followers, and discover the notes other people have left around them.

The idea comes from a simple question: *what if people's memories and recommendations were tied
to the place where they happened?* NOMADA answers it by combining an interactive map with social
features (following users, likes, saved notes).

The system has two parts:

```
┌─────────────────────────┐   HTTPS + JWT   ┌──────────────────────────┐      ┌─────────┐
│  Android app (Kotlin)   │ ──────────────► │  REST API (Spring Boot)  │ ───► │  MySQL  │
│  Jetpack Compose · MVVM │ ◄────────────── │  this repository         │      └─────────┘
└──────────┬──────────────┘      JSON       └────────────┬─────────────┘
           │                                             │ SMTP
           ▼                                             ▼
   Firebase (Auth, Storage,                    Account verification
   Crashlytics) · Google Maps                  email
```

## What the server does

- **Authentication and authorization**: sign-up, login and session renewal with **JWT**
  (30-minute access token and 7-day refresh token), passwords hashed with BCrypt and a custom
  security filter on top of Spring Security.
- **Account verification by email**: after signing up, users receive an email with a
  confirmation link. The account is not activated until they click it.
- **Geolocated notes** with type inheritance (*classic, event, historical, food, landscape,
  cultural*): create, delete, rate (1‑10), and query by map area, by type, by age or sorted.
- **Per-note privacy**: `PUBLIC`, `FOLLOWERS` or `PRIVATE`.
- **Social network**: follow and unfollow, followers and following, likes and saved notes.
- **User profiles** and user search.
- **Typed domain errors** (note not found, note owned by someone else, rating out of range,
  trying to follow yourself…) translated into consistent HTTP responses with a shared `ApiError`.

## Architecture

**Layered** architecture, separating presentation, domain and data access:

```
src/main/java/org/server
├── ui/            # REST controllers and DTOs (contract with the client)
│   ├── controllers/   Authentication, Notas, Social, Follow, User
│   └── model/         NoteDTO, EventNoteDTO, NoteMapDTO, UserDTO...
├── domain/        # Business logic
│   ├── service/       AuthenticationService, NoteService, SocialService, FollowService, UserService
│   └── errors/        Domain exceptions + ExceptionHandlers (@ControllerAdvice)
├── dao/           # Persistence with Spring Data JPA
│   ├── model/         Entities: User, Note (+ subtypes), likes, saved notes, follows
│   └── repositories/
├── security/      # Spring Security: configuration, token filter and JWT service
├── common/        # Configuration and entity ↔ DTO mapping
└── utils/         # Email sending (MailComponent)
```

**Notable technical decisions**

- **JPA single-table inheritance** for note types: a single `notes` table with a `note_type`
  discriminator. Events add start and end dates.
- **The authenticated user is always taken from the `SecurityContext`**, never from a request
  parameter, so nobody can act on behalf of another user.
- **DTOs don't expose sensitive data**: the password was removed from `UserDTO`.
- **UUID identifiers** for users.

## API (summary)

| Resource | Main endpoints |
|---|---|
| Authentication | `POST /login` · `POST /register` · `GET /refresh` · `GET /confirm` · `GET /firebase_id` |
| Notes `/notes` | `GET /all` · `GET /{noteId}` · `GET /area` · `GET /type` · `GET /sorted` · `GET /antiquity` · `POST /addNota` · `PATCH /{id}/rate` · `DELETE /{id}` |
| Social `/social` | `POST /like` · `DELETE /liked` · `GET /likes` · `GET /saves` · `POST /saves` · `DELETE /saves` |
| Followers `/users` | `POST /follow` · `DELETE /unfollow` · `GET /followers` · `GET /following` · `GET /is-following` · `GET /{username}/followers` · `GET /{username}/following` |
| Users `/user` | `GET /info/{username}` · `GET /notes` · `GET /notes/{username}` |

## Quality and deployment

- **Unit tests** with **JUnit 5 + Mockito** for every domain service
  (`AuthenticationService`, `NoteService`, `SocialService`, `FollowService`, `UserService`),
  plus `.http` collections to test the authentication flows by hand.
- **Docker**: multi-stage `Dockerfile` (builds with Maven + JDK 21 and runs on a lightweight JRE 21).
- **CI/CD with GitHub Actions**: every push to `main` builds the image and publishes it to Docker Hub.
- In production the API was deployed on the school's server, with the app consuming it over HTTPS.

## How to run it

Requirements: **JDK 21** and a **MySQL** database. The schema is in
`src/main/resources/schema.sql` and there is sample data in `data.sql`.

1. In `src/main/resources/application.properties` (or through environment variables), set the
   MySQL connection, the JWT signing key and the SMTP credentials.
2. Start the application:

   ```bash
   ./mvnw spring-boot:run
   ```

   Or with Docker:

   ```bash
   docker build -t nomada-server .
   docker run -p 8080:8080 nomada-server
   ```

3. Tests:

   ```bash
   ./mvnw test
   ```

## Team and methodology

Built by **Jorge Novillo**, **Adrián Saavedra**, **Marcos Sánchez**, **Mario Valiente** and
**Aitor García**, following **Scrum** (user-story backlog, sprints, planning, reviews and
retrospectives, managed in Taiga) and a **GitFlow** workflow with feature branches and
Pull Requests reviewed by the team.

**[docs/METHODOLOGY.md](docs/METHODOLOGY.md)** explains how we organized ourselves and what each
person worked on.