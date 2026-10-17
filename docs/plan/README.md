# Planung: freier Orion-artiger iOS-Browser auf Reynard-Basis

Status: Entwurf (Planungs-Branch `plan/opus-roadmap`), Stand 2026-10-17. Kein Code geändert.
Detail-Budget/Kostenmodell steht im Claude-Docs-Dokument „Projektplan: Freier iOS-Browser mit Gecko“; hier nur, was ins Repo gehört.

## Ziel
Freier (GPL-3.0) Browser für iOS, Fokus EU (Japan optional), breite WebExtension-Unterstützung (MV2/MV3), Engine austauschbar: **Gecko** bevorzugt, **WebKit** als Fallback. Kein VPN, keine Zusatzdienste. Start: Sideloading.

## Ausgangslage im Repo (Ist-Stand, geprüft)
- `browser/Reynard` (~74k Zeilen Swift gesamt): App, Tabs, Stores, UI, JIT-Enabler (`JIT/`, Pairing via idevice).
- `browser/GeckoView`: Swift-Schicht über Gecko (Session, Runtime, Delegates, **Addons**, ContentBlocking). Das ist die natürliche Engine-Naht.
- `browser/Helper`: Helper-Prozess/ExtensionBridge.
- `engine/firefox` (Submodule, aktuell `FIREFOX_157_0_RELEASE`), `patches/` (370 Dateien, MPL-2.0; Schwerpunkt `widget/uikit`, `gfx`, `ipc`, `dom/ipc`, `js/src/jit`).
- Build: `tools/development/{update-gecko,apply-patches,build-idevice,build-gecko}.sh`; CI in `.github/workflows`.
- Entitlements: `Reynard.private.entitlements` nutzt private Rechte (`platform-application`, `no-sandbox`, `get-task-allow` …) → **nur Sideload/TrollStore, nie App Store, nie Notarisierung**. Normale Variante nur `increased-memory-limit`.
- Kopplung: App-Code (Tab, SessionManager) hängt direkt an `GeckoSession`/`GeckoRuntime`. Eine Engine-Abstraktion ist daher ein echtes Refactoring, kein Anbau.

## Leitentscheidungen
1. **Fork, kein Rewrite.** Upstream (minh-ton) liefert Gecko-Port, UI, Add-ons; wir ergänzen Engine-Abstraktion, WebKit-Adapter, Add-on-Testsuite.
2. **Engine-Naht** an der Grenze `GeckoView` ↔ `Client` (siehe `01-engine-abstraction.md`).
3. **Zwei Auslieferungspfade, getrennt gebaut:**
   - A) *Sideload-Gecko* (heutiger Reynard-Weg, private APIs). Sofort machbar.
   - B) *Konform*: WebKit + Apples `WKWebExtension` (iOS 18.4+) → App-Store-fähig; Gecko nur über BrowserEngineKit + Apple-Entitlement (EU), siehe Gate 2.
4. Lizenz: App GPL-3.0, Patches MPL-2.0 (bleibt). GPL-vs-App-Store ist offen und juristisch zu klären (keine Rechtsberatung).

## Phasen und Gates
| Phase | Zeitraum | Inhalt | Exit-Kriterium |
|---|---|---|---|
| 0 | 5.–18.10.2026 | Harness, Build reproduzieren, Reality-Check | Gate 1 |
| 1 | bis 15.01.2027 | MVP: Engine-Interface, WebKit-Adapter, Gecko-Adapter, 3 Test-Add-ons laufen | MVP |
| 2 | bis 28.03.2027 | Alpha: Add-on-Matrix, Content-Blocking, Sync-freie Basis-Features | Alpha |
| 3 | bis 15.07.2027 | Beta (Sideload), Entitlement-Antrag, Audit | Beta; Puffer bis 18.12.2027 |

**Gate 1 (19.10.2026):** Upstream-Build lokal/CI reproduzierbar, läuft auf mindestens 1 echtem Gerät mit JIT, Inkpour lädt als Gecko-Add-on. Sonst Kurswechsel auf WebKit-first.
**Gate 2 (Phase 3):** Apple-Entitlement realistisch? (EU-only, WPT/Test262-Schwellen, Sicherheitsprozess, Vertriebsweg). Fällt Gate 2, bleibt Pfad A + WebKit-Pfad B.

## Offene Fragen (bitte klären)
1. Outlook-Web/Teams→IMAP-Add-on: braucht rohe Sockets? Unter Gecko/iOS nicht vorhanden; vermutlich nur über native Bridge lösbar.
2. Wochenaufwand 15–20 h Annahme ok?
3. Zusammenarbeit/Upstream-PRs mit minh-ton gewünscht oder Hard-Fork?
4. Mehrere Max-Accounts: ToS-konform? (vor Nutzung klären)

## Dokumente
- `01-engine-abstraction.md` – Schnittstellenentwurf, Migrationsschritte
- `02-phase0-checklist.md` – konkrete Aufgaben der nächsten 14 Tage
- `03-addon-test-matrix.md` – Testfälle für eigene Add-ons
- `../../AGENTS.md` – Regeln für Coding-Agenten
