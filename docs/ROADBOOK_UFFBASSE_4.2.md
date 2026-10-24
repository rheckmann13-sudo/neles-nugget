# Uffbasse – technisches Roadbook, stabiler Stand 4.2

**Stand:** 24.10.2026  
**Produkt:** Uffbasse – Unser Nugget (iPhone-fähige Web-App)  
**Repository:** https://github.com/rheckmann13-sudo/neles-nugget  
**Stabiler Sicherungszweig:** [stable/uffbasse-4.2](https://github.com/rheckmann13-sudo/neles-nugget/tree/stable/uffbasse-4.2)  
**Sicherungspunkt:** Commit `a2d715b3409a03cb4f0ecba24f509d39e500daf4`  
**Produktiv-URL:** https://rheckmann13-sudo.github.io/neles-nugget/

## 1. Zweck und Umfang
Die Funktion **Unterwegs → Wo ist was?** verwaltet das gemeinsame Nugget-Inventar mit Gegenstand, Menge, Einheit, Aufbewahrungsort, genauer Position und Bemerkung. Bedienprinzip: Suchen im Vordergrund, Erfassung über „+ Gegenstand hinzufügen“, maximal wenige Schritte bis zum Ziel.

## 2. Technische Architektur

```text
iPhone / Browser / Home-Screen-Web-App
             |
             | GitHub Pages (statische App-Dateien)
             v
          index.html
             |
             | Supabase JS / Auth-Session
             v
Supabase-Projekt „Nugget Uffbasse“
             |
             +-- public.inventory_items (Inventar)
             +-- Zugriffsschutz über Row Level Security
             |
             +-- public.adventures (Abenteuer, separat)
             
Druckvorschau -> lokale PDF-Erzeugung im Browser
              -> Datei speichern/teilen (ohne externes PDF-CDN)
```

## 3. Inventar – Bedienabläufe
**Lesen:** Cloud-Anmeldung → Abfrage `inventory_items` → Liste sortiert anzeigen → Suche lokal filtern.  
**Anlegen:** „+ Gegenstand hinzufügen“ → Daten eingeben → Speichern → Supabase INSERT → betroffene Zeile bestätigen → Liste neu laden.  
**Bearbeiten:** „Bearbeiten“ → Eingabemaske mit bestehenden Werten → UPDATE über Datensatz-ID → Bestätigung → Neuladen.  
**Löschen:** „Löschen“ → Sicherheitsabfrage → DELETE über Datensatz-ID → Bestätigung → Neuladen.  
**Drucken:** „Inventarliste drucken / PDF“ → frische Cloud-Abfrage → nach Aufbewahrungsort gruppierte Vorschau → „PDF speichern“ → lokale A4-PDF-Datei.

## 4. Datenmodell
Die App verwendet in `public.inventory_items` insbesondere:
- `id` – eindeutige Datensatz-ID
- `name` – Gegenstand
- `quantity` – Menge
- `unit` – Einheit
- `location` – Aufbewahrungsort
- `position` – genaue Position
- `note` – Bemerkung
- `created_by` – Benutzer-ID des Erstellers
- `updated_at` – Änderungszeitpunkt

Authentifizierte Zugriffe werden über die Supabase-Session und die bestehenden RLS-Regeln gesteuert. **Die Sicherung des GitHub-Codes ist kein Backup der Cloud-Datenbank.**

## 5. Behobene Probleme und Erkenntnisse
| Phase | Problem | Maßnahme |
|---|---|---|
| 3.0–3.4 | Änderungen in Cloud gespeichert, Anzeige veraltet | Datenabfragen und Aktualisierung überprüft |
| 3.7 | Service Worker zwischenspeicherte Supabase-API-Antworten | Cross-Origin-API-Abfragen vom Offline-Cache ausgenommen |
| 3.5 | Eingabeformular nahm zu viel Platz ein | Formular nur bei Hinzufügen/Bearbeiten öffnen |
| 3.8–4.0 | iOS-Druckdialog reagierte nicht; Tasten zunächst ohne Handler | Sichtbare Vorschau und spätere Handler-Bindung |
| 4.1 | Externe PDF-Bibliothek wurde nicht geladen | Abhängigkeit beseitigt |
| **4.2** | **Direkter PDF-Export** | **PDF im Browser ohne externe Bibliothek erzeugt; auf iPhone erfolgreich getestet** |

## 6. Qualitätssicherung – nachgewiesene Tests
- Cloud-Liste zeigte drei Gegenstände nach Korrektur des Service Workers.
- Ein doppelter Messer-Eintrag konnte gelöscht werden; anschließend zwei Einträge korrekt angezeigt.
- Menge einer Gabel von vier auf fünf geändert und anschließend korrekt aus Cloud geladen.
- Druckvorschau nach Aufbewahrungsort gegliedert angezeigt.
- PDF-Speicherung in Version 4.2 auf iPhone vom Nutzer erfolgreich bestätigt.

**Hinweis:** Diese Tests sind manuelle Funktionsprüfungen, keine automatisierte Testabdeckung. Ein vollständiger Cross-Browser-Test steht aus.

## 7. Versionssicherung und Wiederherstellung
Der Zweig `stable/uffbasse-4.2` zeigt auf den bestätigten Programmstand. Weitere Entwicklung erfolgt auf dem normalen Entwicklungszweig. Im Fehlerfall den stabilen Zweig als Vergleich verwenden oder gezielt den geprüften Commit wiederherstellen; **nicht** blind Datenbanktabellen überschreiben.

## 8. Betrieb und Wartung
- Nach Änderungen Versionsanzeige und App-Cache-Version gemeinsam erhöhen.
- Service Worker darf keine Supabase-API-Antworten offline zwischenspeichern.
- Nach Speichern/Löschen Datenbankantwort prüfen und Liste neu laden.
- Cloud-Daten regelmäßig separat exportieren bzw. sichern.
- PDF-Export mit langen Gegenstandsbezeichnungen, vielen Einträgen und mehrseitigen Listen zusätzlich testen.
- Zugriffskontrolle für alle berechtigten Nugget-Nutzer getrennt prüfen.

## 9. Nächste sinnvolle Schritte
1. Ein separates Backup-/Exportkonzept für die Supabase-Inventardaten definieren.
2. Mehrseitigen PDF-Ausdruck und Sonderzeichen systematisch testen.
3. Den stabilen Zweig nach Bedarf durch ein formelles GitHub-Release ergänzen.
