# PlatziFlixe — Arquitectura del Sistema

---

## Levantar el entorno local

### Requisitos previos
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado y corriendo
- Puerto `8000` (API) y `5432` (PostgreSQL) libres en tu máquina

### Primera vez (o entorno limpio)

```bash
# 1. Ir a la carpeta del backend
cd Backend

# 2. Construir imágenes y levantar contenedores en background
docker-compose up -d --build

# 3. Ejecutar migraciones de base de datos
docker-compose exec api bash -c "cd /app && uv run alembic -c app/alembic.ini upgrade head"

# 4. Cargar datos de ejemplo (3 cursos, 3 profesores, 6 lecciones)
docker-compose exec api bash -c "cd /app && uv run python -m app.db.seed"
```

### Uso cotidiano (ya construido)

```bash
cd Backend

docker-compose up -d        # Iniciar
docker-compose down         # Detener
docker-compose logs -f      # Ver logs en tiempo real
docker-compose restart      # Reiniciar
```

### Verificar que funciona

```bash
# Health check — debe responder {"status":"ok","database":true,"courses_count":3}
curl http://localhost:8000/health

# Lista de cursos
curl http://localhost:8000/courses

# Detalle de un curso
curl http://localhost:8000/courses/curso-de-react
```

También puedes abrir la documentación interactiva en el navegador:
- **Swagger UI:** `http://localhost:8000/docs`
- **ReDoc:** `http://localhost:8000/redoc`

### Reiniciar datos de ejemplo

```bash
# Limpiar datos y recargar desde cero
docker-compose exec api bash -c "cd /app && uv run python -m app.db.seed clear"
docker-compose exec api bash -c "cd /app && uv run python -m app.db.seed"
```

### Limpiar todo (contenedores + volúmenes + imágenes)

```bash
docker-compose down -v --rmi all --remove-orphans
```

### Credenciales de la base de datos

| Parámetro | Valor |
|-----------|-------|
| Host | `localhost:5432` |
| Base de datos | `platziflix_db` |
| Usuario | `platziflix_user` |
| Contraseña | `platziflix_password` |

---

## Resumen Ejecutivo

PlatziFlix es una plataforma de cursos online compuesta por **tres proyectos independientes** que comparten la misma API:

| Proyecto | Tecnología | Rol |
|----------|-----------|-----|
| **Backend** | Python + FastAPI + PostgreSQL | API REST + Lógica de negocio |
| **Frontend** | Next.js 15 + React 19 + TypeScript | Aplicación web SSR |
| **Mobile Android** | Kotlin + Jetpack Compose | App nativa Android (MVVM) |
| **Mobile iOS** | Swift + SwiftUI | App nativa iOS (MVVM) |

---

## 1. Diagrama de Arquitectura General (Big Picture)

```mermaid
graph TB
    subgraph Clientes["Clientes"]
        WEB["🌐 Frontend\nNext.js 15 / React 19\nlocalhost:3000"]
        AND["🤖 Android\nKotlin + Jetpack Compose\n10.0.2.2:8000"]
        IOS["🍎 iOS\nSwift + SwiftUI\nlocalhost:8000"]
    end

    subgraph Backend["Backend (FastAPI)"]
        API["FastAPI\nUvicorn ASGI\n:8000"]
        SVC["CourseService\nLógica de Negocio"]
        ORM["SQLAlchemy ORM\n+ Alembic Migrations"]
    end

    subgraph DB["Base de Datos"]
        PG[("PostgreSQL 15\nplatziflix_db")]
    end

    WEB -->|"HTTP GET\n/courses\n/courses/{slug}\n/classes/{id}"| API
    AND -->|"HTTP GET\nRetrofit + OkHttp"| API
    IOS -->|"HTTP GET\nURLSession + Combine"| API

    API --> SVC
    SVC --> ORM
    ORM --> PG

    style Clientes fill:#1a1a2e,stroke:#e94560,color:#fff
    style Backend fill:#16213e,stroke:#0f3460,color:#fff
    style DB fill:#0f3460,stroke:#533483,color:#fff
```

---

## 2. Diagrama de Clases — Backend (Modelos de Datos)

