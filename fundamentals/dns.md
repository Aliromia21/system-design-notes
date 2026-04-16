# DNS (Domain Name System)

## Was ist es?

DNS übersetzt Domainnamen (github.com) in IP-Adressen. 
Es ist das Telefonbuch des Internets.

## Wie funktioniert eine DNS-Abfrage?

1. Browser prüft lokalen Cache
2. Betriebssystem prüft /etc/hosts
3. Recursive Resolver (8.8.8.8) wird gefragt
4. Resolver fragt Root Server → TLD Server → Authoritative Server
5. Antwort wird gecacht und zurückgegeben

Gesamtdauer: ~20-50ms (gecacht: <5ms)

## DNS Record Types

| Type | Funktion | Beispiel |
|------|----------|----------|
| A | Name → IPv4 | github.com → 140.82.121.3 |
| AAAA | Name → IPv6 | github.com → 2606:50c0:8000::64 |
| CNAME | Name → anderer Name | www.github.com → github.com |
| MX | Mail Exchange | gmail.com → smtp.google.com |
| NS | Name Server | github.com → ns1.github.com |
| TXT | Verifikation | SPF, DKIM Records |

## DNS als Load Balancer

Mehrere A Records = DNS Round Robin. Einfach aber primitiv:
- Kein Health Check
- Kein Gewicht
- Client cached die IP

## TTL

| Szenario | TTL | Warum |
|----------|-----|-------|
| Stabile Infrastruktur | 3600 (1h) | Wenig Traffic |
| Häufige Änderungen | 60-300 | Schnelle Propagation |
| Migration | 30-60 | Sofortige Umschaltung |

## Bezug zu ThreatStream

In Produktion:
- threatstream.dev → CNAME → Vercel CDN (Dashboard)
- api.threatstream.dev → A → Load Balancer → Event API
- ws.threatstream.dev → A → Stats API (WebSocket)

DNS ist die erste Routing-Schicht.