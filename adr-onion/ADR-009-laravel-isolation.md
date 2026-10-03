# ADR-009: Aislamiento del Framework Laravel

## Estado
Aceptado

## Contexto
Laravel fomenta patrones ActiveRecord y Facades globales que tienden a acoplar la lógica de negocio al framework.

## Decisión
- Los modelos Eloquent se denominan `*Model` (ej. `ProductModel`) y residen en `App\Infrastructure\Persistence\Model\`.
- Las entidades de negocio son clases PHP puras en `App\Domain\Model\`.
- Se utilizan Mappers dedicados (`ProductMapper`, `SaleMapper`) para la conversión bidireccional entre Eloquent y Dominio.
- El manejo de transacciones se delega al puerto `UnitOfWork` implementado por `LaravelUnitOfWork`.

## Consecuencias
- Si el framework o la base de datos cambian, el Dominio no se ve afectado.
