# Mapeo SDD — De `spec-python/` a Laravel + React en Arquitectura Onion

> **Desarrollo Guiado por Especificación (Spec-Driven Development)**
> **Prueba técnica · Ficha ADSO 3413974**
>
> Este documento demuestra la **trazabilidad completa y rigurosa** entre los documentos base de `spec-python/` y la solución implementada en los 6 repositorios bajo **Arquitectura Onion**.

---

## 1. Trazabilidad Documento por Documento

### 1.1 `constitution.md` $\rightarrow$ Reglas de Arquitectura y Verificación
| Artículo / Principio en `constitution.md` | Implementación en Laravel (Onion) | Archivo de Código / Evidencia |
|---|---|---|
| **Art. I: Dirección de Dependencias** | 4 anillos concéntricos + Bootstrap. Las dependencias van estrictamente hacia el centro. | `app/Domain/`, `app/Application/`, `app/Infrastructure/`, `app/Presentation/` |
| **Art. II: Puertos Outbound** | 10 puertos salientes definidos en Application. | `app/Application/Ports/Outbound/` (10 interfaces) |
| **Art. III: Composition Root** | Único Service Provider que amarra interfaces con implementaciones concretas. | `app/Bootstrap/PortBindingsServiceProvider.php` |
| **Art. IV: Pureza de Dominio (DIP)** | Dominio no importa nada del framework ni de terceros. Cero `Illuminate`. | `tests/Architecture/ArchitectureTest.php::test_domain_does_not_depend_on_illuminate` |
| **Art. V: Propiedad del Esquema** | La API es la única dueña de las migraciones. `infra` solo levanta MySQL vacío. | `test-simple-stock-flow-api/database/migrations/` |
| **Art. VI: Invariantes en el Agregado** | Reglas de negocio forzadas dentro de las entidades, no en servicios anémicos. | `app/Domain/Model/Product.php`, `Sale.php` |
| **Art. VII: Datos Derivados** | `total` y `subtotal` nunca se almacenan en tablas; se calculan dinámicamente. | `Sale::getTotal()`, `SaleItem::getSubtotal()` |
| **Art. VIII: Pruebas Automáticas** | Tests unitarios y de arquitectura ejecutables sin depender de servicios externos. | `tests/Unit/DomainRulesTest.php`, `tests/Architecture/ArchitectureTest.php` |
| **Art. IX: Cero Secretos en Git** | Variables de entorno configuradas vía `.env.example` sin valores sensibles. | `.env.example` en todos los repositorios |
| **Art. XI: Idioma Canónico** | Código, métodos y variables en **inglés**. Documentación y mensajes de error en **español**. | Todo el codebase |

---

### 1.2 `spec.md` $\rightarrow$ Historias de Usuario y Reglas de Negocio
| Regla / Elemento en `spec.md` | Implementación en Dominio / Aplicación | Verificación / Test |
|---|---|---|
| **RN-01: Stock nunca negativo** | `Product::withdraw()` valida stock disponible y lanza `InsufficientStockException`. | `DomainRulesTest::test_product_withdraw_exceeding_stock_fails` |
| **RN-02: Precio > 0** | Value Object `Money` valida que el importe sea estrictamente positivo. | `DomainRulesTest::test_negative_money_throws_exception` |
| **RN-03: Cantidad > 0** | Value Object `Quantity` valida enteros estrictamente mayores que cero. | `DomainRulesTest::test_quantity_must_be_positive` |
| **RN-04: Venta con al menos 1 línea** | `Sale` valida que el array de ítems no esté vacío (`EmptySaleException`). | `DomainRulesTest::test_sale_rejects_empty_items` |
| **RN-05: Producto no repetido** | `Sale` rechaza ítems con el mismo `productId` (`RepeatedProductException`). | `DomainRulesTest::test_sale_rejects_repeated_products` |
| **RN-06: Congelación al vender** | `SaleItem` congela nombre, precio y categoría en el momento de la venta. | `PlaceSaleService::execute()`, tabla `sale_item` |
| **RN-07: Venta inmutable** | No existen métodos de edición ni borrado en el agregado `Sale` ni rutas HTTP PUT/DELETE. | Invariante estructural |
| **RN-08: Baja lógica** | `Product::softDelete()` establece `deleted_at`. Un producto vendido no se borra. | `ProductCatalogService::deleteProduct()` |
| **RN-09: Monomoneda (`COP`)** | `Money` maneja divisa fija `'COP'` y precisión decimal con `BigDecimal`. | `Money::of()`, `Money::zero()` |
| **RN-10: Username normalizado** | `Username` normaliza automáticamente a minúsculas y recorta espacios. | `DomainRulesTest::test_username_is_normalized_to_lowercase` |
| **RN-11: Roles admin y seller** | Value Object `Role` restringe al conjunto cerrado `{'admin', 'seller'}`. | `DomainRulesTest::test_role_validations` |
| **RN-12: Total derivado** | La suma de subtotales genera el total dinámicamente. | `DomainRulesTest::test_sale_total_is_derived_dynamically` |
| **DP-01: Producto renombrado en reporte** | Reporte toma el nombre congelado más reciente dentro del rango. | `EloquentSalesReportQuery::getReport()` |
| **DP-04: Admin no crea admins** | El caso de uso `registerSeller` solo puede crear vendedores. | `AuthenticationService::registerSeller()` |

---

