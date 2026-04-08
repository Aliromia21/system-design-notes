# Horizontal vs Vertical Scaling

## Was ist es?

Scaling bedeutet ein System an wachsende Last anzupassen. Es gibt zwei grundlegende Ansätze: Vertical Scaling (stärkerer Server) und Horizontal Scaling (mehr Server).

## Vertical Scaling (Scale Up)

Mehr Ressourcen für einen einzelnen Server — mehr RAM, CPU, Speicher.

**Wann nutzen wir Vertical Scaling :**
- Datenbanken (PostgreSQL, MongoDB) — vertikales Scaling ist einfacher weil Transaktionen auf einem Server bleiben
- Frühe Phase eines Projekts — solange ein Server die Last bewältigt, ist es die einfachste Lösung
- Stateful Services — Services die viel lokalen State halten (z.B. WebSocket-Verbindungen)

**Grenzen:**
- Hardware hat ein Maximum — der stärkste Server hat auch ein Limit
- Kosten steigen exponentiell — doppelte CPU ≠ doppelter Preis, eher 3-4x
- Single Point of Failure — ein Server = ein Ausfall bringt alles runter

## Horizontal Scaling (Scale Out)

Mehr Instanzen des gleichen Services hinter einem Load Balancer.

**Wann nutzen wir Horizontal Scaling:**
- Stateless Services (REST APIs, Event Producer) — jeder Request ist unabhängig
- Kafka Consumer — Consumer Groups verteilen Partitions automatisch
- Read-heavy Workloads — mehrere Read Replicas der Datenbank

**Herausforderungen:**
- Load Balancing — wer entscheidet welcher Server den Request bekommt?
- Session Management — Sticky Sessions oder externes Session Store (Redis)
- Datenbank als Bottleneck — N App-Server schreiben alle in eine Datenbank
- Cache Invalidation — wenn Server 1 den Cache aktualisiert, weiß Server 2 nichts davon

## Vergleich

| Aspekt | Vertical | Horizontal |
|--------|----------|------------|
| Komplexität | Niedrig | Hoch |
| Kosten | Teuer bei großer Skalierung | Kosteneffizient |
| Obergrenze | Hardware-Limit | Theoretisch unbegrenzt |
| Ausfallsicherheit | Single Point of Failure | Redundanz eingebaut |
| Code-Änderungen | Keine | Oft nötig (Stateless Design) |
| Datenbank | Einfach | Komplex (Replikation, Sharding) |

## Bezug zu meinen Projekten

**ThreatStream:** Die Event API ist stateless — sie validiert, produziert an Kafka, und antwortet. Kein lokaler State. Perfekt für horizontales Scaling. Der Consumer skaliert horizontal über Kafka Consumer Groups — Kafka verteilt Partitions automatisch auf mehrere Consumer-Instanzen. Die Stats API hält WebSocket-Verbindungen (stateful) — hier ist vertikales Scaling der erste Schritt.

**Monitoring Platform:** Der BullMQ Worker skaliert horizontal — Redis garantiert durch Atomic Locking dass kein Job doppelt verarbeitet wird. Die API selbst ist stateless und könnte hinter einem Load Balancer laufen.

## Praxisregel

Starte vertikal. Skaliere horizontal wenn: (1) ein einzelner Server die Last nicht mehr bewältigt, (2) du Ausfallsicherheit brauchst, oder (3) du geografisch verteilte User hast.