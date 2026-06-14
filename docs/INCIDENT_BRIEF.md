# INCIDENT BRIEF: Estabilización Retail X - Temporada Alta

## 1. Resumen Ejecutivo
Durante el pico de temporada de Retail X, se han detectado tres incidentes críticos: fallas en sincronización de pagos, errores de cálculo de interés (-12 cuentas) y brechas en la inactivación de usuarios. Disponemos de 72 horas y 42 horas/hombre totales de desarrollo para contener y solucionar el impacto operativo.

## 2. Alcance (In Scope / Out of Scope)
* **IN SCOPE:** - Mitigación y solución estructural de la cola de pagos.
  - Corrección de la base de datos para los 12 créditos con interés negativo.
  - Generación de evidencias de logs para la auditoría de usuarios.
* **OUT OF SCOPE:** - Refactorización completa del módulo de Onboarding.
  - Automatización del panel de auditoría de seguridad solicitado por el cliente.

## 3. Información Mínima para Reproducir
* **Pagos:** Generar una ráfaga de 50 peticiones simultáneas de webhooks mockeando éxito de pasarela y observar tasa de errores 500 por concurrencia.
* **Interés Negativo:** Filtrar en BD créditos con estado `MORA` y recalcular aplicando la fórmula del script actual con fechas de corte de fin de semana.

## 4. Flujo Esperado vs. Flujo Actual
* **Flujo Esperado:** Pasarela procesa pago -> Envía Webhook -> LeanCore procesa evento -> Saldo se actualiza en Dashboard inmediatamente.
* **Flujo Actual:** Pasarela procesa pago -> Envía Webhook -> Servidor saturado descarta petición (Drop) -> El pago se queda en la pasarela y no se ve en el Dashboard.

## 5. Criterios de Aceptación
* El 100% de los webhooks recibidos deben responder un HTTP 200 (recibido) y guardarse en una tabla de auditoría antes de procesarse.
* Ningún crédito en mora puede tener un saldo de interés menor a $0.00$.

## 6. Riesgos y Dependencias Externas
* Dependencia del Sandbox o logs de la Pasarela de Pagos externa para validar tiempos de respuesta de los Webhooks.


