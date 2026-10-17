# Add-on-Testmatrix

Spalten: Gecko (Sideload) | WebKit (WKWebExtension). Status: ? offen, ✓ ok, ✗ geht nicht, ~ teilweise.

| Add-on | Zweck | MV | Kritische APIs | Gecko | WebKit |
|---|---|---|---|---|---|
| Inkpour (v3) | KI-Chat-Mitschnitt | 3 | content scripts, storage, downloads? | ? | ? |
| URL-Redirector | Umleitungen | 3 | declarativeNetRequest / webRequest | ? | ? |
| Outlook-Web → IMAP | Mail-Brücke | 3 | Sockets/Native Messaging nötig? | ? | ? |
| Teams → IMAP | Mail-Brücke | 3 | s.o. | ? | ? |
| IRC-artige Clients | Chat | 3 | WebSocket, Background-Lebenszyklus | ? | ? |
| Referenz: uBlock Origin Lite | Blocker | 3 | DNR | ? | ? |

Pro Zeile: Installation, Berechtigungsdialog, Popup, Background nach App-Suspend, Persistenz nach Neustart.
