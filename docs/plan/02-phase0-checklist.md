# Phase 0 – Checkliste (5.–18.10.2026)

- [ ] Fork-CI: `build.yml` läuft im Fork (macOS-Runner, Xcode-Version festhalten, Cache für `engine/firefox` und sccache)
- [ ] Reproduzierbarer Gecko-Build: `update-gecko.sh` → `apply-patches.sh` → `build-gecko.sh`; Dauer, Plattenbedarf (>150 GB?), Fehlerbild dokumentieren
- [ ] Mac mit Xcode für lokale Builds benennen (M4/M7?) – CI-Minuten gegen eigenen Mac abwägen
- [ ] Sideload-IPA auf echtem Gerät (iPhone aktuell + ein älteres), JIT-Weg testen (AltStore/SideStore, Pairing)
- [ ] Baseline-Messung: Start, Speicher, Seiten-Kompatibilität, 3 Referenzseiten
- [ ] Inkpour, URL-Redirector als Gecko-Add-ons laden; Ergebnis in `03-addon-test-matrix.md`
- [ ] `AGENTS.md` abnehmen; Merge-Gate (Build + Lint + Grep-Check Engine-Grenze) einrichten
- [ ] Kosten pro Ticket mitschreiben (Tokens/Ticket), wöchentliches Limit beobachten
- [ ] Entscheidung Gate 1 am 19.10.2026
- [ ] Parallel: Prüfung Apple-BrowserEngineKit-Entitlement-Voraussetzungen (nur Recherche, kein Antrag)
