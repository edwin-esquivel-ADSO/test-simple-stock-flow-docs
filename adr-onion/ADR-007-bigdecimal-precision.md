# ADR-007: Manejo de Precisión Decimal con Brick\Math\BigDecimal

## Estado
Aceptado

## Contexto
PHP no cuenta con un tipo primitivo para decimales de precisión fija. El uso de `float` está terminantemente prohibido para operaciones monetarias por riesgo de imprecisión en punto flotante.

## Decisión
Se utiliza la librería `brick/math` (`Brick\Math\BigDecimal`) para encapsular las operaciones monetarias dentro del Value Object `Money`.
- Escala: 2 decimales.
- Modo de redondeo: `RoundingMode::HALF_UP` (equivalente a Python `ROUND_HALF_UP`).
- Moneda fijada a `COP` (monomoneda).

## Consecuencias
- Exactitud matemática garantizada en totales e importes.
- El mapper rechaza importes con más de 2 decimales en lugar de redondear silenciosamente.
