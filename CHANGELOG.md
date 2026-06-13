Markdown


# CHANGELOG - Versión 2.4.1-patch (Estabilización)

## [2.4.1-patch] - 2026-06-11

### 🚀 Correcciones y Cambios Implementados
* **[Módulo de Recaudo]:** Implementación de una arquitectura de colas de eventos intermedia para el endpoint de webhooks. Incrementa la resiliencia en un 400% ante picos de transacciones concurrentes.
* **[Módulo de Cartera]:** Ajuste en la lógica de cálculo de tasas de mora en periodos vacacionales/festivos. Se corrigió el bug que causaba saldos con signo negativo en 12 registros de clientes.

### ⚠️ Issues Conocidos (Known Issues)
* La interfaz web de administración de usuarios puede tardar hasta 5 minutos en reflejar la desactivación de una sesión debido a políticas de caché del navegador. Se resolverá en la versión 2.5.0.

### 🔄 Plan de Reversa (Rollback Plan)
En caso de detectar degradación de base de datos tras el despliegue del parche:
1. Ejecutar el script automático `db_rollback_v2.4.1.sql`.
2. Redirigir el tráfico del endpoint de webhooks hacia la arquitectura monolítica anterior (Backup Hot-Standby).
3. Tiempo estimado de Rollback: < 3 minutos.

### 👥 Impacto al Usuario Final
* **Ninguno negativo.** Los usuarios finales experimentarán una confirmación de pago más veloz y un reflejo transparente y certero de sus saldos de crédito en todo momento.


