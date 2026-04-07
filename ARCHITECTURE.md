# Platziflix — Arquitectura del Sistema

## Resumen Ejecutivo

**Platziflix** es una plataforma educativa de video streaming compuesta por tres proyectos independientes que comparten una API REST como fuente de datos:

| Proyecto | Tecnología | Propósito |
|----------|-----------|-----------|
| **Backend** | Python 3.11 + FastAPI + PostgreSQL | API REST + base de datos |
| **Frontend** | Next.js 15 + React 19 + TypeScript | Aplicación web |
| **Mobile/Android** | Kotlin + Jetpack Compose | App Android |
| **Mobile/iOS** | Swift + SwiftUI | App iOS |

---

## 1. Big Picture — Vista General del Sistema

```mermaid
graph TB
    subgraph CLIENTS["Clientes"]
        WEB["🌐 Web\nNext.js 15\nlocalhost:3000"]
        ANDROID["🤖 Android\nKotlin + Compose\n10.0.2.2:8000"]
        IOS["🍎 iOS\nSwift + SwiftUI\nlocalhost:8000"]
    end

    subgraph BACKEND["Backend (Docker)"]
        API["⚡ FastAPI\nUvicorn ASGI\n:8000"]
        SVC["🔧 CourseService\nBusiness Logic"]
        ORM["🗄️ SQLAlchemy\nORM Layer"]
        ALEMBIC["📦 Alembic\nMigrations"]
    end

    subgraph DATABASE["Base de Datos (Docker)"]
        PG["🐘 PostgreSQL 15\n:5432\nplatziflix_db"]
    end

    WEB -- "HTTP REST" --> API
    ANDROID -- "HTTP REST\nRetrofit" --> API
    IOS -- "HTTP REST\nURLSession" --> API

    API --> SVC
    SVC --> ORM
    ORM --> PG
    ALEMBIC -.-> PG

    classDef client fill:#4A90D9,stroke:#2C5F8A,color:#fff
    classDef backend fill:#27AE60,stroke:#1A7A40,color:#fff
    classDef db fill:#E67E22,stroke:#A85A18,color:#fff
    class WEB,ANDROID,IOS client
    class API,SVC,ORM,ALEMBIC backend
    class PG db
```

---

## 2. Diagrama de Clases — Backend

```mermaid
classDiagram
    class BaseModel {
        <<abstract>>
        +Integer id PK
        +DateTime created_at
        +DateTime updated_at
        +DateTime deleted_at
    }

    class Course {
        +String name
        +String description
        +Text thumbnail
        +String slug UNIQUE
        +List~Teacher~ teachers
        +List~Lesson~ lessons
    }

    class Teacher {
        +String name
        +String email UNIQUE
        +List~Course~ courses
    }

    class Lesson {
        +Integer course_id FK
        +String name
        +String description
        +String slug
        +String video_url
        +Course course
    }

    class course_teachers {
        <<table>>
        +Integer course_id FK
        +Integer teacher_id FK
    }

    class CourseService {
        -Session db
        +__init__(db: Session)
        +get_all_courses() List~Dict~
        +get_course_by_slug(slug: str) Dict
    }

    class Settings {
        +str project_name
        +str version
        +str database_url
    }

    BaseModel <|-- Course
    BaseModel <|-- Teacher
    BaseModel <|-- Lesson
    Course "many" -- "many" Teacher : course_teachers
    Course "1" -- "many" Lesson
    CourseService ..> Course : queries
    CourseService ..> Lesson : eager loads
```

---

## 3. Diagrama de Clases — Frontend (TypeScript)

```mermaid
classDiagram
    class Course {
        +number id
        +string title
        +string teacher
        +number duration
        +string thumbnail
        +string slug
    }

    class CourseDetail {
        +string description
        +Class[] classes
    }

    class Class {
        +number id
        +string title
        +string description
        +string video
        +number duration
        +string slug
    }

    class Progress {
        +number progress
        +number user_id
    }

    class Quiz {
        +number id
        +string question
        +QuizOption[] options
    }

    class QuizOption {
        +number id
        +string answer
        +boolean correct
    }

    class FavoriteToggle {
        +number course_id
    }

    CourseDetail --|> Course : extends
    CourseDetail "1" *-- "many" Class
    Quiz "1" *-- "many" QuizOption
```

---

## 4. Diagrama de Clases — Android (Kotlin)

