# App 3 — Citas médicas

Sistema para una clínica que administraba citas por teléfono y hojas de cálculo. Ahora busca disponibilidad, reserva, cancela y envía recordatorios sin reservas duplicadas.

## 📑 Entregable

| Sección | Enlace |
|---|---|
| Objetivo, actores y alcance | [§1](#1-objetivo-actores-y-alcance) |
| Requisitos funcionales y de calidad | [§2](#2-requisitos) |
| Diagrama C4 contexto | [docs/c4-contexto.mmd](docs/c4-contexto.mmd) |
| Diagrama C4 contenedores | [docs/c4-contenedores.mmd](docs/c4-contenedores.mmd) |
| Flujo de operación crítica | [docs/flujo-critico.mmd](docs/flujo-critico.mmd) |
| Stack con justificación | [§3](#3-stack-propuesto) |
| ADRs | [ADR-001](docs/adr-001-arquitectonico.md), [ADR-002](docs/adr-002-tecnologico.md) |
| Riesgos | [docs/riesgos.md](docs/riesgos.md) |
| Métricas | [docs/metricas.md](docs/metricas.md) |

---

## 1. Objetivo, actores y alcance

### Objetivo
Evitar reservas duplicadas y garantizar que los pacientes reciban recordatorios de sus citas. Permitir consultar disponibilidad, reservar, cancelar y reprogramar.

### Actores
- **Paciente** — busca disponibilidad, reserva y cancela citas.
- **Recepcionista** — gestiona agenda en nombre del paciente.
- **Médico** — consulta su agenda.
- **Proveedor de email/SMS** — envía recordatorios (sistema externo).

### Alcance
**Dentro:**
- Consulta de disponibilidad por médico y fecha (slots de 30 min).
- Reserva con prevención de doble booking.
- Cancelación y reprogramación.
- Recordatorios automáticos 24 h antes (email/SMS).

**Fuera:**
- Historia clínica electrónica.
- Facturación.
- Integración con seguros médicos.

---

## 2. Requisitos

### Funcionales
| ID | Requisito |
|---|---|
| RF-01 | Listar médicos activos y sus especialidades |
| RF-02 | Consultar disponibilidad por médico y fecha |
| RF-03 | Reservar cita con validación de slot único |
| RF-04 | Rechazar doble reserva con HTTP 409 |
| RF-05 | Permitir cancelar citas en estado RESERVADA |
| RF-06 | Enviar recordatorio asíncrono 24 h antes |
| RF-07 | Scheduler cada 5 min busca citas próximas |
| RF-08 | Marcar `recordatorio_enviado=true` tras éxito |

### De calidad
| ID | Requisito |
|---|---|
| RNF-01 | Health check `/actuator/health` con DB y broker |
| RNF-02 | Latencia p95 de `POST /citas` menor a 300 ms |
| RNF-03 | Frontend responsive con calendario |
| RNF-04 | Docker Compose multi-servicio reproducible |
| RNF-05 | Datos de paciente no sensibles (solo contacto) |

---

## 3. Stack propuesto

| Capa | Tecnología | Justificación |
|---|---|---|
| Backend | Spring Boot 3.4 (Java 21) | Ecosistema maduro, JPA, Spring AMQP, @Scheduled |
| BD | PostgreSQL 16 | Unique constraint compuesto, ACID |
| Mensajería | RabbitMQ 3.13 | Colas durables, DLQ, reintentos |
| Scheduler | Spring @Scheduled | Job cada 5 min sin necesidad de CronJob externo |
| Frontend | HTML/CSS/JS vanilla | Demo funcional |
| Contenedores | Docker + Compose | Multi-servicio reproducible |

Detalle completo en [ADR-002](docs/adr-002-tecnologico.md).

---

## 4. Ejecución rápida

```bash
docker compose up -d --build
# http://localhost:8082