```mermaid
classDiagram
    class BaseModel {
        +Integer id
        +DateTime created_at
        +DateTime updated_at
        +DateTime deleted_at
    }

    class Course {
        +String name
        +Text description
        +String thumbnail
        +String slug
        +List~Teacher~ teachers
        +List~Lesson~ lessons
    }

    class Teacher {
        +String name
        +String email
        +List~Course~ courses
    }

    class Lesson {
        +Integer course_id
        +String name
        +Text description
        +String slug
        +String video_url
        +Course course
    }

    class CourseTeacher {
        <<association>>
        +Integer course_id
        +Integer teacher_id
    }

    BaseModel <|-- Course
    BaseModel <|-- Teacher
    BaseModel <|-- Lesson
    Course "1" --> "N" Lesson : contiene
    Course "N" --> "M" Teacher : course_teachers
    CourseTeacher --> Course
    CourseTeacher --> Teacher
```

---

## 3. Diagrama de Clases — Frontend (TypeScript Types)

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
    CourseDetail "1" --> "N" Class : contiene
    Quiz "1" --> "N" QuizOption : tiene
```

---

## 4. Diagrama de Clases — Mobile Android

```mermaid
classDiagram
    class Course {
        <<Domain Model>>
        +Int id
        +String name
        +String description
        +String thumbnail
        +String slug
    }

    class CourseDTO {
        <<Data Transfer Object>>
        +Int id
        +String name
        +String description
        +String thumbnail
        +String slug
        +String createdAt
        +String updatedAt
        +String deletedAt
        +List~Int~ teacherIds
    }

    class CourseRepository {
        <<interface>>
        +getCourses() Flow~List~Course~~
    }

    class RemoteCourseRepository {
        -ApiService apiService
        -CourseMapper mapper
        +getCourses() Flow~List~Course~~
    }

    class MockCourseRepository {
        +getCourses() Flow~List~Course~~
    }

    class CourseListViewModel {
        -CourseRepository repository
        +StateFlow~CourseListUiState~ uiState
        +handleEvent(CourseListUiEvent)
    }

    class CourseListUiState {
        +Boolean isLoading
        +List~Course~ courses
        +String errorMessage
    }

    class ApiService {
        <<Retrofit Interface>>
        +getAllCourses() Response~List~CourseDTO~~
    }

    CourseRepository <|.. RemoteCourseRepository : implements
    CourseRepository <|.. MockCourseRepository : implements
    CourseListViewModel --> CourseRepository
    CourseListViewModel --> CourseListUiState
    RemoteCourseRepository --> ApiService
    RemoteCourseRepository ..> CourseDTO : usa
    CourseDTO ..> Course : mapper transforma
```

---

## 5. Diagrama de Clases — Mobile iOS

```mermaid
classDiagram
    class Course {
        <<Domain Model: Identifiable>>
        +Int id
        +String name
        +String description
        +String thumbnail
        +String slug
        +[Int] teacherIds
        +Bool isActive
        +String displayDescription
    }

    class CourseDTO {
        <<Codable>>
        +Int id
        +String name
        +String description
        +String thumbnail
        +String slug
        +String createdAt
        +String updatedAt
        +String deletedAt
        +[Int] teacherIds
    }

    class CourseRepositoryProtocol {
        <<Protocol>>
        +getCourses() async throws [Course]
        +getCourseBySlug(String) async throws Course
    }

    class RemoteCourseRepository {
        -NetworkManager networkManager
        +getCourses() async throws [Course]
        +getCourseBySlug(String) async throws Course
    }

    class CourseListViewModel {
        <<MainActor ObservableObject>>
        +@Published courses [Course]
        +@Published isLoading Bool
        +@Published errorMessage String?
        +fetchCourses()
        +searchCourses(String)
    }

    class NetworkManager {
        <<Singleton>>
        +shared NetworkManager
        +request(APIEndpoint) async throws T
    }

    class APIEndpoint {
        <<Protocol>>
        +baseURL String
        +path String
        +method HTTPMethod
        +headers [String: String]
    }

    CourseRepositoryProtocol <|.. RemoteCourseRepository : conforms
    CourseListViewModel --> CourseRepositoryProtocol
    RemoteCourseRepository --> NetworkManager
    NetworkManager ..> APIEndpoint : usa
    CourseDTO ..> Course : mapper transforma
