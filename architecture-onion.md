# Simple Stock Flow — Arquitectura Onion · Traducción Oficial para Laravel

> **Documento de Arquitectura y Especificación de Capas**
> **Prueba técnica · Ficha ADSO 3413974**
> Rige bajo los principios de `constitution.md`. Traduce la especificación base (Hexagonal) a **Arquitectura Onion (Cebolla)** en **PHP / Laravel**.

---

## 1. El Propósito y las 3 Afirmaciones

Simple Stock Flow es el sistema de control de stock y ventas para un almacén pequeño.
Todo el diseño se apoya en 3 afirmaciones innegociables:
1. **El stock que muestra el catálogo es el stock que hay.**
2. **Una venta registrada no se puede alterar después.**
3. **El reporte de un período cerrado dice hoy lo mismo que dirá en un año.**

**Actores:**
- `seller`: Vende y consulta ventas/catálogo/reportes.
- `admin`: Todo lo anterior + mantiene catálogo, gestiona imágenes y crea usuarios vendedores.
- *Anónimo*: Solo login (`/api/auth/login`) y health check (`/health`).

---

## 2. Las 12 Reglas de Negocio

| # | Regla | Responsabilidad |
|---|---|---|
| RN-01 | El stock nunca es negativo | Agregado `Product` (Domain) |
| RN-02 | El precio es mayor que cero | Value Object `Money` (Domain) |
| RN-03 | La cantidad es mayor que cero | Value Object `Quantity` (Domain) |
| RN-04 | Una venta tiene al menos una línea | Agregado `Sale` (Domain) |
| RN-05 | Un producto no se repite en la misma venta | Agregado `Sale` (Domain) |
| RN-06 | El precio y el nombre se congelan al vender | Entidad interna `SaleItem` (Domain) |
| RN-07 | Una venta registrada no se modifica ni se anula | Invariante estructural: sin métodos de edición ni endpoints |
| RN-08 | Un producto vendido no se borra: se da de baja | Agregado `Product` (baja lógica vía `deleted_at`) |
| RN-09 | Todos los importes en la misma moneda (`COP`) | Value Object `Money` |
| RN-10 | `username` único, normalizado a minúsculas | Value Object `Username` + Agregado `User` |
| RN-11 | El rol está en el conjunto cerrado `{admin, seller}` | Value Object `Role` |
| RN-12 | El total siempre es la suma de sus líneas | Derivado dinámico en `Sale` (nunca se almacena) |

---

## 3. Arquitectura Onion — Las Capas Concéntricas

La arquitectura se organiza en 4 anillos concéntricos y 1 punto de ensamblaje (Bootstrap):

```
        ┌──────────────────────────────────────┐
        │  Bootstrap                           │  ← Conecta interfaces con implementaciones
        │  app/Bootstrap/                      │    NO ES UN ANILLO. Importa todo.
        │  PortBindingsServiceProvider.php     │    Nadie lo importa a él.
        └───────────────┬──────────────────────┘
     ┌──────────────────┴──────────────────┐
     ▼                                     ▼
┌─────────────────┐            ┌──────────────────────┐
│ Presentation    │            │ Infrastructure       │
│ Anillo 4        │            │ Anillo 3             │
│ Controllers     │            │ Eloquent Models      │
│ Requests        │            │ Mappers              │
│ Resources       │            │ Repositorios         │
│ Renderers       │            │ JWT / Hash / Storage │
│ PROHIBIDO:      │            │ Migraciones          │
│ importar Infra  │            └──────────┬───────────┘
└────────┬────────┘                       │
         │        imports hacia adentro   │
         ▼                                ▼
       ┌───────────────────────────────────────┐
       │ Application   Anillo 2                │
       │ Casos de Uso · 5 Puertos Inbound      │
       │ 10 Puertos Outbound                   │
       │ Solo importa Domain. CERO Laravel.    │
       └──────────────────┬────────────────────┘
                          │ solo importa Domain
                          ▼
       ┌───────────────────────────────────────┐
       │ Domain   Anillo 1 (NÚCLEO)            │
       │ Entidades · Value Objects · Reglas    │
       │ PHP PURO. Cero dependencias externas. │
       └───────────────────────────────────────┘
```

### Reglas de Dependencia Inviolables
1. **`Domain` (Anillo 1):** PHP puro. `grep -R "Illuminate\\\\" app/Domain` DEBE arrojar exactamente 0.
2. **`Application` (Anillo 2):** Solo importa `Domain`. Nunca importa `Infrastructure` ni `Illuminate`.
3. **`Infrastructure` (Anillo 3):** Implementa los puertos salientes (`Outbound Ports`) definidos por `Application`. Conoce `Application` y `Domain`.
4. **`Presentation` (Anillo 4):** Solo consume puertos de entrada (`Inbound Ports`) de `Application`. **Prohibido terminantemente importar `Infrastructure`**.
5. **`Bootstrap`:** Ensambla la inyección de dependencias (Service Provider) vinculando interfaces de puertos con sus implementaciones concretas.

---

## 4. Distribución del Código en Laravel (`test-simple-stock-flow-api`)

