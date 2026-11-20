# Team Organization and Methodology — NOMADA (Final Degree Project)

This document is shared by both project repositories:
[TFG_Client](https://github.com/j-novillo/TFG_Client) (Android app) and
[TFG_Server](https://github.com/j-novillo/TFG_Server) (REST API).

## Context

NOMADA is the **Final Degree Project (TFG) of the Higher Vocational Training Program in
Cross-Platform Application Development (DAM)**, completed at IES Francisco de Quevedo (Madrid), 2024‑2025 academic year.
It is a **collaborative project by five people**, developed between **March and June 2025**.

| Member | GitHub |
|---|---|
| Jorge Novillo | [@j-novillo](https://github.com/j-novillo) (formerly `jorjitoNM`) |
| Adrián Saavedra | [@cronoxxxx](https://github.com/cronoxxxx) |
| Marcos Sánchez | [@Marcossanchez149](https://github.com/Marcossanchez149) |
| Mario Valiente | [@MarioValiente2](https://github.com/MarioValiente2) |
| Aitor García | [@aitor345674](https://github.com/aitor345674) |

## Scrum

The team followed **Scrum**, adapted to a student team with a fixed delivery deadline:

- **Product Backlog**: the app's features (sign-up and login, geolocated notes, map, profiles,
  followers, likes and saved notes, images, filters…) were written as user stories and
  prioritized by their value for the minimum viable product.
- **Short sprints**: work was split into iterations. Each one had to end with a working,
  demoable increment on the `develop` branch.
- **Sprint Planning**: at the start of each sprint, stories were picked from the backlog and
  assigned to team members. Most stories involved changes to both the client and the server.
- **Daily follow-up**: short meetings to share progress, blockers and dependencies between the
  client and the API (for example, agreeing on an endpoint's contract before consuming it from the app).
- **Sprint Review**: review of the increment at the end of each sprint to validate what was built
  and adjust backlog priorities.
- **Retrospective**: what to improve in the way we worked (branch conventions, PR size, how to
  resolve merge conflicts…).

## Project Management with Taiga

To apply Scrum we used **[Taiga](https://taiga.io/)**, an agile project management tool. The
project can be viewed at
**[tree.taiga.io/project/jorgen-tfg](https://tree.taiga.io/project/jorgen-tfg/timeline)**.

Taiga was the hub for the whole work cycle:

- **Backlog**: the product's user stories, prioritized.
- **Sprints**: each sprint was assigned backlog stories with their tasks and owners.
- **Task board**: the status of each task (to do, in progress, done) was visible at a glance,
  and it was the reference point in our follow-up meetings.
- **Timeline**: the team's activity history, useful for tracking how the project evolved.

Git branches mapped to Taiga user stories, so each feature could be traced from the task to the
Pull Request that merged it.

## Git Workflow (GitFlow)

Both repositories follow a **GitFlow** model:

```
main  ←─ stable releases / final delivery
  └── develop  ←─ continuous integration during the sprint
        ├── feature/<feature>      (one branch per user story)
        ├── fix / hotfix-<issue>   (bug fixes)
        └── ...
```

- Each user story was developed on its own `feature-*` branch
  (for example `feature-register-user`, `feature-like-note`, `feature-note-images`,
  `feature-DarkMode`, `featureFiltrarPorTipoDeNote`…).
- Integration was done through **Pull Requests** against `develop`, reviewed by the rest of the team.
- `develop` was merged into `main` at milestones and for the final delivery.
- In the server repository, every push to `main` triggers a **GitHub Actions workflow** that
  builds the Docker image and publishes it to Docker Hub, ready to deploy.

Some figures from the history:

| | Client | Server |
|---|---|---|
| Commits | ~250 | ~150 |
| Pull Requests | 42 | 33 |
| Feature branches | 30+ | 25+ |

## Division of Work

Stories were split by feature rather than by layer, so most members worked on both the Android
app and the API. **Jorge Novillo**'s contribution (one of the main contributors to both
repositories) focused on:

- **Project bootstrap**: initial skeleton of the client and the server (layer structure,
  model, dependencies and `.gitignore`).
- **Security and sessions** (server): Spring Security with **access and refresh JWTs**, login,
  sign-up, refresh endpoint, token filter and retrieving the user from the `SecurityContext`.
- **Account verification by email** (server): sending an email with a confirmation link via SMTP.
- **API error handling**: `ApiError` and exception handlers to return consistent error responses.
- **Client authentication**: Start, Login and Sign-up screens with Jetpack Compose, token
  management, OkHttp interceptor and authenticator, and Firebase Auth integration.
- **Images**: uploading and downloading note images and profile pictures with Firebase Storage,
  and displaying them on the map and in the detail view.
- **Likes**, **dark mode**, login and sign-up tests, resolving integration conflicts and
  upgrading dependencies.
