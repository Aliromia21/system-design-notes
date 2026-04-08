# Load Balancers

## Was ist es?

Ein Load Balancer verteilt eingehenden Netzwerk-Traffic auf mehrere Server. Er sitzt zwischen Client und Backend und entscheidet welcher Server den nächsten Request bekommt.

## Algorithmen

### Round Robin
Verteilt Requests reihum: Server 1 → 2 → 3 → 1 → 2 → 3...
- **Vorteil:** Einfachste Implementierung
- **Nachteil:** Ignoriert aktuelle Serverlast

### Least Connections
Request geht an den Server mit den wenigsten aktiven Verbindungen.
- **Vorteil:** Reagiert auf tatsächliche Last
- **Nachteil:** Muss Connection-Count tracken

### IP Hash
Client-IP bestimmt den Zielserver. Gleiche IP → immer gleicher Server.
- **Vorteil:** Session-Affinität ohne externe Session-Stores
- **Nachteil:** Ungleiche Verteilung bei wenigen hochaktiven IPs

### Weighted Round Robin
Wie Round Robin, aber leistungsstärkere Server bekommen proportional mehr Requests.
- **Vorteil:** Berücksichtigt unterschiedliche Server-Kapazitäten
- **Nachteil:** Gewichte müssen manuell konfiguriert werden

## Layer 4 vs Layer 7

| Aspekt | Layer 4 (Transport) | Layer 7 (Application) |
|--------|--------------------|-----------------------|
| Schaut auf | IP + Port | HTTP-Inhalt (URL, Headers, Cookies) |
| Routing | Nur nach IP/Port | Nach Pfad, Host, Header |
| Performance | Schneller (weniger Parsing) | Langsamer, aber intelligenter |
| Beispiel | AWS NLB, NGINX Stream | AWS ALB, NGINX, HAProxy |
| Use Case | TCP/UDP Traffic | HTTP APIs, WebSocket |

## Wann welchen Algorithmus?

- **Stateless APIs (REST):** Round Robin oder Least Connections
- **WebSocket-Server:** IP Hash (Sticky Sessions) — persistente Verbindungen müssen am gleichen Server bleiben
- **Heterogene Server:** Weighted Round Robin
- **Kafka Consumer:** Kein Load Balancer nötig — Kafka verteilt Partitions selbst

## Bezug zu meinen Projekten

**ThreatStream:**
- Event API: Round Robin — jede Instanz ist stateless, validiert Events identisch
- Stats API: IP Hash — WebSocket-Verbindungen brauchen Session-Affinität
- Consumer: Kafka Consumer Groups ersetzen den Load Balancer

**Monitoring Platform:**
- API: Round Robin möglich — stateless REST endpoints
- Worker: BullMQ + Redis verteilt Jobs automatisch — kein externer Load Balancer nötig

## Praxisregel

Starte ohne Load Balancer. Füge einen hinzu wenn:
 (1) ein Server die Last nicht bewältigt
 (2) du Ausfallsicherheit brauchstoder
 (3) du Zero-Downtime-Deployments willst.