```mermaid
classDiagram
    direction TB

    class Course {
        +Int id
        +String name
        +String description
        +String thumbnail
        +String slug
    }

    class CourseDTO {
        +Int id
        +String name
        +String description
        +String thumbnail
        +String slug
        +String? createdAt
        +String? updatedAt
        +String? deletedAt
        +List~Int~? teacherIds
    }

    class CourseRepository {
        <<interface>>
        +getAllCourses() Result~List~Course~~
    }

    class RemoteCourseRepository {
        -ApiService apiService
        +getAllCourses() Result~List~Course~~
    }

    class MockCourseRepository {
        +getAllCourses() Result~List~Course~~
    }

    class ApiService {
        <<interface>>
        +getAllCourses() Response~List~CourseDTO~~
    }

    class CourseMapper {
        <<object>>
        +fromDTO(dto: CourseDTO) Course
        +fromDTOList(dtos: List~CourseDTO~) List~Course~
    }

    class CourseListUiState {
        +Boolean isLoading
        +List~Course~ courses
        +String? error
        +Boolean isRefreshing
    }

    class CourseListViewModel {
        -CourseRepository courseRepository
        -StateFlow~CourseListUiState~ _uiState
        +StateFlow~CourseListUiState~ uiState
        +handleEvent(event: CourseListUiEvent)
        -loadCourses()
        -refreshCourses()
    }

    class AppModule {
        <<object>>
        +provideApiService() ApiService
        +provideCourseRepository() CourseRepository
        +provideCourseListViewModel() CourseListViewModel
    }

    CourseRepository <|.. RemoteCourseRepository
    CourseRepository <|.. MockCourseRepository
    RemoteCourseRepository --> ApiService
    RemoteCourseRepository --> CourseMapper
    CourseMapper ..> CourseDTO
    CourseMapper ..> Course
    CourseListViewModel --> CourseRepository
    CourseListViewModel --> CourseListUiState
    AppModule ..> CourseListViewModel
    AppModule ..> RemoteCourseRepository
```

---

## 5. Diagrama de Clases — iOS (Swift)

```mermaid
classDiagram
    class Course {
        +Int id
        +String name
        +String description
        +String thumbnail
        +String slug
        +Int[] teacherIds
        +Date? createdAt
        +Date? updatedAt
        +Date? deletedAt
        +Bool isActive
        +String displayDescription
    }

    class Teacher {
        +Int id
        +String name
        +String email
    }

    class ClassModel {
        +Int id
        +String name
        +String description
        +String slug
    }

    class CourseDTO {
        +Int id
        +String name
        +String description
        +String thumbnail
        +String slug
        +String? createdAt
        +String? deletedAt
        +Int[]? teacherId
    }

    class CourseRepositoryProtocol {
        <<protocol>>
        +getAllCourses() async throws Course[]
        +getCourseBySlug(slug: String) async throws Course
    }

    class RemoteCourseRepository {
        -NetworkService networkService
        +getAllCourses() async throws Course[]
        +getCourseBySlug(slug: String) async throws Course
    }

    class NetworkService {
        <<protocol>>
        +request(endpoint: APIEndpoint) async throws Data
    }

    class NetworkManager {
        +static shared NetworkManager
        -URLSession urlSession
        +request(endpoint: APIEndpoint) async throws Data
    }

    class APIEndpoint {
        <<protocol>>
        +baseURL: String
        +path: String
        +method: HTTPMethod
        +headers: Dictionary?
    }

    class CourseAPIEndpoints {
        <<enum>>
        getAllCourses
        getCourseBySlug(String)
    }

    class NetworkError {
        <<enum>>
        invalidURL
        networkUnavailable
        timeout
        requestFailed(statusCode: Int)
        decodingError(Error)
        unknown(Error)
    }

    class CourseListViewModel {
        +Course[] courses
        +Bool isLoading
        +String? errorMessage
        +String searchText
        +Course[] filteredCourses
        +Bool isEmpty
        +loadCourses() async
        +refreshCourses() async
    }

    CourseRepositoryProtocol <|.. RemoteCourseRepository
    NetworkService <|.. NetworkManager
    APIEndpoint <|.. CourseAPIEndpoints
    RemoteCourseRepository --> NetworkService
    RemoteCourseRepository --> CourseDTO
    CourseListViewModel --> CourseRepositoryProtocol
```

---

## 6. Flujo de Datos — Usuario Web

