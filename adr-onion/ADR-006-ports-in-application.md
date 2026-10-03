# ADR-006: Ubicación de Puertos en la Capa Application

## Estado
Aceptado

## Contexto
En la arquitectura Onion clásica de Palermo, las interfaces de repositorio suelen colocarse en la capa de Dominio. Sin embargo, los Artículos II y IV de la constitución del proyecto establecen que los contratos que el sistema requiere del exterior (`ports/outbound/`) son definidos por la capa de Aplicación.

## Decisión
Los puertos de entrada (`Inbound`) y de salida (`Outbound`) se ubican dentro de `App\Application\Ports\`.

## Consecuencias
- `Domain` se mantiene exclusivamente enfocado en el modelo puro y sus invariantes de agregado.
- `Application` posee los contratos que necesita para orquestar los casos de uso.
