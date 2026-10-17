# Engine-Abstraktion (Entwurf)

## Problem
`Tab`, `SessionManager`, Addon-UI und Stores nutzen Gecko-Typen direkt (`GeckoSession`, `GeckoRuntime`, `Addon*`, Delegates). Ein zweiter Engine-Adapter ist nur möglich, wenn der Client gegen Protokolle programmiert.

## Vorbild
Firefox iOS: `BrowserKit/Sources/WebEngine` (Engine, EngineSession, EngineView).

## Zielstruktur (neues SwiftPM-Paket `browser/EngineKit`, ohne Gecko/WebKit-Import)
- `Engine` – Lebenszyklus, Runtime-Settings, Session-Factory
- `EngineSession` – load/reload/stop, history, state save/restore, find, zoom, print, Reader
- `EngineView` – UIView-Host
- `EngineSessionDelegate` – Navigation, Progress, Content, Permission, Prompt, Scroll, Media (bündelt die 12 GeckoView-Delegates)
- `ExtensionRuntime` – install/uninstall/enable, Permissions, Messaging, Action (Popup/Badge), Events
- `ContentBlocker` – Regeln/Listen, ETP-Stufen
- `DataStore` – Cookies, Cache, Site Data löschen
- Capability-Flags (`supportsMV2`, `supportsWebRequestBlocking`, `supportsNativeMessaging` …), damit UI Unterschiede ehrlich zeigt.

## Adapter
- `GeckoEngine` = bestehendes `browser/GeckoView`, hinter die Protokolle gezogen (mechanisch, kein Verhaltenswechsel).
- `WebKitEngine` = `WKWebView` + `WKWebExtensionController` (iOS 18.4+). Bekannte Lücken: kein Gecko-`webRequest`-Blocking im MV2-Stil, Unterschiede bei Service-Worker-Lebenszyklus, Natives nur über `WKWebExtension` native messaging.

## Migrationsschritte (je ein PR, je mit grünem Build)
1. EngineKit anlegen, Protokolle + Datentypen (rein additiv).
2. Gecko-Typen in `Client/` zählen und per Protokoll ersetzen, Modul für Modul: Tab → SessionManager → Stores → Addons-UI.
3. Delegates bündeln, Adapter-Tests (Fake-Engine) schreiben.
4. WebKitEngine mit Minimalumfang: Laden, Tabs, Verlauf.
5. Extensions über WebKit; Capability-Flags in UI.
6. Engine pro Tab/pro Profil wählbar (Einstellungen), später pro Site.

## Regeln
- `Client/` darf weder `Gecko*` noch `WK*` importieren (CI-Check per grep).
- Engine-spezifische Patches bleiben unter `patches/` (MPL) und werden nicht in Adapter-Logik vermischt.
