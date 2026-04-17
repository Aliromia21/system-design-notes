# CAP Theorem

## Was ist es?

Das CAP Theorem besagt, dass ein verteiltes System nur zwei von drei Eigenschaften gleichzeitig garantieren kann: Consistency, Availability, Partition Tolerance.

## Die drei Eigenschaften

**Consistency:** Jede Leseoperation liefert den zuletzt geschriebenen Wert. Alle Nodes sehen die gleichen Daten zur gleichen Zeit.

**Availability:** Jeder Request bekommt eine Antwort — auch wenn die Daten möglicherweise nicht aktuell sind.

**Partition Tolerance:** Das System funktioniert weiter, auch wenn die Netzwerkverbindung zwischen Nodes unterbrochen wird.

## Warum nur zwei?

Bei einer Netzwerk-Partition muss das System entscheiden:
- **CP:** Konsistenz bewahren, aber Anfragen ablehnen (nicht verfügbar)
- **AP:** Verfügbar bleiben, aber möglicherweise veraltete Daten liefern

CA ist nur bei einem einzelnen Server möglich — in verteilten Systemen sind Partitions unvermeidlich.

## Beispiele

| System | Wahl | Begründung |
|--------|------|------------|
| PostgreSQL (single) | CA | Kein Partition möglich |
| PostgreSQL (replicas) | CP | Schreibt nur auf Primary |
| MongoDB (default) | CP | Schreibt nur auf Primary |
| Cassandra | AP | Schreibt auf jeden Node |
| Kafka | CP | Verweigert Writes ohne ISR |
| Redis (single) | CA | Ein Server |

## Eventual Consistency

In AP-Systemen werden Daten irgendwann konsistent — aber nicht sofort.
Beispiel: Ein Tweet erscheint in Japan 3 Sekunden später als in Deutschland.
Für Social Media akzeptabel, für Bankkonten nicht.

## Bezug zu ThreatStream

- **Kafka:** CP — verweigert Writes ohne In-Sync Replicas
- **PostgreSQL:** CA (single node), CP mit Replicas
- **Dashboard:** Toleriert Eventual Consistency — 1 Sekunde Delay für Analytics akzeptabel
- **WebSocket Batching:** 1-Sekunden-Intervall ist bewusst gewählt, kein Echtzeit-Zwang

## Praxisregel

Die meisten Systeme sind nicht rein CP oder AP. Sie sind konfigurierbar:
- MongoDB: Consistency Level pro Query wählbar
- Cassandra: Quorum-Level bestimmt CP vs AP pro Request
- Die richtige Frage ist nicht "CP oder AP?" sondern "Welche Consistency braucht dieser Use Case?"