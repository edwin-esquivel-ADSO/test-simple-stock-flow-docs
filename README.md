# test-simple-stock-flow-docs

> **Prueba técnica · Ficha ADSO 3413974**
> Horario: de **9:00 a. m. a 3:00 p. m.** (15:00)

Este repositorio contiene el **spec** de *Simple Stock Flow*. Es el único con contenido: los otros cinco empiezan vacíos.

## Instrucciones

Cada aprendiz debe **crear el fork** de los seis repositorios del proyecto y **resolver el proyecto
con el spec planteado**.

1. Hacer fork, a su cuenta de GitHub, de cada repositorio de la tabla del final.
2. Leer el spec en [`test-simple-stock-flow-docs`](https://github.com/code-sena/test-simple-stock-flow-docs).
   Se entrega en dos versiones: `spec-python/` y `spec-.net/`.
3. Desarrollar en los forks.

## El reto se desarrolla con React y PHP (Laravel)

El spec está escrito para Python y para .NET, pero el reto **no** se hace en esos lenguajes:

| Capa | Tecnología del reto |
|---|---|
| Frontend | React |
| Backend | PHP con Laravel |

Lo que el spec define sobre el negocio —historias, criterios de aceptación, reglas, contrato de la
API, modelo de datos— se respeta. Lo que define sobre la tecnología se traduce a React y Laravel.

## La prueba no consiste en escribir el código

El propósito principal es ver la **capacidad de desempeño con SDD** (*Spec-Driven Development*,
desarrollo guiado por especificación): cómo se lee, se interpreta y se aplica una especificación
para llevarla a un stack distinto. El código es el medio, no el fin.

## Qué contiene este repositorio

| Carpeta | Contenido |
|---|---|
| [`spec-python/`](spec-python/) | El spec de Simple Stock Flow escrito para un backend Python |
| [`spec-.net/`](spec-.net/) | El mismo sistema especificado para un backend .NET |

Las dos versiones describen el **mismo producto** (las mismas historias, reglas de negocio y
endpoints); cambian las decisiones de tecnología. Ninguna de las dos es el stack del reto.

## Cómo se lee el spec

En cualquiera de las dos carpetas, en este orden:

1. `constitution.md` — principios innegociables.
2. `spec.md` — qué debe hacer el sistema: actores, historias, criterios de aceptación, reglas de negocio.
3. `plan.md`, `architecture.md` y `arquitectura-panoramica.md` — cómo se construye.
4. `data-model.md` y `api-contract.md` — el modelo de datos y el contrato de la API.
5. `tasks.md` — qué hay que hacer y con qué evidencia se da por hecho.
6. `adr/` — las decisiones de arquitectura, con sus alternativas y consecuencias.

## Los seis repositorios

| Repositorio | Qué va ahí |
|---|---|
| [`test-simple-stock-flow-docs`](https://github.com/code-sena/test-simple-stock-flow-docs) | El spec: `spec-python/` y `spec-.net/` |
| [`test-simple-stock-flow-api`](https://github.com/code-sena/test-simple-stock-flow-api) | Backend en PHP (Laravel) |
| [`test-simple-stock-flow-app`](https://github.com/code-sena/test-simple-stock-flow-app) | Frontend en React |
| [`test-simple-stock-flow-page`](https://github.com/code-sena/test-simple-stock-flow-page) | Sitio público estático de presentación |
| [`test-simple-stock-flow-infra`](https://github.com/code-sena/test-simple-stock-flow-infra) | Contenedores, red, volúmenes y motor de base de datos vacío |
| [`test-simple-stock-flow-tool`](https://github.com/code-sena/test-simple-stock-flow-tool) | Utilidades: sembrador de datos de demostración |

---

# Hub Central de Documentación — Solución

## 1. El Reto SDD (Spec-Driven Development) y Traducción a Onion

El desafío consistió en aplicar la metodología **SDD (Desarrollo Guiado por Especificación)**: interpretar especificaciones abstractas y modelos de referencia originalmente diseñados en .NET y Python para construirlos e implementarlos en un stack completamente distinto:
- **Backend:** PHP 8.2+ con Laravel 11 estructurado bajo **Arquitectura Onion (Cebolla)** pura.
- **Frontend:** React 18 con Vite bajo **Arquitectura Onion** desacoplada mediante puertos.
- **Base de Datos:** MySQL 8.4 LTS con 9 restricciones `CHECK` físicas y soberanía del esquema.
- **Infraestructura:** Docker Compose multi-contenedor con red privada aislada.

---

## 2. Mapa Integral del Ecosistema (Diagrama C4 Contenedores)

```
                            SISTEMA COMPLETO SIMPLE STOCK FLOW
                            
            ┌────────────────────────────────────────────────────────┐
            │                  CLIENTES EXTERNOS                     │
            │  Navegador Web / Administradores / Vendedores          │
            └───────────┬────────────────────────────────┬───────────┘
                        │                                │
            Puerto 8085 │                    Puerto 8080 │
                        ▼                                ▼
       ┌─────────────────────────┐          ┌─────────────────────────┐
       │ test-simple-stock-flow- │          │ test-simple-stock-flow- │
       │ page (Landing Pública)  │          │ app (Frontend React)    │
       │ Nginx / HTML5 / CSS3    │          │ Arquitectura Onion      │
       └─────────────────────────┘          └────────────┬────────────┘
                                                         │
                                        Llamadas HTTP    │  Puerto 8000
                                        Bearer JWT       ▼
                                            ┌─────────────────────────┐
                                            │ test-simple-stock-flow- │
                                            │ api (Backend Laravel)   │
                                            │ Arquitectura Onion      │
                                            └──────┬───────────┬──────┘
                                                   │           │
                          Inyección CLI            │           │  Almacenamiento
                          vía HTTP                 │           │  de fotos
            ┌─────────────────────────┐            │           ▼
            │ test-simple-stock-flow- │            │     ┌──────────────────┐
            │ tool (CLI Seeder)       ├────────────┘     │ api_storage (Vol)│
            │ Cliente PHP / cURL      │                  └──────────────────┘
            └─────────────────────────┘            Puerto 3306
                                                   │
                                                   ▼
                                            ┌─────────────────────────┐
                                            │ test-simple-stock-flow- │
                                            │ infra (MySQL 8.4 DB)    │
                                            │ Esquema de la API       │
                                            └─────────────────────────┘
```

---

## 3. Matriz de Documentos Maestros en este Repositorio

| Documento | Ubicación | Descripción Técnica |
|---|---|---|
| **Arquitectura Onion Oficial** | [`architecture-onion.md`](architecture-onion.md) | Especificación canónica de los 4 anillos concéntricos, el Composition Root y la pureza de dominio en Laravel. |
| **Mapeo SDD 1:1** | [`spec-laravel-onion/mapping-sdd.md`](spec-laravel-onion/mapping-sdd.md) | Matriz de trazabilidad exhaustiva de cada artículo de la constitución, regla de negocio y tarea hacia el código. |
| **ADR-005: Onion Architecture** | [`adr-onion/ADR-005-onion-architecture.md`](adr-onion/ADR-005-onion-architecture.md) | Decisión de diseño: adopción de anillos concéntricos en sustitución del doble hexágono. |
| **ADR-006: Ports in Application**| [`adr-onion/ADR-006-ports-in-application.md`](adr-onion/ADR-006-ports-in-application.md) | Definición formal de los 5 puertos Inbound y 10 puertos Outbound. |
| **ADR-007: BigDecimal Precision**| [`adr-onion/ADR-007-bigdecimal-precision.md`](adr-onion/ADR-007-bigdecimal-precision.md) | Precisión matemática exacta sin errores de coma flotante IEEE 754. |
| **ADR-008: Layer Verification** | [`adr-onion/ADR-008-layer-verification.md`](adr-onion/ADR-008-layer-verification.md) | Automatización de pruebas arquitectónicas para blindar las capas contra imports indebidos. |
| **ADR-009: Laravel Isolation** | [`adr-onion/ADR-009-laravel-isolation.md`](adr-onion/ADR-009-laravel-isolation.md) | Regla de cero acoplamiento del framework en el núcleo del dominio. |
| **ADR-010: Docker Strategy** | [`adr-onion/ADR-010-docker-strategy.md`](adr-onion/ADR-010-docker-strategy.md) | Inicialización de base de datos MySQL vacía y soberanía de migraciones en la API. |

---

## 4. Las 12 Reglas de Negocio Implementadas y Validadas

1. **RN-01 (Stock no negativo):** Controlado por el agregado `Product` y reforzado con el constraint físico `ck_product_stock_non_negative`.
2. **RN-02 (Precio positivo):** Value Object `Money` y constraint `ck_product_price_positive`.
3. **RN-03 (Cantidad positiva):** Value Object `Quantity` y constraint `ck_sale_item_quantity_positive`.
4. **RN-04 (Venta no vacía):** Agregado `Sale` valida un mínimo de 1 línea de venta.
5. **RN-05 (No repetición de producto):** Validación en `Sale` y restricción única `uq_sale_item_sale_product`.
6. **RN-06 (Congelamiento histórico):** `SaleItem` copia el nombre, categoría y precio unitario al momento de vender.
7. **RN-07 (Inmutabilidad de venta):** No existen métodos de edición, actualización ni eliminación sobre ventas.
8. **RN-08 (Baja lógica):** Eliminación por marca de tiempo (`deleted_at`), protegiendo integridad con `RESTRICT`.
9. **RN-09 (Monomoneda COP):** Moneda fija `"COP"`.
10. **RN-10 (Username normalizado):** Normalización forzada a minúsculas y constraint `ck_user_username_normalized`.
11. **RN-11 (Roles admin y seller):** Restringido al conjunto cerrado `{'admin', 'seller'}` por VO y constraint físico.
12. **RN-12 (Total derivado):** El total se calcula dinámicamente como suma de subtotales; nunca se persiste en base de datos.
