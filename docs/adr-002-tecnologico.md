# ADR-002: Stack Spring Boot + PostgreSQL + RabbitMQ + Scheduler

## Estado
Aceptada

## Contexto
El sistema necesita:
- API REST para que pacientes, recepcionistas y medicos gestionen citas.
- Persistencia transaccional con garantia de no-duplicidad.
- Envio de recordatorios 24h antes de la cita.
- Independencia entre la reserva (sincrona) y el envio del recordatorio (asincrono).

## Decision
- **Spring Boot 3.4 + Java 21**: ecosistema maduro, JPA, Spring AMQP, @Scheduled.
- **PostgreSQL 16**: unique constraint compuesto, transacciones ACID.
- **RabbitMQ 3.13**: colas durables + DLQ para recordatorios.
- **@Scheduled**: job cada 5 min que busca citas en ventana +23h/+25h.
- **HTML/JS vanilla**: demo sin build step.

## Consecuencias
**Positivas:**
- El constraint UNIQUE es la garantia mas fuerte contra doble reserva.
- El scheduler corre en la misma app (no requiere CronJob externo).
- El consumer de recordatorios es idempotente (verifica recordatorio_enviado).

**Negativas:**
- Spring @Scheduled no es distribuido: si se escala la app a N replicas,
  todas ejecutarian el job. Requiere ShedLock o similar.
- El proveedor de email/SMS esta simulado en la demo.

**Alternativas consideradas:**
- Django + PostgreSQL: descartado por consistencia con el resto de apps.
- Lock pesimista (SELECT ... FOR UPDATE): mas complejo, peor performance.
- Redis con locks distribuidos: overkill para la escala actual.