```

---

## 6. Flujo de Datos — Listado de Cursos

```mermaid
sequenceDiagram
    participant U as Usuario
    participant C as Cliente (Web/App)
    participant API as FastAPI :8000
    participant SVC as CourseService
    participant DB as PostgreSQL

    U->>C: Abre la aplicación
    C->>API: GET /courses
    API->>SVC: get_all_courses()
    SVC->>DB: SELECT * FROM courses WHERE deleted_at IS NULL
    DB-->>SVC: Lista de cursos
    SVC-->>API: List[CourseDict]
    API-->>C: JSON Array [{id, name, description, thumbnail, slug}]
    C-->>U: Renderiza lista de cursos
```

---

## 7. Flujo de Datos — Detalle de Curso

```mermaid
sequenceDiagram
    participant U as Usuario
    participant C as Cliente
    participant API as FastAPI :8000
    participant SVC as CourseService
    participant DB as PostgreSQL

    U->>C: Selecciona un curso
    C->>API: GET /courses/{slug}
    API->>SVC: get_course_by_slug(slug)
    SVC->>DB: SELECT courses + JOIN teachers + JOIN lessons\nWHERE slug=? AND deleted_at IS NULL
    DB-->>SVC: Curso + Profesores + Lecciones
    SVC-->>API: CourseDetailDict
    API-->>C: JSON {id, name, slug, teacher_id[], classes[]}
    C-->>U: Renderiza detalle del curso

    alt Curso no encontrado
        API-->>C: 404 {"detail": "Course not found"}
        C-->>U: Página 404
    end
```

---

## 8. Flujo de la Aplicación Frontend (Next.js App Router)

```mermaid
flowchart TD
    START([Usuario abre la web]) --> HOME

    HOME["📄 / — página principal\nsrc/app/page.tsx\nServer Component async"]
    HOME -->|"fetch GET /courses"| BACKEND_LIST["Backend API\nGET /courses"]
    BACKEND_LIST -->|"JSON []"| HOME
    HOME --> RENDER_LIST["Renderiza CourseCard\npor cada curso"]

    RENDER_LIST -->|"Click en curso"| COURSE

    COURSE["📄 /course/[slug]\nsrc/app/course/[slug]/page.tsx\nServer Component async"]
    COURSE -->|Suspense fallback| LOADING["⏳ loading.tsx\nSpinner animado"]
    COURSE -->|Error en fetch| ERROR["❌ error.tsx\nError boundary + retry"]
    COURSE -->|Curso no existe| NOTFOUND["🔍 not-found.tsx\nPágina 404"]
    COURSE -->|"fetch GET /courses/{slug}"| BACKEND_DETAIL["Backend API\nGET /courses/{slug}"]
    BACKEND_DETAIL -->|JSON| COURSE
    COURSE --> RENDER_DETAIL["Renderiza CourseDetail\ncon lista de clases"]

    RENDER_DETAIL -->|"Click en clase"| CLASS

    CLASS["📄 /classes/[class_id]\nsrc/app/classes/[class_id]/page.tsx\nServer Component async"]
    CLASS -->|"fetch GET /classes/{id}"| BACKEND_CLASS["Backend API\nGET /classes/{id}"]
    BACKEND_CLASS -->|JSON| CLASS
    CLASS --> VIDEOPLAYER["🎬 VideoPlayer\nsrc/components/VideoPlayer\n<video> HTML5 nativo"]
```

---

## 9. Flujo MVVM — Android (Jetpack Compose)

```mermaid
flowchart LR
    subgraph Presentation["Presentation Layer"]
        SCREEN["CourseListScreen\n@Composable"]
        VM["CourseListViewModel\nStateFlow<UiState>"]
    end

    subgraph Domain["Domain Layer"]
        REPO_I["CourseRepository\ninterface"]
    end

    subgraph Data["Data Layer"]
        REPO["RemoteCourseRepository"]
        MAPPER["CourseMapper\nDTO → Domain"]
        API["ApiService\nRetrofit Interface"]
        NET["NetworkModule\nOkHttpClient"]
    end

    SCREEN -->|"handleEvent()"| VM
    VM -->|"collect StateFlow"| SCREEN
    VM -->|"getCourses()"| REPO_I
    REPO_I -->|implements| REPO
    REPO --> API
    REPO --> MAPPER
    API --> NET
    NET -->|"HTTP GET :8000"| BACKEND[("FastAPI\nBackend")]
