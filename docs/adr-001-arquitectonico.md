# ADR-001: Prevencion de doble reserva mediante constraint unico en BD

## Estado
Aceptada

## Contexto
El sistema administra citas medicas. El problema principal es que dos personas
pueden intentar reservar el mismo horario con el mismo medico al mismo tiempo.
Los intentos de solucion a nivel de aplicacion (SELECT + INSERT) tienen race
conditions: ambos leen "libre" y ambos insertan.

## Decision
Usar un **constraint UNIQUE compuesto en PostgreSQL** sobre `(medico_id, fecha_hora)`.

Flujo:
1. La app intenta insertar la cita.
2. Si el slot esta libre, la BD acepta (201 Created).
3. Si ya esta ocupado, la BD lanza `DataIntegrityViolationException`.
4. El servicio lo captura y lanza `SlotOcupadoException`.
5. El controller lo traduce a **HTTP 409 Conflict** con mensaje claro.

Adicionalmente, un modulo de recordatorios asincronos via RabbitMQ.

## Consecuencias
**Positivas:**
- Imposible doble reserva, incluso con race conditions.
- No requiere locks pesimistas ni transacciones serializables.
- Simple de entender y auditar.
- El scheduler de recordatorios es independiente.

**Negativas:**
- El mensaje de error depende del motor de BD (DataIntegrityViolationException).
- Si se necesita reservar slots con duracion variable, el constraint es mas complejo.

**Mitigaciones:**
- Capturar la excepcion y traducirla a mensaje de negocio.
- Si en el futuro se admiten slots de distinta duracion, usar tabla `slots_disponibles`
  pre-generados y reservar sobre esa tabla.
