# "Rez-Tool"" – GBP-Rezensions-Generator

## Hintergrund

Die Marketing-Agentur, bei der ich gearbeitet habe, betreut ein Unternehmernetzwerk, dessen Mitglieder regelmäßig Kunden und Empfehlungen untereinander austauschen. Oft sind die Bewertungen schlicht an einer Schreibblockade gescheitert: Die Mitglieder waren zwar zufrieden, aber das leere Textfeld hat zu viel Zeit und Mühe gekostet.

Die Lösung war, diese Hürde komplett wegzunehmen. Der Ablauf ist folgender:

1. kurzes Feedback-Formular auf einer von mir selbst entwickelten Website ausfüllen (Sternebewertung, Highlights und Kritik per Multiple-Choice)
2. sofort per E-Mail einen fertigen, KI-generierten Rezensionstext erhalten
3. Über einen Direktlink in der E-Mail lässt sich der Text mit einem Klick im passenden Google Business Profile veröffentlichen

Der Workflow ist das Backend dieses Produkts. Er empfängt die Formulareingabe, routet sie durch den passenden Pfad, baut das Ergebnis zusammen und kümmert sich um die gesamte ausgehende Kommunikation.

## Wie es funktioniert

Ein Webhook empfängt das vom Website-Frontend gesendete Formular. Die Daten werden in einer JavaScript-Code-Node normalisiert (Slug-Formatierung, Typkonvertierung, Fallback-Werte), bevor sie auf den ersten Entscheidungspunkt treffen:

**Positive Rezensionen (≥ 4 Sterne)**

Die normalisierten Eingaben gehen an ein Mistral-Modell (via Ollama) mit einem eng eingegrenzten Prompt: 50–120 Wörter, erste Person, keine Emojis, keine einleitenden Labels – nur der Rezensionstext. Ein zweiter Gemini-Flash-Aufruf läuft parallel für einen Fallback-Zweig.

Ein Code-Node löst dann die Google Place ID des Mitglieds aus einem In-Memory-Slug-Mapping auf, baut die Direktlink-URL, wendet eine rekursive Bereinigungsschleife auf verbleibende LLM-Ausgabe-Artefakte an und baut das finale Datenobjekt zusammen. Das Ergebnis geht per HTML-E-Mail mit gebrandetem Layout und einem bedingten „Google-Bewertung öffnen"-Button an den Verfasser – der Button erscheint nur, wenn ein GBP-Link verfügbar ist. Der vollständige Datensatz wird per timestamp-basierter Deduplizierung in ein Google Sheet geschrieben zur Nachverfolgung.

**Kritische Rezensionen (< 4 Sterne)**

Ein separater Zweig löst gleichzeitig eine interne Support-Alert-Mail aus und sendet eine Bestätigungsmail an den Verfasser. Die Support-Mail enthält alle Formulardaten plus eine KI-generierte abgemilderte Textvariante, als HTML-Tabelle formatiert. Die Daten landen im gleichen Google Sheet.

In beiden Fällen antwortet der Workflow dem auslösenden Webhook mit einem JSON-Payload, damit die Website den generierten Text inline anzeigen kann.

## Wichtige Design-Entscheidungen

**GBP-Slug-Mapping im Code** – Google Place IDs werden direkt in einem JavaScript-Code-Node gepflegt statt per Google-Sheet-Lookup. Das eliminiert einen Netzwerk-Round-Trip pro Ausführung und hält die Daten nah an der Logik, die sie nutzt. Ein gruppiertes Alias-System behandelt Fälle, in denen eine Place ID mehrere Mitgliedsnamen abdeckt.

**Rekursive Ausgabe-Bereinigung** – LLM-Outputs enthalten manchmal restliche Labels oder Formatierungen, auch wenn der Prompt sie explizit verbietet. Eine `do-while`-Schleife wendet eine Regex-Bereinigung wiederholt an, bis die Ausgabe stabil ist – robust unabhängig vom Modellverhalten.

**Dual-Modell-Architektur** – Zwei LLMs verarbeiten den positiven Pfad (Ollama-hosted Mistral für die Hauptrezension, Gemini Flash für die Nebenvariante). Der kritische Pfad hat sein eigenes separates Paar. Damit ist kein KI-Service ein Single Point of Failure.

**Bedingtes E-Mail-Template** – Die HTML-E-Mail passt sich basierend darauf an, ob ein GBP-Link aufgelöst werden konnte – über eine einfache If-Else-Abfrage wird dynamisch zwischen CTA-Button und einem einfachen Hinweistext hin- und herschaltet.

# 

## Stack

| Komponente               | Technologie                      |
| ------------------------ | -------------------------------- |
| Trigger                  | Webhook (POST)                   |
| KI – Hauptrezension      | Ollama · Mistral Large 3 (lokal) |
| KI – Fallback / Variante | Google Gemini Flash 2.5          |
| Datenpersistenz          | Google Sheets (Append-or-Update) |
| E-Mail-Versand           | SMTP (HTML)                      |
| Website-Antwort          | n8n „Respond to Webhook"-Node    |