```

---

## 10. Flujo MVVM — iOS (SwiftUI)

```mermaid
flowchart LR
    subgraph Presentation["Presentation Layer"]
        VIEW["CourseListView\nSwiftUI View"]
        VM["CourseListViewModel\n@MainActor\n@Published properties"]
    end

    subgraph Domain["Domain Layer"]
        PROTO["CourseRepositoryProtocol\nSwift Protocol"]
    end

    subgraph Data["Data Layer"]
        REPO["RemoteCourseRepository"]
        MAPPER["CourseMapper\nDTO → Domain"]
        NET["NetworkManager\nSingleton\nURLSession"]
        ENDPOINTS["CourseAPIEndpoints\nenum APIEndpoint"]
    end

    VIEW -->|"@StateObject"| VM
    VM -->|"@Published → body re-render"| VIEW
    VM -->|"getCourses()"| PROTO
    PROTO -->|conforms| REPO
    REPO --> NET
    REPO --> MAPPER
    NET --> ENDPOINTS
    ENDPOINTS -->|"HTTP GET :8000"| BACKEND[("FastAPI\nBackend")]
```

---

## 11. Diagrama de Base de Datos (ERD)

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
    COURSES ||--o{ COURSE_TEACHERS : "asignado en"
    TEACHERS ||--o{ COURSE_TEACHERS : "imparte en"
```

---

## 12. Comparativa de Stacks por Plataforma

| Aspecto | Backend | Frontend | Android | iOS |
|--------|---------|----------|---------|-----|
| **Lenguaje** | Python 3.11 | TypeScript 5 | Kotlin | Swift 5 |
| **Framework** | FastAPI 0.104 | Next.js 15.3 | Jetpack Compose | SwiftUI |
| **Patrón** | Service Layer | Server Components | MVVM + Clean | MVVM + Clean |
| **BD/Estado** | PostgreSQL 15 | SSR (sin estado) | StateFlow | @Published + Combine |
| **HTTP Client** | — | fetch nativo | Retrofit 2.9 | URLSession |
| **Serialización** | Pydantic | TypeScript types | Gson | Codable |
| **Imágenes** | — | next/image | Coil 2.5 | AsyncImage |
| **Testing** | pytest (10 tests) | Vitest + RTL | pendiente | pendiente |
| **Contenedor** | Docker + compose | — | — | — |
| **Auth** | ❌ No implementado | ❌ No implementado | ❌ No implementado | ❌ No implementado |

---

## 13. Puntos Destacados y Observaciones

### Fortalezas
- **Arquitectura Clean consistente** en móvil: ambas plataformas siguen MVVM + Data/Domain/Presentation
- **Backend minimalista y correcto**: 4 endpoints bien definidos, sin over-engineering
- **Mappers explícitos**: DTO → Domain en Android e iOS, evitan acoplar la API al dominio
- **Soft Delete** en todos los modelos del backend (campo `deleted_at`)
- **Server Components** en Next.js: rendimiento óptimo sin estado en cliente
- **Type safety end-to-end**: TypeScript en frontend, data classes en Kotlin, structs en Swift

### Gaps / Trabajo Pendiente
- **Autenticación**: ninguna capa la implementa aún
- **Endpoints faltantes**: Frontend/Mobile llaman a `/classes/{id}` pero el backend no lo expone
- **Sin caché local** en móvil (Room / CoreData)
- **Paginación**: la API devuelve todos los cursos sin paginación
- **Sin CRUD**: solo lectura (GET), no hay creación ni edición
- **Types sin uso** en Frontend: `Progress`, `Quiz`, `FavoriteToggle` — features futuras

### Convenciones de URL del Backend
| Cliente | Base URL configurada |
|---------|---------------------|
| Frontend | `http://localhost:8000` |
| Android (emulador) | `http://10.0.2.2:8000` |
| iOS | `http://localhost:8000` |
