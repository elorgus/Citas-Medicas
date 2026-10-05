# Riesgos y mitigaciones

| # | Riesgo | Impacto | Probabilidad | Mitigacion |
|---|---|---|---|---|
| R1 | Doble reserva del mismo slot por race condition | Critico | Alta | Constraint UNIQUE(medico_id, fecha_hora) en BD + manejo de DataIntegrityViolationException + HTTP 409 |
| R2 | Recordatorio no enviado por fallo del proveedor de email/SMS | Alto | Media | Cola RabbitMQ con reintentos (2s, 4s, 8s) + DLQ + estado `recordatorio_enviado` en BD |
| R3 | Job @Scheduled no distribuido en multiples replicas | Medio | Media | ShedLock o lock en BD para garantizar un solo ejecutor; documentar limite actual |

## Detalle R1 - Doble reserva
**Mitigacion:** El constraint UNIQUE compuesto es la garantia de motor. La app captura la excepcion y devuelve 409 Conflict con mensaje. Se probo con dos curls simultaneos: uno gana (201), otro pierde (409).

## Detalle R2 - Recordatorio no enviado
**Mitigacion:** El consumer marca `recordatorio_enviado=true` solo despues de exito. Si falla, RabbitMQ reintenta. Tras 3 intentos, va a DLQ para revision manual. El scheduler puede reintentar en el proximo ciclo si el flag sigue en false.

## Detalle R3 - Job no distribuido
**Mitigacion:** En la primera version, una sola instancia. Si se escala, agregar ShedLock o un lock pesimista en una tabla `jobs_lock`. Documentado como deuda tecnica.