```
app/
├── Domain/                                          # ANILLO 1: PHP puro
│   ├── Model/
│   │   ├── Product.php                              # Agregado raíz (sin version)
│   │   ├── Sale.php                                 # Agregado raíz (sin total almacenado)
│   │   ├── SaleItem.php                             # Entidad interna (sin subtotal almacenado)
│   │   ├── Category.php                             # Entidad de referencia
│   │   └── User.php                                 # Agregado raíz
│   ├── ValueObject/
│   │   ├── Money.php                                # Brick\Math\BigDecimal, escala 2, HALF_UP
│   │   ├── Quantity.php                             # Entero > 0
│   │   ├── ProductId.php, SaleId.php, etc.
│   │   ├── Username.php                             # Normalizado a minúsculas
│   │   └── Role.php                                 # admin | seller
│   ├── Exception/                                   # Mensajes en español para el 422
│   │   ├── BusinessRuleViolation.php
│   │   ├── InsufficientStockException.php
│   │   ├── InvalidPriceException.php
│   │   ├── InvalidQuantityException.php
│   │   ├── EmptySaleException.php
│   │   ├── RepeatedProductException.php
│   │   ├── ProductNotFoundException.php
│   │   └── DuplicateUsernameException.php
│   └── Service/
│       └── .gitkeep
│
├── Application/                                     # ANILLO 2: Casos de uso
│   ├── Ports/
│   │   ├── Inbound/                                 # 5 puertos de entrada
│   │   │   ├── PlaceSale.php
│   │   │   ├── ManageProducts.php
│   │   │   ├── GetSales.php
│   │   │   ├── GetSalesReport.php
│   │   │   ├── Authenticate.php
│   │   │   ├── PlaceSaleCommand.php
│   │   │   ├── ProductView.php, SaleView.php
│   │   │   └── SalesReport.php, SalesReportRow.php
│   │   └── Outbound/                                # 10 puertos salientes
│   │       ├── ProductRepository.php
│   │       ├── SaleRepository.php
│   │       ├── CategoryRepository.php
│   │       ├── UserRepository.php
│   │       ├── FileStorage.php
│   │       ├── PasswordHasher.php
│   │       ├── TokenGenerator.php
│   │       ├── Clock.php
│   │       ├── UnitOfWork.php                       # Transacciones atómicas (callable)
│   │       └── SalesReportQuery.php
│   ├── UseCase/
│   │   ├── PlaceSaleService.php                     # T-10
│   │   ├── ProductCatalogService.php                # T-04
│   │   ├── GetSalesService.php                      # T-07
│   │   ├── SalesReportService.php                   # T-08
│   │   └── AuthenticationService.php                # T-06
│   └── Exception/
│       └── ConcurrencyConflict.php                  # ADR-002
│
├── Infrastructure/                                  # ANILLO 3: Implementaciones
│   ├── Persistence/
│   │   ├── Model/                                   # Eloquent (ProductModel, etc.)
│   │   ├── Mapper/                                  # ProductMapper (gestiona version)
│   │   ├── Repository/                              # EloquentProductRepository, etc.
│   │   └── LaravelUnitOfWork.php                    # DB::transaction() encapsulado
│   ├── Security/
│   │   ├── JwtTokenGenerator.php
│   │   └── Argon2PasswordHasher.php
│   ├── Storage/
│   │   └── LocalFileStorage.php
│   └── Configuration/
│       └── Settings.php
│
├── Presentation/                                    # ANILLO 4: HTTP
│   ├── Http/
│   │   ├── Controller/                              # 8 controladores REST
│   │   ├── Request/                                 # Validación de forma sintáctica
│   │   ├── Resource/                                # Transformación a JSON
│   │   └── ProblemDetails/                          # 3 cuerpos de error del contrato
│   └── Middleware/
│       ├── AuthenticateToken.php
│       └── RequireRole.php
│
└── Bootstrap/                                       # Punto de Ensamblaje
    └── PortBindingsServiceProvider.php              # Inyección de dependencias
```

---

## 5. El Dilema de la Transacción Atómica (`UnitOfWork`)

Para evitar que `Application` conozca a Laravel o llame a `DB::transaction()`, el spec define el puerto saliente `UnitOfWork`:

```php
namespace App\Application\Ports\Outbound;

interface UnitOfWork {
    public function run(callable $operation): mixed;
}
```

La implementación concreta reside en `Infrastructure`:
```php
namespace App\Infrastructure\Persistence;

use App\Application\Ports\Outbound\UnitOfWork;
use Illuminate\Support\Facades\DB;

final class LaravelUnitOfWork implements UnitOfWork {
    public function run(callable $operation): mixed {
        return DB::transaction($operation);
    }
}
```

El caso de uso `PlaceSaleService` coordina la operación atómica a través de la interfaz pura, cumpliendo estrictamente con la inversión de dependencias (DIP).

---

## 6. Manejo de Concurrencia y Datos Derivados

1. **Concurrencia Optimista (`version`):**
   - El campo `version` vive únicamente en la tabla SQL y en el `ProductModel` de Eloquent.
   - La entidad `Product` del dominio desconoce por completo la existencia de `version`.
   - `ProductMapper` y `EloquentProductRepository` verifican que al actualizar, el número de filas afectadas sea > 0 (`UPDATE product SET stock = ? WHERE id = ? AND version = ?`). Si devuelve 0, lanzan `ConcurrencyConflict`.
2. **Totales y Subtotales:**
   - La base de datos NO almacena columnas `total` ni `subtotal` (Artículo VII).
   - `Sale.total` se calcula sumando las líneas `SaleItem.subtotal = unit_price * quantity`.
