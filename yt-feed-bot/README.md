# YT Feed Bot – LLM-gestützte YouTube-Zusammenfassung

## Hintergrund

Entwicklungen im KI-Tooling passieren schnell genug, dass manuelles YouTube-Checken während des Arbeitstags keine zuverlässige Methode ist, um auf dem laufenden Stand zu bleiben. Ich wollte ein System, das die Kanäle, die mich interessieren, überwacht, das Rauschen herausfiltert und nur das Neue direkt in Slack gepostet wird.

## Wie es funktioniert

Ein Schedule-Trigger feuert alle 8 Stunden. Eine Set-Node definiert eine Liste von 25 YouTube-Kanal-RSS-URLs, die dann in einzelne Items gesplittet und in einem Batch-Loop verarbeitet werden.

Für jeden Kanal wird der RSS-Feed abgerufen und durch einen Filter-Node geschickt, der den Veröffentlichungs-Timestamp gegen `Date.now() - 8h` prüft. Nur Videos aus dem aktuellen Zeitfenster kommen durch.

Jedes passende Video durchläuft einen Batch-Loop, in dem eine Chain-LLM-Node Titel und die ersten 600 Zeichen der Beschreibung an Mistral (via OpenRouter) schickt. Der Prompt fordert eine Zusammenfassung auf Deutsch in 2-3 Sätzen – kein Markdown, keine Füllphrasen, nur den Kern. Ein 3-Sekunden-Delay zwischen den Batches verhindert Rate-Limit-Fehler.

Wenn der Loop durch ist, werden alle Zusammenfassungen in einem Datenobjekt aggregiert. Eine Set-Node formatiert die finale Slack-Nachricht: ein Timestamp-Header, dann jedes Video gruppiert nach Kanalname mit Titel, Zusammenfassung und Link, getrennt durch horizontale Linien. Die Nachricht geht in einen dedizierten Slack-Channel.

Falls im 8-Stunden-Fenster keine neuen Videos erschienen sind, sendet der Workflow trotzdem eine kurze Hinweis-Nachricht.

## Design-Entscheidungen

Der RSS-Ansatz vermeidet bewusst die YouTube Data API – kein Quota, keine Authentifizierung, und die Feeds sind aktuell genug für einen 8-Stunden-Rhythmus. Die Modellwahl (Mistral Ministral 3B via OpenRouter) ist eine Entscheidung für ein leichtgewichtiges, schnelles europäisches Modell für einen reinen Summarisierungstask ohne nennenswerte Kontextlängenanforderungen.

# 

## Stack

| Komponente  | Technologie                                    |
| ----------- | ---------------------------------------------- |
| Trigger     | Schedule (alle 8 Stunden)                      |
| Datenquelle | YouTube RSS-Feeds (kein API-Key nötig)         |
| LLM         | Mistral Ministral 3B via OpenRouter            |
| Ausgabe     | Slack (Channel-Nachricht, mrkdwn-Formatierung) |
