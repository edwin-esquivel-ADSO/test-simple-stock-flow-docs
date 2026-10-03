# ADR-010: Estrategia de Contenedores y Propietario del Esquema

## Estado
Aceptado

## Contexto
El reto se compone de 6 repositorios independientes. Es fundamental delimitar la frontera de responsabilidad de despliegue y persistencia.

## Decisión
1. `test-simple-stock-flow-infra` proporciona únicamente la orquestación (`docker-compose.yml`), volúmenes y el motor MySQL 8.4 vacío. No contiene scripts DDL ni migraciones.
2. `test-simple-stock-flow-api` es el dueño absoluto del esquema relacional mediante migraciones de Laravel.
3. Las dependencias (`vendor`, `node_modules`) se gestionan mediante volúmenes independientes para compatibilidad cruzada de sistemas operativos.

## Consecuencias
- Cero acoplamiento entre la infraestructura básica y las definiciones de tablas de negocio.
