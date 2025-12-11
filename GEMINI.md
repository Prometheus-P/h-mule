# Gemini Project: Human Mule (가칭)

## Project Overview

**Human Mule** is a B2B logistics platform for express delivery of documents and small items across Seoul, utilizing the subway system. It aims to be a crowd-sourced service that is cheaper than traditional quick-service and faster than standard parcel delivery.

The project is a mobile-first application with a web-based component. The front-end is built with **Flutter**, and the back-end is a **Go** API server. The database is **PostgreSQL**. The project emphasizes a "SQL-first" approach using `sqlc` for type-safe Go code generation from SQL queries.

The architecture is designed to be scalable, with a clear separation of concerns between the front-end, back-end, and infrastructure. It includes plans for background job processing using `Asynq` and a CI/CD pipeline with GitHub Actions.

## Building and Running

The following commands are based on the initial project documentation. The actual commands may differ once the project structure is fully in place.

### Backend (Go)

To run the backend server, navigate to the backend directory and run the main application:

```bash
# Navigate to the backend application directory
cd apps/backend

# Run the API server
go run ./cmd/api
```

### Mobile/Web (Flutter)

To run the mobile or web application, navigate to the mobile directory and use the `flutter run` command:

```bash
# Navigate to the mobile application directory
cd apps/mobile

# Run the application on a selected simulator or device
flutter run
```

### Testing

The project uses the following commands for testing:

- **Go:** `go test ./...`
- **Flutter:** `flutter test`

Linting is also configured:

- **Go:** `golangci-lint`
- **Dart/Flutter:** `flutter analyze`

## Development Conventions

The project has a clear set of development conventions outlined in the documentation.

- **Issue-driven development:** All work should start with a GitHub Issue.
- **Branching strategy:** The project uses a `main` / `develop` / `feature/*` / `hotfix/*` branching model.
- **Commit messages:** Commits should follow the Conventional Commits specification (e.g., `feat:`, `fix:`, `docs:`).
- **Pull Requests:** PRs should be small (200-300 lines) and include a summary of the work and test results.
- **AI-assisted development:** The project encourages the use of AI for code generation, but all AI-generated code must be reviewed by a human before being merged.
- **Language:**
    - **Documentation:** Korean
    - **Code, comments, identifiers, and commit messages:** English
