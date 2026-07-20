# n8n Automation Workflows

Eine Sammlung produktiver Automatisierungsworkflows, gebaut in [n8n](https://n8n.io/). Jeder Workflow wurde entworfen und umgesetzt, um ein konkretes operatives oder fachliches Problem zu lösen – von der E-Mail-Infrastruktur-Überwachung bis zur KI-gestützten Kundenkommunikation.

Alle Workflows laufen auf einer selbst gehosteten n8n-Instanz.

---

## Projekte

| Projekt                                                     | Was es macht                                                                                                                                          | Stack                                                          |
| ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| [Rez-Tool](./rez-tool/)                                     | Generiert personalisierte Google-Rezensionen aus Nutzereingaben via LLM, verschickt sie per E-Mail und löst direkte Google Business Profile-Links auf | Webhook · Gemini Flash · Ollama/Mistral · Google Sheets · SMTP |
| [SMTP DMARC Report](./smtp-dmarc-report/)                   | Parst eingehende DMARC-Aggregatberichte, führt eine dreistufige DKIM/SPF-Analyse durch und schreibt Fehler in ein strukturiertes Google Sheet         | IMAP · XML · JavaScript · Google Sheets                        |
| [YT Feed Bot](./yt-feed-bot/)                               | Überwacht 25 YouTube-Kanäle per RSS alle 8 Stunden, fasst neue Videos per LLM zusammen und postet eine Zusammenfassung in Slack                       | RSS · OpenRouter · Mistral · Slack                             |
| [directory-scraper-alerts](./directory-scraper-alerts/) | Alert-System für vier Immobilien-Datenquellen in Niedersachsen, unterstützt durch eine selbstverwaltete PostgreSQL-Datenbank auf einem VPS            | PostgreSQL · SMTP · Google Sheets · Cron                       |

---

> **Hinweis zu sensiblen Daten**: Alle E-Mail-Adressen, Credential-IDs, Google Sheet-Dokument-IDs, API-Keys und ähnliches wurden durch beschreibende Platzhalter ersetzt. Die Workflows spiegeln die tatsächliche Produktionslogik wider.
