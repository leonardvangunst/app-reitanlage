# Stallverwaltung – Preislogik & Tarifversionierung

Dieses Dokument beschreibt die **zentrale Preisverwaltung** für Boxen/Tarife inklusive **jährlicher Preisanpassung**, **Gültigkeitszeiträumen** und **Abrechnungsregeln**.

Ziel:

* zentrale Pflege der Boxenpreise
* jährliche Anpassung (Inflationsindex)
* saubere, revisionssichere Monatsabrechnungen

---

## 1. Grundprinzip

* **BoxType** = Tarif / Produktname (z. B. Paddockbox, Offenstall Wald)
* **Preis** = versionierter Datensatz mit Gültigkeitszeitraum
* **Contract** referenziert nur den BoxType, **nie einen festen Preis**
* **MonthlyStatement** zieht den **für den Monat gültigen Preis** und speichert einen Snapshot

➡️ Preise können geändert werden, ohne bestehende Statements zu beeinflussen.

---

## 2. Entitäten

### 2.1 BoxType (Tarif)

**CPT:** `box_type`

Beispiele:

* Paddockbox
* Innen-/Außenbox
* Offenstall Halle
* Offenstall am Wald
* Offenstall Heinsweg

BoxType enthält **keine Preise**, nur:

* Namen
* Beschreibung / Leistungsumfang

---

### 2.2 BoxPrice (Preisversion)

**CPT:** `box_price`

Eine Preisversion definiert **einen gültigen Preis für einen Zeitraum**.

#### Metafelder

| Meta-Key           | Typ       | Beschreibung                       |
| ------------------ | --------- | ---------------------------------- |
| _price_box_type_id | int       | Referenz auf BoxType               |
| _price_valid_from  | date      | Startdatum (immer 1. eines Monats) |
| _price_valid_to    | date/null | Enddatum (null = aktuell gültig)   |
| _price_mode        | enum      | fixed / seasonal                   |
| _price_fixed       | decimal   | fixer Monatspreis                  |
| _price_summer      | decimal   | Sommerpreis (bei seasonal)         |
| _price_winter      | decimal   | Winterpreis (bei seasonal)         |
| _arena_optional    | bool      | Reithalle optional                 |
| _arena_price       | decimal   | Preis Reithalle (z. B. 30.00)      |
| _created_by        | int       | Admin/Staff                        |
| _created_at        | datetime  | Audit                              |

#### Regeln

* **Keine überlappenden Zeiträume** pro BoxType erlaubt
* Pro BoxType existiert **immer genau eine aktive Preisversion** (valid_to = null)

---

## 3. Saisonlogik

Bei `price_mode = seasonal`:

* **Winter:** Oktober – März
* **Sommer:** April – September

Der für einen Monat relevante Preis wird anhand des Monats bestimmt.

---

## 4. Jahresanpassung (Februar-Prozess)

### Ziel

Einheitliche Preisanpassung **ab 01.02. eines Jahres** auf alle Verträge.

### Ablauf (Admin-only)

1. Admin öffnet Backend → Preisverwaltung
2. Auswahl eines BoxType
3. Klick auf **„Neue Preisperiode anlegen“**
4. System schlägt vor:

   * valid_from = `01.02.YYYY`
   * Preise = Kopie der aktuellen Periode
5. Admin passt Preise an (oder gibt Inflationsindex in % an)
6. Speichern

### Systemaktion beim Speichern

* bisherige Preisversion:

  * `_price_valid_to = 31.01.YYYY`
* neue Preisversion:

  * `_price_valid_from = 01.02.YYYY`
  * `_price_valid_to = null`

➡️ Alle bestehenden Contracts übernehmen **automatisch** den neuen Preis ab Februar.

---

## 5. Rundungsregeln

Standard-Rundung (empfohlen):

* **auf 0,50 €** runden

Alternative (konfigurierbar):

* keine Rundung
* auf 1,00 € runden

Rundung erfolgt **bei Anlage der neuen Preisversion**, nicht zur Laufzeit.

---

## 6. Abrechnung (MonthlyStatement)

### Preisermittlung für Monat `YYYY-MM`

1. Ermittle alle aktiven Contracts im Monat
2. Ermittle passende Preisversion:

   * `_price_valid_from <= letzter Tag des Monats`
   * `_price_valid_to` ist `null` oder `>= erster Tag des Monats`
3. Berechne Grundpreis:

   * fixed → `_price_fixed`
   * seasonal → Sommer/Winterpreis
4. Addiere optionale Bestandteile:

   * Reithalle (wenn im Contract aktiviert)
   * bestätigte Extras vom Pferd (Extra-Heu, Anhänger)
5. Speichere Snapshot in `monthly_statement`

➡️ Statements sind **immutable** (nach Lock keine Änderung mehr).

---

## 7. Sonderfälle

### 7.1 Sonderpreise / Bestandskunden

Optional pro Contract:

* `_contract_price_override`
* `_contract_override_reason`
* `_contract_override_valid_from/to`

Regel:

1. Wenn Override aktiv → Override-Preis
2. sonst → BoxPrice

---

## 8. Best Practices

* Preiswechsel **nur zum 1. eines Monats** erlauben
* niemals Preise live aus BoxType rechnen
* Monatsabrechnungen immer als Snapshot speichern
* Preisänderungen immer dokumentieren (Audit)

---

## 9. Zusammenfassung

* BoxType = Tarif
* BoxPrice = versionierter Preis mit Zeitraum
* Contract = Auswahl des Tarifs
* Statement = Snapshot

➡️ Skalierbar, revisionssicher, stalltauglich.

---

**Status:** freigegebenes Konzeptdokument
