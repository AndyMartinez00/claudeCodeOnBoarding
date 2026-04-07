# 00 — Análisis de Impacto: Sistema de Ratings

**Feature:** Rating de cursos (1 a 5 estrellas)
**Fecha:** 2026-04-07
**Proyectos afectados:** Backend, Frontend
**Proyectos excluidos:** Mobile (Android, iOS)

---

## Resumen ejecutivo

| Proyecto | Archivos nuevos | Archivos modificados | Nivel de cambio |
|----------|----------------|---------------------|-----------------|
| **Backend** | 2 | 4 | Medio |
| **Frontend** | 2 | 4 | Medio |

---

## Backend

### 1. Base de datos — nueva tabla `ratings`

El sistema no tiene usuarios, por lo que los ratings serán **anónimos** en esta primera versión. Se necesita una nueva tabla:

```
ratings
├── id           Integer PK
├── course_id    Integer FK → courses.id
├── rating       Integer CHECK (1-5)
├── created_at   DateTime
├── updated_at   DateTime
└── deleted_at   DateTime  (soft delete, patrón del proyecto)
```

**Archivo nuevo:** `app/models/rating.py`

---

### 2. Modelo `Course` — agregar relación

`app/models/course.py` — agregar `relationship` con la nueva tabla:

```python
ratings = relationship("Rating", back_populates="course", cascade="all, delete-orphan")
```

---

### 3. `CourseService` — 3 cambios

**a) `get_all_courses()`** — agregar `rating_average` y `rating_count` en el dict retornado. Requiere `AVG` + `COUNT` sobre `ratings` por `course_id` (subquery o carga de relación).

**b) `get_course_by_slug()`** — mismo agregado; puede incluir distribución por estrella si se requiere en el futuro.

**c) Método nuevo: `rate_course(slug, rating)`** — valida que el curso exista, que `1 ≤ rating ≤ 5`, inserta en `ratings`, retorna el nuevo promedio y conteo.

---

### 4. `main.py` — endpoint nuevo

```
POST /courses/{slug}/rating
Body:    { "rating": 3 }
Response: { "rating_average": 4.2, "rating_count": 15 }
```

Los endpoints `GET /courses` y `GET /courses/{slug}` no cambian su firma, solo su respuesta (se suman los campos `rating_average` y `rating_count`).

---

### 5. Migración Alembic — archivo nuevo

`app/alembic/versions/[hash]_add_ratings_table.py`

No se edita la migración existente; se crea una nueva (regla del proyecto: las migraciones son unidireccionales).

---

### 6. Contrato — `Backend/specs/00_contracts.md`

- Documentar el nuevo endpoint `POST /courses/{slug}/rating`.
- Actualizar el contrato de `Course` con los campos `rating_average` y `rating_count`.

---

### Resumen de archivos Backend

| Acción | Archivo |
|--------|---------|
| **CREAR** | `Backend/app/models/rating.py` |
| **CREAR** | `Backend/app/alembic/versions/[hash]_add_ratings_table.py` |
| **MODIFICAR** | `Backend/app/models/course.py` — agregar relationship |
| **MODIFICAR** | `Backend/app/services/course_service.py` — método nuevo + actualizar existentes |
| **MODIFICAR** | `Backend/app/main.py` — nuevo endpoint POST |
| **MODIFICAR** | `Backend/specs/00_contracts.md` — actualizar contrato |

---

## Frontend

### 1. Tipos — `src/types/index.ts`

Agregar campos de rating a `Course` y un tipo para el body del POST:

```typescript
export interface Course {
  // ...campos actuales...
  rating_average?: number; // null si no tiene ratings aún
  rating_count?: number;
}

export interface CourseRating {
  rating: number; // 1–5
}
```

---

### 2. Componente nuevo — `StarRating`

`src/components/StarRating/StarRating.tsx` + `StarRating.module.scss`

Dos modos de uso:

- **Display** (read-only): muestra promedio + cantidad. Server Component — sin `"use client"`.
- **Interactive**: botones de 1-5 estrellas que hacen `POST`. Requiere `"use client"` por los `onClick`.

La separación en un solo componente con prop `interactive` permite que las páginas de lista usen el modo display (sin JS en cliente) y el detalle use el modo interactivo.

---

### 3. `Course.tsx` — mostrar rating en tarjeta

Agregar `rating_average` y `rating_count` a las props y renderizar `<StarRating>` en modo display debajo de la duración. Campos opcionales con fallback a `null` — no rompe la tarjeta cuando el curso no tiene ratings.

---

### 4. `CourseDetail.tsx` — rating display + widget interactivo

En el bloque `.stats` actual, agregar:

- `<StarRating>` en modo display (promedio actual + conteo).
- `<StarRatingInteractive>` con el `POST` al API.

El widget interactivo debe extraerse como un Client Component separado para no contaminar toda la página con `"use client"` (el resto de la página sigue siendo Server Component).

---

### 5. `page.tsx` — pasar campos de rating al componente `Course`

Agregar `rating_average` y `rating_count` al spread de props que se le pasan a `<CourseComponent>`.

> **Bug pre-existente detectado:** `page.tsx` hace `return data.data` pero la API retorna el array directamente sin envoltura `{ data: [] }`. Corregir a `return data` en la misma pasada.

---

### 6. Llamada al API — función `rateCourse`

```typescript
async function rateCourse(slug: string, rating: number) {
  await fetch(`http://localhost:8000/courses/${slug}/rating`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ rating }),
  });
}
```

---

### Resumen de archivos Frontend

| Acción | Archivo |
|--------|---------|
| **CREAR** | `Frontend/src/components/StarRating/StarRating.tsx` |
| **CREAR** | `Frontend/src/components/StarRating/StarRating.module.scss` |
| **MODIFICAR** | `Frontend/src/types/index.ts` — agregar campos de rating |
| **MODIFICAR** | `Frontend/src/components/Course/Course.tsx` — mostrar rating en tarjeta |
| **MODIFICAR** | `Frontend/src/components/CourseDetail/CourseDetail.tsx` — rating display + widget |
| **MODIFICAR** | `Frontend/src/app/page.tsx` — pasar rating props + fix bug `data.data` |

---

## Flujo completo del feature

```
Usuario ve tarjeta de curso
  └─ GET /courses → rating_average + rating_count en cada curso
     └─ Course.tsx muestra ★★★★☆ (4.0) · 12 ratings

Usuario entra al detalle
  └─ GET /courses/{slug} → rating_average + rating_count
     └─ CourseDetail muestra rating actual
     └─ StarRatingInteractive permite hacer click en 1-5 estrellas

Usuario hace click en estrella (ej: 4 estrellas)
  └─ POST /courses/{slug}/rating  { "rating": 4 }
     └─ Backend valida 1 ≤ rating ≤ 5
     └─ Inserta en tabla ratings
     └─ Retorna nuevo promedio y conteo
        └─ Frontend actualiza display
```

---

## Decisiones de diseño pendientes

Antes de implementar, hay dos puntos que requieren definición:

1. **Ratings duplicados** — sin sistema de usuarios, el mismo visitante puede votar N veces. ¿Se acepta en v1, o se limita por sesión/cookie?

2. **Ubicación del widget interactivo** — ¿solo en la página de detalle (`/course/[slug]`) o también en la tarjeta de la lista principal? Impacta si `Course.tsx` necesita convertirse en Client Component.
