# CLAUDE.md — Platziflix

Guía de contexto para Claude Code. Lee este archivo al inicio de cada sesión de desarrollo.

> Arquitectura detallada, diagramas de clases, ER y flujos de datos: ver [`ARCHITECTURE.md`](./ARCHITECTURE.md)

---

## Qué es este proyecto

**Platziflix** es una plataforma educativa de video streaming. Contiene **cuatro sub-proyectos independientes** en el mismo repositorio que comparten una API REST como única fuente de datos:

```
/
├── Backend/        Python + FastAPI + PostgreSQL  (API REST)
├── Frontend/       Next.js 15 + React 19          (Web)
├── Mobile/
│   ├── PlatziFlixAndroid/   Kotlin + Jetpack Compose
│   └── PlatziFlixiOS/       Swift + SwiftUI
└── ARCHITECTURE.md          Diagramas completos del sistema
```

---

## Cómo levantar cada proyecto

### Backend (entrada principal para desarrollo)
```bash
cd Backend

make start            # Levanta Docker Compose (api + postgres)
make migrate          # Corre migraciones Alembic
make seed             # Inserta datos de prueba
make logs             # Tail de logs
make stop             # Detiene contenedores
make clean            # Elimina contenedores + volúmenes + imágenes
```

- API disponible en `http://localhost:8000`
- PostgreSQL en `localhost:5432` (usuario: `platziflix_user`, pass: `platziflix_password`, db: `platziflix_db`)
- Hot reload habilitado — los cambios en `Backend/app/` se reflejan sin reiniciar

### Frontend
```bash
cd Frontend

npm run dev           # Dev server con Turbopack → http://localhost:3000
npm run build         # Build de producción
npm run test          # Vitest (unitarios)
npm run lint          # ESLint
```

Requiere que el Backend esté corriendo en `http://localhost:8000`.

### Mobile Android
- Abrir `Mobile/PlatziFlixAndroid/` en Android Studio
- La base URL es `10.0.2.2:8000` para emulador / `<IP_LOCAL>:8000` para dispositivo físico
- Requiere configurar `network_security_config.xml` para permitir HTTP en desarrollo

### Mobile iOS
- Abrir `Mobile/PlatziFlixiOS/PlatziFlixiOS.xcodeproj` en Xcode
- La base URL es `http://localhost:8000`

---

## API Contract — Referencia rápida

**Base URL:** `http://localhost:8000`

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| `GET` | `/courses` | Lista todos los cursos activos |
| `GET` | `/courses/{slug}` | Detalle del curso + clases |
| `GET` | `/courses/{slug}/classes/{id}` | Una clase específica (contrato) |
| `GET` | `/health` | Estado de la API y base de datos |

**Campos clave por entidad:**
- **Course:** `id`, `name`, `description`, `thumbnail`, `slug`, `teacher_id[]`, `classes[]`
- **Class/Lesson:** `id`, `course_id`, `name`, `description`, `slug`, `video_url`
- **Teacher:** `id`, `name`, `email`

Todas las entidades tienen `created_at`, `updated_at`, `deleted_at` (soft delete — nunca borrar físicamente).

El contrato completo está en `Backend/specs/00_contracts.md`.

---

## Patrones arquitectónicos por proyecto

### Backend
- **Layered Architecture**: Route → `CourseService` → SQLAlchemy ORM → PostgreSQL
- Usar `joinedload` para relaciones (evitar N+1 queries)
- Soft delete: siempre filtrar `WHERE deleted_at IS NULL`
- Nuevos endpoints siguen el patrón: ruta en `app/main.py`, lógica en `app/services/`
- Nuevas tablas: crear modelo en `app/models/`, luego `make create-migration`

### Frontend
- **Server Components por defecto** — solo agregar `"use client"` cuando sea estrictamente necesario
- Fetch de datos en el componente de página (async function), no en componentes hijos
- Estilos: siempre usar `.module.scss` por componente; las variables globales están en `src/styles/vars.scss` e importadas automáticamente
- Tipos en `src/types/index.ts` — agregar interfaces nuevas ahí
- Cache de fetch: `{ cache: "no-store" }` como patrón estándar del proyecto

### Mobile Android
- **Clean Architecture de 3 capas:** `data/` → `domain/` → `presentation/`
- Nuevas pantallas: agregar `*Screen.kt` en `presentation/`, `*ViewModel.kt` con `*UiState` y `*UiEvent`
- Nuevas llamadas a API: agregar método en `ApiService.kt`, implementar en `RemoteCourseRepository.kt`, mapear con `*Mapper.kt`
- Estado via `StateFlow<UiState>` en el ViewModel

### Mobile iOS
- **Clean Architecture de 3 capas:** `Data/` → `Domain/` → `Presentation/`
- Nuevas vistas: `*View.swift` en `Presentation/Views/`, `*ViewModel.swift` con `@Published` properties
- Nuevos endpoints: agregar case en `CourseAPIEndpoints.swift`, implementar en `RemoteCourseRepository.swift`
- Usar `@MainActor` en todos los ViewModels

---

## Restricciones y convenciones importantes

- **No eliminar registros físicamente** — el sistema usa soft delete (`deleted_at`). Siempre setear `deleted_at` en lugar de `DELETE`.
- **No agregar state management global al Frontend** — la arquitectura usa SSR; Redux/Zustand no están en el stack.
- **No cambiar el contrato de la API** sin actualizar los cuatro proyectos que lo consumen.
- **Las migraciones son unidireccionales** — no editar archivos en `Backend/app/alembic/versions/` ya creados; crear una nueva migración.
- **Backend/specs/00_contracts.md** es la fuente de verdad del contrato. Cualquier nuevo endpoint debe documentarse ahí.

---

## Datos de seed (desarrollo)

El backend incluye datos de prueba accesibles con `make seed`:

- **3 profesores:** Juan Pérez, María García, Carlos Rodríguez
- **3 cursos:** `curso-de-react`, `curso-de-python`, `curso-de-javascript`
- **6 lecciones** distribuidas entre los cursos

---

## Features preparadas pero no implementadas

El Frontend tiene tipos TypeScript definidos para estas features (en `src/types/index.ts`) pero sin UI ni endpoints:

- `Progress` — tracking de progreso por usuario
- `Quiz` — preguntas por clase
- `FavoriteToggle` — marcar cursos favoritos

Al implementarlas, seguir el mismo patrón de contrato en `Backend/specs/00_contracts.md` y agregar los endpoints correspondientes al Backend primero.

---

## Archivos clave por proyecto

| Archivo | Propósito |
|---------|-----------|
| `Backend/app/main.py` | Endpoints FastAPI |
| `Backend/app/services/course_service.py` | Lógica de negocio |
| `Backend/app/models/` | Modelos SQLAlchemy |
| `Backend/app/core/config.py` | Configuración centralizada |
| `Backend/docker-compose.yml` | Infraestructura local |
| `Frontend/src/app/page.tsx` | Página de inicio |
| `Frontend/src/app/course/[slug]/page.tsx` | Detalle del curso |
| `Frontend/src/types/index.ts` | Interfaces TypeScript |
| `Frontend/src/styles/vars.scss` | Tokens de color globales |
| `Mobile/PlatziFlixAndroid/.../CourseListViewModel.kt` | Estado Android |
| `Mobile/PlatziFlixiOS/.../CourseListViewModel.swift` | Estado iOS |
| `Backend/specs/00_contracts.md` | Contrato oficial de la API |
| `ARCHITECTURE.md` | Diagramas completos del sistema |
