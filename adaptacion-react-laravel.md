# Adaptación del spec a React + Laravel

El spec (`spec-python/`) está escrito para Python; este proyecto lo lleva a **PHP 8.3 con Laravel** en el
backend y **React 18 con Vite** en el frontend. Lo que el spec dice sobre el negocio no cambia: HU-01 a HU-10,
las reglas RN-1 a RN-6, el contrato de la API y el modelo de datos. Lo que dice sobre la tecnología se traduce.

## 1. Traducción de tecnología

| Spec (Python) | Aquí (Laravel / React) | Referencia |
|---|---|---|
| Capas hexagonales (artículos I–III) | `app/Domain`, `app/Application` (con `Ports`), `app/Adapters`, `app/Http` | architecture.md §3 |
| Puertos como `Protocol` | Interfaces PHP en `app/Application/Ports` | T-07, T-08 |
| Objetos de valor `Isbn`, `Quantity`, `DateRange` | Clases `final` con constructor privado y `of()` | T-03 |
| Composición en un único punto | `AppServiceProvider` enlaza puertos con adaptadores; Laravel es solo el contenedor | T-17, artículo III |
| ORM y migraciones | SQL explícito en migraciones de Laravel; el motor arranca vacío | ADR-001, T-02, T-18 |
| `import-linter` | `tests/Architecture/ArchitectureTest.php` con un detector de importaciones | T-19 |
| JWT con biblioteca | `JwtTokenIssuer` (HS256, 8 h, `sub` y `role`) y middleware `AuthenticateWithJwt` | CA-01.1, CA-01.3 |
| Hash de claves | `PasswordHasherAdapter` con bcrypt (`password_hash` de PHP) | RNF-3, T-14 |
| Cuerpo de error único | Renderers en `bootstrap/app.php`: `DomainError`, validación, 404, `22P02` y cualquier otro fallo | RNF-4, T-16 |
| Frontend | React + Vite, `Authorization: Bearer`, muestra `message` y `details` del error | T-22 |

## 2. Decisiones propias de la adaptación

| # | Decisión | Por qué |
|---|---|---|
| D-A1 | Sin tablas del framework: `SESSION_DRIVER=array`, `CACHE_STORE=array`, `QUEUE_CONNECTION=sync` | data-model §2: son seis tablas y nada más |
| D-A2 | Sin rutas fuera del contrato: se quitaron `/up` y `/` | api-contract §3 |
| D-A3 | `JWT_SECRET` obligatorio, sin valor de respaldo | api-contract §4; un secreto en el código anula la firma |
| D-A4 | `X-Simulated-Time` solo se acepta con `ALLOW_SIMULATED_TIME=true` | T-21 exige 3 vencidos **por la API**; si cualquier cliente fijara la hora, eludiría RN-2 y RN-3 |
| D-A5 | CORS con un solo origen (`CORS_ORIGIN`) y sin credenciales | api-contract §4 |
| D-A6 | Identificador que no es UUID: 404 en la ruta, 422 en el cuerpo | RNF-4: el motor no debe filtrar un error 500 |
| D-A7 | Sin `created_at` / `updated_at` | data-model §8 |

## 3. Trazabilidad: historia → código → evidencia

| Historia / criterio | Dónde vive | Prueba |
|---|---|---|
| HU-01 · CA-01.1 a CA-01.3 | `LoginService`, `JwtTokenIssuer`, `AuthenticateWithJwt` | `tests/Contract/RoutesTest` (401 y 403) |
| HU-02 · CA-02.1 a CA-02.3 | `CreateLibrarianService`, `User::createLibrarian` | `UserAndMemberTest` |
| HU-03 · CA-03.1 a CA-03.5 | `BookService`, `Book`, `DiskCoverStorage` | `BookTest`, `ValueObjectsTest` |
| HU-04 · CA-04.1 a CA-04.3 | `BookService::search`, `PostgresBookRepository` | integración (T-12) |
| HU-05 · CA-05.1 a CA-05.3 | `MemberService`, `Member` | `UserAndMemberTest` |
| HU-06 · CA-06.1 a CA-06.7 | `LoanService::createLoan`, `Loan::addItem` | `LoanServiceTest`, `LoanTest` |
| HU-07 · CA-07.1 a CA-07.4 | `LoanService::returnItem`, `Loan::returnItem` | `LoanServiceTest`, `LoanTest` |
| HU-08 · CA-08.1 a CA-08.3 | `Loan::status` (calculado, no almacenado) | `LoanTest::test_status_is_calculated_open_overdue_closed` |
| HU-09 · CA-09.1, CA-09.2 | `LoanReader::overdue`, `PostgresLoanReader` | integración (T-12) |
| HU-10 · CA-10.1 a CA-10.5 | `ReportService`, `PostgresReportReader`, ADR-004 | `ValueObjectsTest` (rango), integración |
| RN-1 | `Book::withdraw/restock` y `ck_book_available_range` | `BookTest` |
| RN-2, RN-3 | `Loan::LOAN_DAYS`, `Loan::MAX_OPEN_PER_MEMBER`, `LoanService` | `LoanServiceTest` |
| RN-4 | Línea congelada en `LoanItem`, ADR-004 | `LoanTest::test_line_freezes_title_isbn_and_category` |
| RN-5 | Sin operaciones de borrado de préstamos; baja lógica (ADR-003) | contrato de rutas |
| RN-6 | Fechas UTC; mensajes en español; código y columnas en inglés | revisión |
| RNF-2 | `ck_book_available_range` + `getForUpdate` del socio, ADR-002 | prueba concurrente (pendiente de ejecutar) |
| RNF-4 | Renderers de error en `bootstrap/app.php` | `RoutesTest` |

## 4. Estado de las tareas

Las pruebas están escritas pero **no se han ejecutado en este entorno** (sin PHP ni Docker). La única
verificación realizada fue la compilación del frontend con `vite build`.

| Tarea | Estado | Evidencia |
|---|---|---|
| T-01, T-02 | Implementada | Migración `…000001`; `/health` |
| T-03 a T-06 | Implementada, pruebas escritas | `tests/Unit` |
| T-07 a T-11 | Implementada, pruebas con fakes para préstamo y devolución | `LoanServiceTest` |
| T-12 a T-15 | Implementada | Requiere PostgreSQL para la prueba de integración |
| T-16 | Implementada | `tests/Contract/RoutesTest` |
| T-17 | Implementada | `library:bootstrap-admin` en el arranque del contenedor |
| T-18 | Implementada | Migración `…000002`; `data-model.md` §4 actualizado |
| T-19 | Implementada | `tests/Architecture/ArchitectureTest` |
| T-20 | Implementada | `manager-library-infra` |
| T-21 | Implementada | `manager-library-tool/seed.php` verifica 3 vencidos y reporte con filas |
| T-22 | Implementada | Siete pantallas más alta de bibliotecarios |
| T-23 | Implementada | `manager-library-page/index.html` |

## 5. Fuera de alcance (spec §2)

Multas y pagos, reservas, renovaciones, correo o SMS, importación masiva, préstamos entre sedes y
atributos de libro adicionales (editorial, año, idioma). No se implementan ni se mencionan en el producto.
