A1.
Análisis del Problema e Hipótesis
Incidente 1: Pagos en pasarela no reflejados en el Dashboard
Módulos involucrados: Integraciones (Pasarela de pagos externa), Core/Dashboard.
Hipótesis de Causa: El webhook de la pasarela de pagos está fallando (error 500), timeout en la API de LeanCore, o pérdida de eventos asíncronos debido a la alta transaccionalidad de la temporada.
Cómo validar: Revisar los logs (audit trail) de la pasarela de pagos (reintentos de webhooks fallidos) y los logs de la API de LeanCore en el endpoint de recaudo.
Impacto si se confirma (cliente): Alto. Desconfianza del usuario final, duplicidad de pagos y fricción en servicio al cliente.
Alternativa operativa temporal: Habilitar un proceso de conciliación manual al final del día. Implementar un fallback para alertar si vuelve a pasar. Comunicación a los clientes / usuarios sobre la latencia del servicio.

Incidente 2: Reporte de cartera con 12 créditos en "interés negativo"
Módulos involucrados: Reportería/dashboard (operativa/financiera), Administración de cartera, producto y tecnología (backend).
Hipótesis de Causa: Un error en la fórmula del script que calcula el interés compuesto/mora cuando se acumulan días festivos o un desbordamiento/bad data introducido por una actualización reciente en el backend.
Cómo validar: Ejecutar una consulta matemática directa (query) en la base de datos para esos 12 IDs de crédito y comparar el cálculo real vs. el mostrado en el reporte.
Impacto si se confirma: Crítico para el negocio (pérdida financiera o problemas regulatorios), pero acotado en volumen (solo 12 créditos por ahora confirmados).
Alternativa operativa temporal: Excluir temporalmente esos 12 registros del reporte automatizado y enviarle al cliente un anexo corregido manualmente en Excel/dashboard mientras se corrige el bug.

Incidente 3: Usuarios inactivos aparecen como activos (Auditoría)
Módulos involucrados: Core, Onboarding y accesos.
Hipótesis de Causa: Falta de propagación de estados lógicos en la base de datos (se cambia a inactivo un flag pero no se invalida la sesión/token JWT) o un error de caché en el Frontend.
Cómo validar: Desactivar un usuario de prueba en el ambiente de Staging (QA) y verificar si su token de acceso sigue permitiendo peticiones HTTP a la API.
Impacto si se confirma: Medio-Bajo a corto plazo (riesgo de seguridad/cumplimiento, pero no detiene la operación comercial del pico de temporada).
Alternativa operativa temporal: Realizar una purga o inactivación masiva de sesiones desde el backend por consola de comandos y proveer al cliente el log de auditoría crudo.
A2.
Propuesta de Solución para el Incidente de Pagos
Mitigación Inmediata (Workaround): Implementar un script de conciliación automática (cron job) que corra cada 2 horas, consulte la API de la pasarela por los pagos exitosos del día e impacte los saldos que el webhook no procesó.
Solución Estructural (Causa Raíz - war room): Implementar una arquitectura orientada a eventos con una cola de mensajería (ej. AWS SQS o RabbitMQ). Cuando el webhook de la pasarela llegue, se encola de inmediato garantizando que ningún pago se pierda, procesándose de forma asíncrona y con reintentos automáticos (Idempotencia).
A3.
Conocimiento del Producto (Respuestas Cortas)
Diferencia AppWeb vs. Dashboard: La AppWeb es de cara al usuario final (cliente final que simula y solicita el crédito) ; el Dashboard es la herramienta interna del cliente B2B (Retail X) para administrar la cartera, gestionar riesgos y operar el negocio.
Webhook en Recaudo: Es una notificación asíncrona HTTP POST enviada por la pasarela hacia LeanCore cuando un pago cambia a "exitoso". Es clave porque evita tener que hacer consultas repetitivas (polling) a la pasarela, optimizando recursos de red.
Riesgos de fuentes externas en reportes de cartera: Inconsistencia de datos, latencia en la generación de reportes si la API externa cae y falta de control sobre la veracidad del dato de origen.
Retos NoSQL sin gobierno de datos: Flexibilidad extrema que deriva en esquemas inconsistentes (ej. un registro guarda el interés como float y otro como string), dificultando agregaciones y cálculos financieros exactos en reportes.
Fallas comunes en API-first: Desalineación en los contratos de la API (breaking changes no comunicados), problemas de autenticación/tokens expirados entre servicios y manejo deficiente de timeouts/caídas de los endpoints de terceros.
