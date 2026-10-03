# ADR-008: Verificación Estricta de Dependencias entre Capas

## Estado
Aceptado

## Contexto
Es común que al usar Laravel, los desarrolladores importen inadvertidamente fachadas (`DB`, `Auth`), modelos de Eloquent o clases de infraestructura dentro de Dominio o Aplicación.

## Decisión
Se establecen verificaciones mecánicas y pruebas de reflexión:
1. `grep -R "Illuminate\\\\" app/Domain` debe retornar 0 resultados.
2. Pruebas unitarias de Dominio se ejecutan sin instanciar el kernel de Laravel.
3. Controladores de Presentación solo tienen permitido inyectar puertos de entrada (`Inbound Ports`).

## Consecuencias
- La integridad de la cebolla es demostrable ante cualquier evaluador técnico.
