# Manager Library — API (PHP + Laravel)

Backend de Manager Library: catálogo de libros, socios, préstamos con devolución por línea, libros vencidos y reporte de más prestados.

Implementa en Laravel el contrato `api-contract.md` y el modelo de datos del spec; el negocio no cambia, solo el stack.

## Stack
- PHP 8.3
- Laravel 11
- PostgreSQL 16 (esquema `library`)
- PHPUnit 12
- `tymon/jwt-auth` (T-14)

---

## Arranque local

```bash
cp .env.example .env && composer install && php artisan key:generate && php artisan migrate --force && php artisan serve
```

### Con Docker:
```bash
docker build -t library-api . && docker run -p 8000:8000 --env-file .env library-api
```
*La base de datos debe existir y estar vacía; todo el DDL vive en las migraciones de este repo (ADR-001).*

---

## Variables de entorno

| Variable | Obligatoria | Nota |
| :--- | :---: | :--- |
| `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` | Sí | Conexión a PostgreSQL |
| `DB_SCHEMA` | No (`library`) | Esquema de la aplicación |
| `DB_TEST_DATABASE` | Pruebas | Base que `SchemaTest` borra y recrea |
| `JWT_SECRET` | Sí | Clave del token (HS256, 8 h) |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | Sí | Admin inicial; solo en `.env`, nunca en el repo |
| `COVERS_DIR` | No | Carpeta de portadas |
| `CORS_ORIGIN` | Sí | Origen del frontend |

---

## Pruebas

```bash
php artisan test --filter=HealthTest
php artisan test --filter=SchemaTest
php artisan test tests/Unit/Domain
php artisan test tests/Unit/Application
php artisan test tests/Feature
php artisan test
```

---

## Estado de las tareas
El avance por tarea, con su evidencia, está en `docs/tasks.md`.

| Tarea | Estado |
| :--- | :--- |
| T-01 Esqueleto y `GET /health` | hecha |
| T-02 Migración inicial (seis tablas, índices, semilla) | hecha |
| T-03 Dominio: Isbn, Quantity, DateRange | hecha |
| T-04 Dominio: Book | hecha |
| T-05 Dominio: Loan | hecha |
| T-06 Dominio: Member y User | hecha |
| T-07 Casos de uso de autenticación | hecha |
| T-08 Casos de uso de libros | hecha |
| T-09 Caso de uso prestar | hecha |
| T-10 Caso de uso devolver y vencidos | hecha |
| T-11 Caso de uso reporte más prestados | hecha |
| T-12 Repositorios de libro, socio, usuario | hecha |
| T-13 Repositorio de préstamos + concurrencia | hecha |
| T-14 Adaptador de seguridad (bcrypt + JWT) | hecha |
| T-15 Adaptador de portadas | hecha |
| T-16 Adaptador HTTP (rutas, error único, roles) | hecha |
| T-17 Composición y arranque | hecha |
| T-18 Invariantes en base | hecha |
| T-19 Pruebas de arquitectura | hecha |
| T-20 a T-23 | Ver repositorios hermanos |

**Evidencia:** `php artisan migrate --force` aplica `2026_10_22_000001_create_library_schema` (DONE) y `php artisan test` termina con `Tests: 115 passed (360 assertions)`: T-01 a T-19 completadas. Detalle en `docs/tasks.md`.

---

## Documentación (SDD)
- `docs/plan.md`: decisiones técnicas traducidas de .NET a Laravel y la tabla de desviaciones respecto al spec.
- `docs/tasks.md`: las 23 tareas del spec con estado y evidencia.
- El spec completo (historias, contrato, modelo de datos, constitución, ADR) vive en el repositorio docs.

---

## Arquitectura
Hexagonal sobre Laravel. Ver `docs/plan.md` §2.

```text
app/
├── Domain/                           # PHP puro, sin dependencias de Laravel
│   ├── Entities/                     # Book, Loan, LoanItem, Member, User
│   └── ValueObjects/                 # Isbn, Quantity, DateRange, DocumentNumber
├── Application/
│   ├── Ports/Outbound/               # Interfaces: BookRepository, LoanRepository, ...
│   └── Services/                     # Casos de uso: LoginService, LendBooksService, ...
├── Adapters/
│   ├── Inbound/Http/                 # Controllers, FormRequests, Middleware
│   └── Outbound/                     # Persistence/, Storage/, Security/
└── Bootstrap/Composition/
    └── PortBindings.php              # Único punto de composición
```

---

## Decisiones técnicas clave

| # | Decisión | Spec (.NET) | Laravel |
| :-: | :--- | :--- | :--- |
| **D-01** | Framework | ASP.NET Core Minimal API | Rutas en `routes/api.php`, prefijo `api/v1` en `bootstrap/app.php` |
| **D-03** | Persistencia | EF Core | Query Builder + SQL revisable en `database/sql/` |
| **D-04** | Concurrencia | Testigo de versión | Compare-and-swap sobre `available_copies` |
| **D-07** | Objetos de valor | `record` | Clases `final readonly` |
| **D-11** | Autenticación | JWT HS256, 8 h | `tymon/jwt-auth` (ver T-14) |
| **D-12** | Pruebas | xUnit | PHPUnit |

*Ver todas las decisiones y desviaciones en `docs/plan.md`.*

---

## Repositorios hermanos

| Tarea | Descripción | Repositorio |
| :---: | :--- | :--- |
| **T-20** | infra: docker compose up levanta motor vacío, API y frontend | `test-manager-library-3239137-infra` |
| **T-21** | tool: sembrador de demostración por la API | `test-manager-library-3239137-tool` |
| **T-22** | app: login, catálogo, socios, prestar, devolver, vencidos, reporte | `test-manager-library-3239137-app` |
| **T-23** | page: sitio público estático de presentación | `test-manager-library-3239137-page` |
