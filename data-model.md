# Stallverwaltung – Datenmodell & Metafelder

Dieses Dokument definiert **alle relevanten Metafelder**, deren **Beziehungen**, **Abrechnungsrelevanz** sowie **Bearbeitungsrechte** für die WordPress‑basierte Stallverwaltung.

Ausgelegt für:

* > 70 Einsteller
* viele Reitbeteiligungen / Angehörige
* saubere Abrechnung & Skalierbarkeit

---

## 1. Entitäten & Beziehungen (Überblick)

**User (Person)**

* kann **Einsteller**, **Light‑User (Reitbeteiligung)**, **Staff** oder **Admin** sein
* 1 User (Einsteller) → *n* Pferde
* 1 User → *n* Monats‑Statements

**Horse (Pferd)**

* gehört genau **1 Einsteller (Owner)**
* kann **n zusätzliche berechtigte Nutzer** haben (Reitbeteiligungen)
* hat genau **1 aktiven Contract** (1 Pferd = 1 Box)

**Contract (Vertrag / Boxzuordnung)**

* verknüpft Owner + Horse + BoxType
* definiert abrechnungsrelevante Grundparameter

**BoxType (Tarif / Boxentyp)**

* zentrale Preisdefinition (fix oder saisonal)

**Herd (Taxonomy)**

* ordnet Pferde operativ einer Herde / Gruppe zu

**HorseAccess (Zugriffszuordnung)**

* definiert **welcher User welches Pferd mit welchen Rechten** nutzen darf

**ServiceRequest (abrechnungsrelevante Änderung)**

* vom Owner ausgelöst
* von Staff bestätigt / abgelehnt

**MonthlyStatement (Monatsabrechnung / Snapshot)**

* unveränderlicher Snapshot pro Monat

---

## 2. User (Personen & Rollen)

### Rollen

* `boarder` → Einsteller (Vertragspartner)
* `boarder_light` → Reitbeteiligung / Angehöriger
* `stable_staff` → Stallpersonal
* `administrator`

### User‑Meta (Payment / Stammdaten)

| Meta‑Key            | Typ    | Abrechnung | Bearbeitbar durch     | Beschreibung              |
| ------------------- | ------ | ---------: | --------------------- | ------------------------- |
| user_email          | string |          ✅ | Owner                 | Rechnungs‑ & Loginadresse |
| first_name          | string |          ✅ | Owner                 | Rechnungsempfänger        |
| last_name           | string |          ✅ | Owner                 | Rechnungsempfänger        |
| billing_street      | string |          ✅ | Owner                 | Rechnungsanschrift        |
| billing_zip         | string |          ✅ | Owner                 | Rechnungsanschrift        |
| billing_city        | string |          ✅ | Owner                 | Rechnungsanschrift        |
| phone               | string |          ❌ | Owner                 | Kontakt                   |
| emergency_contact   | string |          ❌ | Owner                 | optional                  |
| sepa_mandate_active | bool   |          ✅ | Owner / Staff / Admin | SEPA‑Mandat aktiv         |
| sepa_iban           | string |          ✅ | Owner / Staff / Admin | nur bei aktivem Mandat    |
| sepa_mandate_ref    | string |          ✅ | Staff / Admin         | Referenz                  |
| sepa_signed_at      | date   |          ✅ | Staff / Admin         | Datum Mandat              |

---

## 3. Horse (Pferd)

**CPT:** `horse`

> **Hinweis:** Pferdename = `post_title`

| Meta‑Key             | Typ    | Abrechnung | Bearbeitbar durch   | Beschreibung                      |
| -------------------- | ------ | ---------: | ------------------- | --------------------------------- |
| _horse_owner_user_id | int    |          ✅ | Staff/Admin         | Einsteller                        |
| _box_label           | string |          ❌ | Staff/Admin         | Box‑Nr / Bezeichnung              |
| _feed_am             | string |          ❌ | Owner / Staff       | Futter morgens (direkt)           |
| _feed_pm             | string |          ❌ | Owner / Staff       | Futter abends (direkt)            |
| _horse_status        | enum   |          ❌ | Owner / Staff       | normal / raus / boxenruhe / krank |
| _worming_self        | bool   |          ❌ | Owner / Staff       | Wurmkur selbst                    |
| _worming_last_date   | date   |          ❌ | Owner / Staff       | optional                          |
| _extra_hay_active    | bool   |          ✅ | **nur Staff/Admin** | durch Request                     |
| _trailer_spot_active | bool   |          ✅ | **nur Staff/Admin** | durch Request                     |
| _notes_internal      | text   |          ❌ | Staff/Admin         | intern                            |

---

## 4. Herd (Herde / Gruppe)

**Taxonomy:** `herd` (für CPT `horse`)

Beispiele:

* `stutenweide`
* `wallache_klein`
* `wallache_gross`
* `offenstall_wald`
* `offenstall_heinsweg`
* `offenstall_halle`

| Eigenschaft | Wert                            |
| ----------- | ------------------------------- |
| Abrechnung  | ❌                               |
| Bearbeitung | Staff / Admin                   |
| Nutzung     | Filter, Dashboard, Futterlisten |

---

## 5. HorseAccess (Zugriffszuordnung – **skalierbar**)

**CPT:** `horse_access`

> Saubere Lösung für viele Reitbeteiligungen (keine Arrays am Pferd).

