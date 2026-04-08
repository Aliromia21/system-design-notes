# CDN (Content Delivery Network)

## Was ist es?

Ein CDN ist ein Netzwerk aus weltweit verteilten Servern (Edge Server), die Kopien von Inhalten zwischenspeichern und an User aus geografischer Nähe ausliefern.
Ziel: niedrige Latenz und hohe Verfügbarkeit.

## Wie funktioniert es?

1. User ruft `app.example.com/style.css` auf
2. DNS leitet zum nächstgelegenen CDN Edge Server (z.B. Frankfurt statt New York)
3. Edge Server prüft Cache:
   - **Cache Hit:** Datei ist vorhanden → sofort ausliefern (~5ms)
   - **Cache Miss:** Datei fehlt → vom Origin Server holen, cachen, ausliefern (~100ms beim ersten Mal)
4. Nächster Request für dieselbe Datei → Cache Hit

## Was wird gecacht?

| Inhalt | CDN geeignet? | TTL |
|--------|--------------|-----|
| JS/CSS Bundles | Perfekt | Monate (Filename-Hashing) |
| Bilder, Fonts | Perfekt | Wochen bis Monate |
| Statische HTML | Gut | Minuten bis Stunden |
| API-Responses | Teilweise | Sekunden (kurze TTL) |
| Personalisierte Daten | Nein | - |
| WebSocket | Nein | - |

## Wichtige Konzepte

**TTL (Time To Live):** Wie lange der CDN eine Datei behält bevor er beim Origin nachfragt. Längere TTL = weniger Origin-Traffic, aber langsamere Updates.

**Cache Invalidation:** Das schwierigste Problem. Strategien:
- Filename-Hashing (`app.a3f8b2.js`) — neuer Build = neuer Filename = automatischer Cache-Bypass
- Purge API — CDN-Cache manuell löschen (Cloudflare, CloudFront bieten das an)
- Versioned URLs — `/v2/api/data` statt `/api/data`

**Origin Shield:** Ein zusätzlicher Cache-Layer zwischen Edge Servern und Origin Server. Reduziert die Last auf den Origin bei Cache Misses von vielen Edge Servern gleichzeitig.

## CDN-Anbieter

| Anbieter | Stärke | Kostenlos? |
|----------|--------|-----------|
| Cloudflare | DNS + CDN + DDoS-Schutz | Ja (Free Tier) |
| AWS CloudFront | Integration mit AWS-Ökosystem | Nein (pay per use) |
| Vercel | Automatisch für Next.js/React | Ja (Hobby Tier) |
| Fastly | Edge Computing, sehr performant | Nein |

## Bezug zu meinen Projekten

**ThreatStream Dashboard:** Das React-Build (`npm run build`) produziert statische Dateien (JS, CSS, HTML). In Produktion würden diese über ein CDN ausgeliefert — User in München bekommt die Dateien vom Edge Server in Frankfurt, nicht vom Origin in der Cloud. API-Requests und WebSocket-Verbindungen gehen weiterhin direkt zum Stats API Server.

**Monitoring Platform:** Frontend ist auf Vercel deployed — Vercel nutzt automatisch ein globales CDN. Das ist der Grund warum die Seite weltweit schnell lädt, obwohl der Backend-Server auf Railway in einer Region läuft.

## Praxisregel

Statische Assets immer über CDN ausliefern. API-Responses nur cachen wenn sie sich selten ändern und nicht personalisiert sind. WebSocket und Echtzeit-Daten können nicht gecacht werden.