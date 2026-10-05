# Metricas

## Metrica de negocio
**Tasa de citas duplicadas**

- Definicion: intentos de reserva rechazados por slot ocupado / total de intentos de reserva.
- Objetivo: 0% de citas duplicadas reales (los rechazos por 409 son OK).
- Fuente: logs del controller + tabla `citas`.
- Accion: si hay duplicados reales, revisar el constraint UNIQUE.

## Metrica de negocio (secundaria)
**Tasa de ausentismo (no-show)**

- Definicion: citas en estado RESERVADA que no se presentaron / total de citas confirmadas.
- Objetivo: menor a 15%.
- Fuente: estado de citas + registro de llegada.
- Accion: si supera, revisar la estrategia de recordatorios (mas anticipacion, SMS, etc.).

## Metrica tecnica
**Latencia p95 de POST /api/v1/citas**

- Definicion: percentil 95 del tiempo de respuesta del endpoint de reserva.
- Objetivo: menor a 300 ms (el constraint UNIQUE debe ser rapido).
- Fuente: Actuator + Micrometer.
- Accion: si supera, revisar indices en (medico_id, fecha_hora).