### 1.3 `data-model.md` $\rightarrow$ Esquema Relacional y Migraciones
| Definición en `data-model.md` | Implementación en Laravel | Detalle Técnico |
|---|---|---|
| **Tabla `category`** | `2026_10_02_000001_create_simple_stock_flow_schema.php` | `id` CHAR(36), `name` VARCHAR(120) UNIQUE |
| **CHECK `ck_category_name_not_blank`** | `DB::statement(...)` | `CHAR_LENGTH(TRIM(name)) > 0` |
| **Tabla `product`** | Migración inicial | Con `category_id`, `deleted_at`, `version` y llaves foráneas |
| **3 CHECKs en `product`** | `ck_product_name_not_blank`, `ck_product_price_positive`, `ck_product_stock_non_negative` | Integridad a nivel de motor MySQL |
| **Tabla `user`** | Migración inicial | `username`, `password_hash`, `role` |
| **3 CHECKs en `user`** | `ck_user_username_normalized`, `ck_user_role_allowed`, `ck_user_password_hash_not_blank` | Restricción estricta de roles y formatos |
| **Tabla `sale` y `sale_item`** | Migración inicial | Con llaves foráneas e índices `idx_sale_sold_at` |
| **2 CHECKs en `sale_item`** | `ck_sale_item_quantity_positive`, `ck_sale_item_unit_price_positive` | Cero cantidades o precios negativos |
| **Semilla de Categorías (§9.1)** | `2026_10_02_000002_seed_categories.php` | 5 categorías con los UUIDs oficiales (General, Herramientas, Electricidad, Fontanería, Pinturas) |
| **Admin Inicial (§9.2)** | `database/seeders/DatabaseSeeder.php` | Lee credenciales desde `.env` y siembra con hash nativo |

---

### 1.4 `api-contract.md` $\rightarrow$ Rutas y Respuestas HTTP
| Endpoint | Método | Controlador | Autorización | Formato de Respuesta |
|---|---|---|---|---|
| `/api/auth/login` | POST | `AuthController@login` | Anónimo | 200 `{ accessToken, expiresAt, username, role }` |
| `/api/auth/register` | POST | `AuthController@register` | Admin | 201 Vacío (sin `Location`) |
| `/api/products` | GET | `ProductController@index` | Autenticado | 200 `{ items, page, size, total, totalPages }` |
| `/api/products/{id}` | GET | `ProductController@show` | Autenticado | 200 `ProductView` (incluye dados de baja) |
| `/api/products` | POST | `ProductController@store` | Admin | 201 `ProductView` |
| `/api/products/{id}` | PUT | `ProductController@update` | Admin | 200 `ProductView` |
| `/api/products/{id}` | DELETE | `ProductController@destroy` | Admin | 204 Vacío (Baja lógica) |
| `/api/products/{id}/image` | POST | `ProductController@uploadImage` | Admin | 200 `ProductView` con `imageUrl` |
| `/api/categories` | GET | `CategoryController@index` | Autenticado | 200 `[{ id, name }, ...]` |
| `/api/sales` | POST | `SaleController@store` | Autenticado | 201 `SaleView` |
| `/api/sales` | GET | `SaleController@index` | Autenticado | 200 `{ items, page, size, total, totalPages }` |
| `/api/sales/{id}` | GET | `SaleController@show` | Autenticado | 200 `SaleView` con líneas congeladas |
| `/api/reports/sales` | GET | `ReportController@sales` | Autenticado | 200 `SalesReport` con `salesCount`, `grandTotal`, `currency`, `rows` |
| `/health` | GET | `HealthController@check` | Anónimo | 200 `{ status: "ok" }` |
| `/media/{key}` | GET | `MediaController@show` | Anónimo | 200 con binario y `Content-Type` de imagen |

**Formatos de Error cumplidos:**
- **422 / 409 / 500:** `application/problem+json` con `title`, `status`, `detail` en español.
- **400:** Formato de error de validación sintáctica `{ title, status, detail, errors: { ... } }`.
- **401 / 403 / 404 / 405:** **Cuerpo vacío (`Content-Length: 0`)** y cabeceras `WWW-Authenticate` y `Allow`.

---

### 1.5 `tasks.md` $\rightarrow$ Mapeo 1:1 de Tareas
- **T-01:** Esquema inicial con los 9 CHECK constraints manuales $\rightarrow$ `database/migrations/2026_10_02_000001_create_simple_stock_flow_schema.php`.
- **T-02:** Semilla de categorías con 5 UUIDs fijos $\rightarrow$ `database/migrations/2026_10_02_000002_seed_categories.php`.
- **T-04:** Servicio de catálogo $\rightarrow$ `app/Application/UseCase/ProductCatalogService.php`.
- **T-05:** Manejo de moneda $\rightarrow$ `app/Domain/ValueObject/Money.php` con `Brick\Math\BigDecimal`.
- **T-06:** Servicio de autenticación $\rightarrow$ `app/Application/UseCase/AuthenticationService.php`.
- **T-07:** Consulta de ventas $\rightarrow$ `app/Application/UseCase/GetSalesService.php`.
- **T-08:** Reporte de ventas $\rightarrow$ `app/Application/UseCase/SalesReportService.php` + `EloquentSalesReportQuery.php`.
- **T-10:** Registro de venta transaccional $\rightarrow$ `app/Application/UseCase/PlaceSaleService.php` + `LaravelUnitOfWork.php`.
- **T-20:** Barreras del motor (9 CHECK constraints) $\rightarrow$ Verificables en base de datos.
- **T-21:** Invariantes del modelo $\rightarrow$ Verificables con `DomainRulesTest.php`.
- **T-22:** Venta inmutable $\rightarrow$ Inexistencia de métodos de edición en agregados ni routers.