| Meta‑Key                | Typ  | Abrechnung | Bearbeitbar durch | Beschreibung   |
| ----------------------- | ---- | ---------: | ----------------- | -------------- |
| _access_user_id         | int  |          ❌ | Staff/Admin       | Light‑User     |
| _access_horse_id        | int  |          ❌ | Staff/Admin       | Pferd          |
| _access_can_view        | bool |          ❌ | Staff/Admin       | sehen          |
| _access_can_edit_feed   | bool |          ❌ | Staff/Admin       | Futter ändern  |
| _access_can_edit_status | bool |          ❌ | Staff/Admin       | raus/boxenruhe |
| _access_can_book_arena  | bool |          ❌ | Staff/Admin       | Hallenbuchung  |

---

## 6. Contract (Vertrag / Box)

**CPT:** `contract`

| Meta‑Key                | Typ       | Abrechnung | Bearbeitbar durch | Beschreibung  |
| ----------------------- | --------- | ---------: | ----------------- | ------------- |
| _contract_owner_user_id | int       |          ✅ | Staff/Admin       | Einsteller    |
| _contract_horse_id      | int       |          ✅ | Staff/Admin       | Pferd         |
| _contract_box_type_id   | int       |          ✅ | Staff/Admin       | Tarif         |
| _contract_arena_enabled | bool      |          ✅ | Staff/Admin       | Reithalle +30 |
| _contract_start_date    | date      |          ✅ | Staff/Admin       | Start         |
| _contract_end_date      | date/null |          ✅ | Staff/Admin       | Ende          |

---

## 7. BoxType (Tarif / Boxentyp)

**CPT:** `box_type`

| Meta‑Key            | Typ       | Abrechnung | Bearbeitbar durch | Beschreibung     |
| ------------------- | --------- | ---------: | ----------------- | ---------------- |
| _box_price_mode     | enum      |          ✅ | Staff/Admin       | fixed / seasonal |
| _box_price_fixed    | decimal   |          ✅ | Staff/Admin       | Monatspreis      |
| _box_price_summer   | decimal   |          ✅ | Staff/Admin       | Sommer           |
| _box_price_winter   | decimal   |          ✅ | Staff/Admin       | Winter           |
| _box_arena_optional | bool      |          ✅ | Staff/Admin       | optional         |
| _box_arena_price    | decimal   |          ✅ | Staff/Admin       | z.B. 30.00       |
| _box_includes       | text/html |          ❌ | Staff/Admin       | Beschreibung     |

Saison:

* Winter: Okt – Mär
* Sommer: Apr – Sep

---

## 8. ServiceRequest (abrechnungsrelevante Änderungen)

**CPT:** `service_request`

| Meta‑Key           | Typ        | Abrechnung | Bearbeitbar durch | Beschreibung                  |
| ------------------ | ---------- | ---------: | ----------------- | ----------------------------- |
| _req_owner_user_id | int        |          ✅ | System            | Einsteller                    |
| _req_horse_id      | int        |          ✅ | System            | Pferd                         |
| _req_type          | enum       |          ✅ | System            | extra_hay / trailer_spot      |
| _req_value         | string/int |          ✅ | Owner             | Menge / on/off                |
| _req_status        | enum       |          ✅ | Staff/Admin       | pending / approved / rejected |
| _req_note          | text       |          ❌ | Owner             | optional                      |
| _req_decided_by    | int        |          ✅ | Staff/Admin       | Bearbeiter                    |
| _req_decided_at    | datetime   |          ✅ | Staff/Admin       | Zeitpunkt                     |

---

## 9. MonthlyStatement (Snapshot)

**CPT:** `monthly_statement`

| Meta‑Key            | Typ      | Abrechnung | Bearbeitbar durch | Beschreibung |
| ------------------- | -------- | ---------: | ----------------- | ------------ |
| _stmt_owner_user_id | int      |          ✅ | System/Staff      | Einsteller   |
| _stmt_month         | string   |          ✅ | System/Staff      | YYYY‑MM      |
| _stmt_rows          | json     |          ✅ | System/Staff      | Positionen   |
| _stmt_total         | decimal  |          ✅ | System/Staff      | Summe        |
| _stmt_created_at    | datetime |          ✅ | System            | erstellt     |
| _stmt_locked        | bool     |          ✅ | Staff/Admin       | gesperrt     |

Owner: **read‑only**.

---

## 10. Rechte‑Matrix (final)

| Aktion            | Owner |          Light | Staff | Admin |
| ----------------- | ----: | -------------: | ----: | ----: |
| Profil / Adresse  |     ✅ |              ❌ |     ✅ |     ✅ |
| Pferd sehen       |     ✅ | ✅ (zugewiesen) |     ✅ |     ✅ |
| Pferdename ändern |     ✅ |              ❌ |     ✅ |     ✅ |
| Futter ändern     |     ✅ |             ⚠️ |     ✅ |     ✅ |
| Status ändern     |     ✅ |             ⚠️ |     ✅ |     ✅ |
| Hallenbuchung     |     ✅ |             ⚠️ |     ✅ |     ✅ |
| ServiceRequest    |     ✅ |              ❌ |     ✅ |     ✅ |
| Abrechnung sehen  |     ✅ |              ❌ |     ✅ |     ✅ |

⚠️ = nur wenn über HorseAccess erlaubt

---

## 11. Konventionen

* CPTs: `horse`, `contract`, `box_type`, `herd`, `horse_access`, `service_request`, `monthly_statement`
* Meta‑Keys: `_snake_case`
* Geld: decimal mit Punkt
* Abrechnung **nur Snapshot**, niemals live

---

**Status:** Freigegebenes Arbeitsdokument