```mermaid
sequenceDiagram
    actor Usuario
    participant Web as Next.js (SSR)
    participant API as FastAPI
    participant DB as PostgreSQL

    Usuario->>Web: Visita /
    Web->>API: GET /courses
    API->>DB: SELECT * FROM courses WHERE deleted_at IS NULL
    DB-->>API: rows[]
    API-->>Web: JSON [{id, name, slug, thumbnail...}]
    Web-->>Usuario: HTML con grid de cursos

    Usuario->>Web: Click en curso
    Web->>API: GET /courses/{slug}
    API->>DB: SELECT con JOIN lessons y teachers
    DB-->>API: course + lessons + teachers
    API-->>Web: JSON {course + classes[]}
    Web-->>Usuario: HTML con detalle del curso

    Usuario->>Web: Click en clase
    Web->>API: GET /classes/{id}
    API->>DB: SELECT lesson
    DB-->>API: lesson row
    API-->>Web: JSON {video_url, name...}
    Web-->>Usuario: HTML con VideoPlayer
```

---

## 7. Flujo de Datos — App Móvil (Android/iOS)

```mermaid
sequenceDiagram
    actor Usuario
    participant VM as ViewModel
    participant Repo as Repository
    participant Net as Network Layer
    participant API as FastAPI

    Usuario->>VM: App Launch
    VM->>VM: init → loadCourses()
    VM->>VM: state = isLoading: true
    VM->>Repo: getAllCourses()
    Repo->>Net: GET /courses
    Net->>API: HTTP Request

    alt Respuesta exitosa
        API-->>Net: 200 [{id, name, slug...}]
        Net-->>Repo: Data descodificada (DTO[])
        Repo->>Repo: Mapper: DTO → Domain
        Repo-->>VM: Result.success([Course])
        VM->>VM: state = courses: [...], isLoading: false
        VM-->>Usuario: Lista de cursos renderizada
    else Error de red
        API-->>Net: Error / Timeout
        Net-->>Repo: Exception
        Repo-->>VM: Result.failure(error)
        VM->>VM: state = error: mensaje
        VM-->>Usuario: Mensaje de error + botón Retry
    end

    Usuario->>VM: Pull to Refresh
    VM->>VM: state = isRefreshing: true
    VM->>Repo: getAllCourses()
    Note over VM,API: mismo flujo...
```

---

## 8. Flujo de Capas — Backend

```mermaid
flowchart TD
    REQ[HTTP Request] --> ROUTE[FastAPI Route\nmain.py]
    ROUTE --> DI[Dependency Injection\nget_db + get_course_service]
    DI --> SVC[CourseService\nBusiness Logic]
    SVC --> QJOIN{Necesita\nrelaciones?}
    QJOIN -- Sí\nget_course_by_slug --> JLOAD[joinedload\nteachers + lessons]
    QJOIN -- No\nget_all_courses --> SIMPLE[Query simple\nfilter deleted_at IS NULL]
    JLOAD --> ORM[SQLAlchemy ORM]
    SIMPLE --> ORM
    ORM --> PG[(PostgreSQL)]
    PG --> ORM
    ORM --> MAP[Mapeo a Dict]
    MAP --> RESP[JSON Response]

    style REQ fill:#4A90D9,color:#fff
    style RESP fill:#27AE60,color:#fff
    style PG fill:#E67E22,color:#fff
```

---

## 9. Arquitectura de Capas — Comparativa

```mermaid
graph LR
    subgraph BACKEND["Backend (Layered)"]
        direction TB
        B1["API Layer\nRoutes + Endpoints"]
        B2["Service Layer\nCourseService"]
        B3["Data Access\nSQLAlchemy + Sessions"]
        B4["Model Layer\nCourse/Teacher/Lesson"]
        B5["Config Layer\npydantic-settings"]
        B1 --> B2 --> B3 --> B4
        B5 -.-> B1
    end

    subgraph FRONTEND["Frontend (Next.js)"]
        direction TB
        F1["Pages (SSR)\napp/page.tsx"]
        F2["Components\nCourse, VideoPlayer"]
        F3["API Calls\nfetch()"]
        F4["Types\nTypeScript interfaces"]
        F1 --> F3
        F1 --> F2
        F2 --> F4
    end

    subgraph MOBILE["Mobile (Clean Architecture)"]
        direction TB
        M1["Presentation\nViewModel + Views"]
        M2["Domain\nModels + Repository Interface"]
        M3["Data\nRepositories + DTOs + Mappers"]
        M4["Network\nHTTP Client + Endpoints"]
        M1 --> M2 --> M3 --> M4
    end

    FRONTEND -- "REST\nHTTP" --> BACKEND
    MOBILE -- "REST\nHTTP" --> BACKEND

    classDef layer fill:#2C3E50,stroke:#ECF0F1,color:#fff
```

