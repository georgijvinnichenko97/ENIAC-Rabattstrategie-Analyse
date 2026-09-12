# ENIAC Rabattstrategie – Analyse
Eine datenbasierte Untersuchung der Auswirkungen von Rabatten auf die E-Commerce-Performance von ENIAC.

 🎯Projektübersicht
Dieses Repository enthält die Analyse des zweiten Datenprojekts von ENIAC. Ziel ist es zu untersuchen, ob Rabatte das nachhaltige Unternehmenswachstum unterstützen oder sich negativ auf Umsatz und Margen auswirken. 

Im Zentrum steht eine interne Diskussion zwischen dem Marketing-Team, das Rabatte als Mittel zur Kundengewinnung, -zufriedenheit und -bindung betrachtet, und den Investoren im Vorstand, die kritisch hinterfragen, ob zu hohe Rabatte die angestrebte Positionierung des Unternehmens im Qualitätssegment gefährden.

❓ Geschäftsfragen
* Wie viele Produkte werden rabattiert?
* Wie hoch sind die Rabatte als Prozentsatz des Produktpreises?
* Wie beeinflussen saisonale Faktoren und besondere Termine wie Weihnachten und Black Friday die Verkäufe?
* Wie sollten Produkte klassifiziert werden, um Reporting und Analyse zu vereinfachen?
* Wie verteilen sich die Produktpreise auf die verschiedenen Kategorien?
* Wie könnten Datenerfassung und Datenqualität verbessert werden?

📊 Datenquellen
Die Analyse verwendet vier CSV-Dateien:

| Datei | Beschreibung |
| :--- | :--- |
| **`orders.csv`** | Eine Zeile pro Bestellung, einschließlich Bestelldatum, bezahltem Gesamtbetrag und Bestellstatus. |
| **`orderlines.csv`** | Eine Zeile pro Produktposition einer Bestellung, einschließlich SKU, Menge, Einzelpreis und Bearbeitungsdatum. |
| **`products.csv`** | Produktkatalog mit SKU, Name, Beschreibung, Basispreis, Aktionspreis, Lagerstatus und Produkttyp. |
| **`brands.csv`** | Zuordnung zwischen dreistelligen Markencodes und Markennamen. |

📂 Ergebnisse der Explorativen Analyse & Datenbereinigung
Die explorative Phase zeigte erhebliche Probleme bei Datenqualität und Konsistenz. Diese Erkenntnisse haben die Bereinigungsstrategie maßgeblich beeinflusst. Die oberste Regel lautete: **Erhalt realer Verkaufsinformationen** vor der aggressiven Entfernung ungewöhnlicher Beobachtungen.

### `orders.csv`
* **Probleme:** Datum als Text, 5 fehlende Werte bei `total_paid`, extreme Ausreißer (bis zu 214.747 €), verschiedene Status.
* **Behandlung:** Datum in Datetime konvertiert. Fehlende Beträge (alle "Pending") entfernt. Extreme Werte (Shopping-Baskets) belassen. Umsatzanalyse strikt auf **"Completed"** gefiltert.

### `orderlines.csv`
* **Probleme:** Preis und Datum als Text, doppelte Dezimalpunkte bei Preisen, nutzlose `product_id` (konstant 0), negative Einzelpreise, Gratisartikel (0 €), Diskrepanzen durch versteckte Versandkosten.
* **Behandlung:** Formatierungsfehler rekonstruiert und mit `orders.total_paid` validiert (99,97 % Übereinstimmung). 3 nicht-reparierbare Werte (nicht "Completed") auf NaN gesetzt. Negatives Vorzeichen korrigiert. 0 €-Werte mit neuem Indikator `is_free_item` versehen. `product_id` gelöscht.

### `products.csv`
* **Probleme:** 8.746 Duplikate, fehlende Werte, Datentyp-Fehler, massiv beschädigte `promo_price` Spalte (> 92 % fehlerhaft).
* **Behandlung:** Duplikate restlos entfernt. Fehlende Beschreibungen mit Produktnamen gefüllt. `promo_price` komplett von der Analyse ausgeschlossen. **Rabatte wurden rekonstruiert**, indem Katalogpreise mit real gezahlten Preisen aus `orderlines` verglichen wurden: `Rabatt % = (Katalogpreis − tatsächlich bezahlter Preis) / Katalogpreis × 100`.

### `brands.csv`
* **Probleme:** Sechs Marken besaßen zwei unterschiedliche Kurz-Codes.
* **Behandlung:** Inkonsistenz dokumentiert, Codes als Verknüpfungsschlüssel jedoch belassen, da eine eindeutige automatische Zuordnung nicht sicher möglich war.

