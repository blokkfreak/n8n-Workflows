# SMTP DMARC Report Verarbeitung

## Hintergrund

Als verantwortliche Person für die E-Mail-Infrastruktur der Agentur (SMTP, DKIM, SPF und DMARC-Records für Kundendomains über Plesk einrichten und pflegen) brauchte ich eine zuverlässige Möglichkeit zu wissen, wann in der Mail-Konfiguration etwas nicht stimmt. E-Mail-Anbieter wie Google, Microsoft und andere schicken DMARC-Aggregatberichte als XML-Anhang per E-Mail. Aber die manuell über mehrere Domains und Sender hinweg zu lesen ist zeitintensiv.

Dieser Workflow macht aus diesem rohen Datenstrom automatisch ein strukturiertes, abfragbares Fehlerprotokoll, welches schnell abrufbar via GoogleSheets ist.

## Wie es funktioniert

Ein IMAP-Trigger pollt auf neue eingehende DMARC-Report-Mails (nur ungelesene Nachrichten). Wenn eine ankommt:

1. **Extraktion** – ZIP-Anhang wird über n8n-Compression-Node entpackt
2. **Buffer lesen** – JavaScript-Code-Node holt binären Dateibuffer für jedes eingehende Item und dekodiert ihn zu lesbaren Text, auch wenn mehrere DMARC-Reports im gleichen Abruf-Zyklus ankommen
3. **XML-Parsing** – XML-Node parst DMARC-Aggregatstruktur in strukturiertes JSON
4. **Dreistufige Analyse** – zweite JavaScript-Code-Node wertet jeden Record aus:
   - prüft offiziellen DMARC-Policy-Evaluation-Felder (`dkim` / `spf` pass oder fail)
   - scannt jeden einzelnen DKIM-Selector-Eintrag auf `fail`, `permerror` und `temperror` (fängt so kaputte Keys, die den Top-Level-Check noch bestehen)
   - scannt alle SPF-Auth-Ergebnisse inklusive `softfail`
5. **Selektive Ausgabe** – Nur Records mit mindestens einem Fehler kommen durch, angereichert mit Report-Datum, Provider-Name, Quell-IP, Fehlergrund und der Maßnahme des empfangenden Servers
6. **Logging** – Ergebnisse werden in ein Google Sheet geschrieben. Jede Domain bekommt ihren eigenen Tab; ein Error-Output leitet unbekannte Domains in ein separates Catch-all-Sheet.

## Warum das relevant ist

Ohne diesen Workflow hätte ein kaputter DKIM-Key oder eine falsche SPF-Konfiguration auf einer Kundendomain unbemerkt bleiben können, bis der Kunde sich über Zustellprobleme beschwert. Mit dem laufenden Workflow tauchten Fehler typischerweise innerhalb von Stunden nach Eingang eines Reports auf – Zeit genug, um zu untersuchen und zu beheben, bevor nennenswerte Mailmengen betroffen waren.

# 

## Stack

| Komponente | Technologie                                                    |
| ---------- | -------------------------------------------------------------- |
| Trigger    | IMAP (E-Mail, nur ungelesene Nachrichten)                      |
| Parsing    | Compression → JavaScript (Buffer) → XML → JavaScript (Analyse) |
| Ausgabe    | Google Sheets (domainweise Tabs, Append)                       |