---

## 10. Modelo de Datos — Base de Datos

```mermaid
erDiagram
    COURSES {
        int id PK
        varchar name
        text description
        varchar thumbnail
        varchar slug UK
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    TEACHERS {
        int id PK
        varchar name
        varchar email UK
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    LESSONS {
        int id PK
        int course_id FK
        varchar name
        text description
        varchar slug
        varchar video_url
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    COURSE_TEACHERS {
        int course_id FK
        int teacher_id FK
    }

    COURSES ||--o{ LESSONS : "tiene"
    COURSES ||--o{ COURSE_TEACHERS : "pertenece a"
    TEACHERS ||--o{ COURSE_TEACHERS : "enseña en"
```

---

## 11. Infraestructura Docker

```mermaid
graph TB
    subgraph DOCKER["Docker Compose Network"]
        subgraph API_CONTAINER["Contenedor: api"]
            UVICORN["Uvicorn ASGI Server\n:8000"]
            FASTAPI["FastAPI App\n/app/main.py"]
            UVICORN --> FASTAPI
        end

        subgraph DB_CONTAINER["Contenedor: db"]
            POSTGRES["PostgreSQL 15\n:5432\nplatziflix_db"]
            VOL[("Volume\npostgres_data")]
            POSTGRES --> VOL
        end

        API_CONTAINER -- "postgresql://\nplatziflix_user@db:5432" --> DB_CONTAINER
    end

    HOST["Host Machine"] -- ":8000" --> API_CONTAINER
    HOST -- ":5432" --> DB_CONTAINER

    DEV["Desarrollo\n./app volume mount\nhot reload"] -.-> API_CONTAINER
```

---

## 12. Resumen Técnico por Proyecto

### Backend
- **Framework:** FastAPI 0.104+ con Uvicorn (ASGI)
- **ORM:** SQLAlchemy 2.0+ con Alembic para migraciones
- **DB:** PostgreSQL 15
- **Patrón:** Layered Architecture + Soft Delete + Dependency Injection
- **Endpoints activos:** `GET /`, `GET /health`, `GET /courses`, `GET /courses/{slug}`
- **Infra:** Docker + Docker Compose, gestor `uv`

### Frontend
- **Framework:** Next.js 15.3 (App Router) + React 19 + TypeScript strict
- **Estilos:** SCSS Modules + sistema de tokens de color
- **Renderizado:** Server Components por defecto (SSR), `no-store` cache
- **Testing:** Vitest + React Testing Library
- **Estado:** Sin state management — data fetching en servidor
- **Features preparadas:** Progress, Quiz, Favorites (tipos definidos, no implementados)

### Mobile Android
- **Lenguaje/UI:** Kotlin + Jetpack Compose + Material 3
- **Patrón:** MVVM + MVI + Clean Architecture (3 capas: Data/Domain/Presentation)
- **Red:** Retrofit + OkHttp + Gson
- **Async:** Coroutines + StateFlow
- **DI:** Manual via AppModule

### Mobile iOS
- **Lenguaje/UI:** Swift + SwiftUI
- **Patrón:** MVVM + Clean Architecture (3 capas: Data/Domain/Presentation)
- **Red:** URLSession nativo + Codable + Async/Await
- **Reactivo:** @Published + @ObservableObject + Combine
- **DI:** Manual via constructor injection
- **Extra:** Accesibilidad (VoiceOver), Design System propio

---

## 13. API Contract — Endpoints Requeridos

| Método | Endpoint | Usado por | Respuesta |
|--------|----------|-----------|-----------|
| `GET` | `/courses` | Web, Android, iOS | `[{id, name, description, thumbnail, slug}]` |
| `GET` | `/courses/{slug}` | Web | `{...course, teacher_id[], classes[]}` |
| `GET` | `/classes/{id}` | Web | `{id, title, description, video, duration, slug}` |
| `GET` | `/health` | Monitoring | `{status, service, version, database, courses_count}` |

> **Nota:** Los proyectos Mobile solo consumen `GET /courses`. El frontend consume todos los endpoints.
