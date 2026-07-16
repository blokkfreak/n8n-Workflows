# Directory Scraper Alert System

## Überblick

Das hier ist die Alert-Schicht eines größeren Immobilien-Monitoringsystems, das ich gebaut habe, um neu inserierte Grundstücke und Objekte in einer definierten Region in Niedersachsen zu tracken. Der vollständige Stack besteht aus:

- **Python-Scraper** – einer pro Datenquelle, laufen als geplante Prozesse auf einem VPS, den ich selbst aufgesetzt habe und betreue.
- **PostgreSQL-Datenbank** – ebenfalls self-hosted auf demselben VPS. Die Scraper schreiben neue Listings in eine zentrale `listings`-Tabelle.
- **Diese n8n-Workflows** – geplante Alert-Jobs, die die Datenbank abfragen, Ergebnisse als HTML-E-Mails formatieren, sie versenden und Records als exportiert markieren.

Die Workflows in diesem Ordner sind die Benachrichtigungs- und Export-Schicht. Das eigentliche Scraping findet hier nicht statt.

## Datenquellen

Vier separate Alert-Workflows decken unterschiedliche Portale und Regionen ab:

| Workflow                                        | Quelle                                                | Region                                                                             |
| ----------------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------------------------------------- |
| [iKISS Alert](./ikiss-alert/)                   | iKISS-Gemeindeportal                                  | Isenbüttel, Meinersen, Wesendorf, Lachendorf, Lehre, Papenteich                    |
| [KIP & Gemeinden Alert](./kip-gemeinden-alert/) | KIP-Plattform + Gemeinden Salzgitter & Wolfenbüttel   | Gifhorn, Peine, Braunschweig, Hannover, Celle, Salzgitter, Wolfenbüttel, Helmstedt |
| [NOLIS Alert](./nolis-alert/)                   | NOLIS-Gemeindeportal                                  | Wendeburg, Hankenbüttel, Vechelde, Edemissen, Flotwedel, Uetze, Burgdorf, Lehrte   |
| [ZVG Alert](./zvg-alert/)                       | zwangsversteigerung.de (Zwangsversteigerungsregister) | Braunschweig, Gifhorn, Celle, Wolfsburg, Salzgitter, Helmstedt, Peine              |

## Systemarchitektur

```
  Python-Scraper (VPS)
  iKISS / KIP / NOLIS / ZVG
         │
         ▼
  PostgreSQL
  (Feld: exported_at trackt, was bereits versendet wurde)
         │
    ┌────┴────────────────────────┐
    │                             │
  DB-basierte Alerts         ZVG Alert
  (iKISS, KIP, NOLIS)        liest JSONL-Datei,
                              die der Scraper schreibt
```

## Export-Tracking

Alle drei datenbankbasierten Workflows nutzen eine `exported_at`-Timestamp-Spalte, um zu tracken, was bereits versendet wurde. Die Query holt nur Zeilen, bei denen `exported_at IS NULL`. Nach einem erfolgreichen Alert markiert ein generiertes `UPDATE`-Statement die betroffenen Zeilen mit `NOW()`. So erscheint jedes Listing in genau einer Alert-E-Mail, unabhängig davon, wie oft der Scraper gelaufen ist.

Der ZVG-Workflow nutzt n8n's eingebaute Workflow Static Data, um eine Liste gesehener Record-Keys über Ausführungen hinweg zu persistieren – da der ZVG-Scraper eine Rolling-JSONL-Datei statt einer Datenbank schreibt.

## Alert-Format

Jeder Workflow generiert eine gestaltete HTML-E-Mail mit einer Tabelle neuer Listings: Titel, Gemeinde/Landkreis, Preis (als `k EUR` formatiert für Werte ≥ 10.000), Datenquelle und Link zum Originaleintrag. Alerts werden nur versendet, wenn es neue Listings zu melden gibt; andernfalls beendet der Workflow am Condition-Node sauber ohne weiteren Output.

Ergebnisse werden außerdem in separate Tabs einesr Google Sheet geschrieben.

## Dateien

```
real-estate-scraper-alerts/
├── README.md
├── ikiss-alert/
│   └── workflow.json
├── kip-gemeinden-alert/
│   └── workflow.json
├── nolis-alert/
│   └── workflow.json
└── zvg-alert/
    └── workflow.json
```