📉 Kategorisierungsstrategie
Die ursprüngliche `type`-Spalte war numerisch und schwer interpretierbar. Eine neue, umsatzorientierte Kategorisierung wurde per Python-Skript erstellt:
* **Signale:** Textmuster im Produktnamen (z.B. "iMac") und die ersten drei Buchstaben der SKU (Markencode).
* **Ziel:** Die Identifikation der 5 umsatzstärksten Kategorien (ca. 55 % des Gesamtumsatzes).
* **Heterogenität:** Jede Kategorie umfasst Einstiegs- bis High-End-Modelle, was die Pauschalisierung von Rabatten auf Kategorie-Ebene verhindert.

❗ Wichtige Ergebnisse
* **Rabatte sind die Norm:** 91 % der Artikel wurden unter Katalogpreis verkauft (durchschnittlich 22,5 % Rabatt). Nur 9 % erzielten den vollen Preis.
* **Saisonalität schlägt Rabatthöhe:** Der Black Friday erzeugte einen extremen Peak (ca. das Zehnfache eines normalen Tages). Das Weihnachtsgeschäft stieg langsamer, aber anhaltender. Der Black Friday zieht vermutlich klassische Weihnachtskäufe vor.
* **Umsatzkonzentration:** iMac und iPhone dominieren den Umsatz.
* **Rabatt ≠ Umsatztreiber:** iMacs generieren höchsten Umsatz bei niedrigen Rabatten. Tablets haben hohe Rabatte, aber niedrigen Umsatzanteil.

🎖️ Empfehlungen
1. **Promotions fokussieren:** Rabattaktionen auf Black Friday und die Vorweihnachtszeit beschränken, anstatt sie ganzjährig zu streuen.
2. **Margen in Premium-Kategorien schützen:** Aggressive Rabatte auf Selbstläufer wie iMac und iPhone reduzieren, da diese sich auch bei niedrigeren Rabatten hervorragend verkaufen.
3. **Datenqualität verbessern:** Versandkosten separat erfassen, Produktkategorien konsistent in der Datenbank speichern und die Pipeline zwischen Shop und Datenbank reparieren.
4. **Kostendaten ergänzen:** Einkaufs-/Herstellungskosten integrieren, um zukünftig nicht nur Umsatz, sondern echte Margen berechnen zu können.
5. **Langzeitanalyse:** Die Analyse über mehrere Jahreszyklen wiederholen, um saisonale Effekte zu verifizieren.

🛠️ Methodik und Tools
* **Python:** pandas, NumPy
* **Visualisierung:** Matplotlib, Seaborn
* **Umgebung:** Jupyter Notebooks
* **Versionskontrolle:** Git & GitHub

📂 Repository-Struktur
```text
eniac-rabattstrategie/
├── README.md
├── requirements.txt
├── data/
│   ├── raw/
│   │   ├── orders.csv
│   │   ├── orderlines.csv
│   │   ├── products.csv
│   │   └── brands.csv
│   └── clean/
│       ├── orders_clean.csv
│       ├── orderlines_clean.csv
│       ├── products_clean.csv
│       ├── products_with_categories.csv
│       └── brands_clean.csv
├── notebooks/
│   ├── 01_data_cleaning.ipynb
│   ├── 02_quality_assessment.ipynb
│   ├── 03_categorization.ipynb
│   └── 04_visualization.ipynb
├── visuals/
│   ├── umsatz_zeitverlauf.png
│   ├── umsatz_kategorie.png
│   └── umsatz_vs_rabatt.png
└── presentation/
    └── ENIAC_Rabattstrategie_Vorstandspraesentation.pdf
```

👥 Team und Zusammenarbeit
**ENIAC Data Analytics Team:** Georgij Vinnichenko, Navid Modir Khazeni, Mehrnoosh Mohebi Damabi.

Das Projekt wurde von Beginn an im Team durchgeführt. Jedes Mitglied führte explorative Analysen zunächst eigenständig durch. In Teambesprechungen wurden die Lösungsansätze verglichen, diskutiert und die jeweils robusteste Methode für das finale Notebook ausgewählt.

⛓️ Limitationen
* **Datenmenge:** Der Datensatz umfasst nur einen Jahreszyklus.
* **Fehlende Margen:** Ohne Kostendaten misst die Analyse reinen Umsatz, nicht den Gewinn.
* **Kausalität:** Ein ursächlicher Zusammenhang zwischen Rabatten und Verkaufsspitzen ist plausibel, statistisch aber durch diesen Datensatz allein nicht zweifelsfrei kausal belegt.
* **Beschädigte Originaldaten:** Das Fehlen verwertbarer `promo_price`-Werte machte eine indirekte Rekonstruktion notwendig.

---
**Lizenz:** Dieses Projekt dient ausschließlich Bildungszwecken.
