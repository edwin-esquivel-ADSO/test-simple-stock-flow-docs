# ADR-005: Adopción de Arquitectura Onion en Laravel

## Estado
Aceptado

## Contexto
El reto técnico exige transformar la especificación de referencia (orientada a Arquitectura Hexagonal con puertos y adaptadores entrantes/salientes) a una Arquitectura Onion (Cebolla) concéntrica.

## Decisión
Se estructura el backend en 4 anillos concéntricos:
1. **Domain:** Entidades, Value Objects y Excepciones puras de negocio. Cero frameworks.
2. **Application:** Casos de uso, DTOs y definición de contratos (Puertos Inbound y Outbound).
3. **Infrastructure:** Adaptadores de persistencia (Eloquent, Mappers), seguridad, almacenamiento y servicios externos.
4. **Presentation:** Controladores HTTP, Form Requests y API Resources.
5. **Bootstrap (Punto de Ensamblaje):** Service Provider para inyección de dependencias.

## Consecuencias
- Las dependencias fluyen estrictamente hacia el centro.
- Dominio queda 100% aislado y testeable sin Laravel ni base de datos.
