ENIAC Rabattstrategie – Analyse
Eine datenbasierte Untersuchung der Auswirkungen von Rabatten auf die E-Commerce-Performance von ENIAC.

Projektübersicht
Dieses Repository enthält die Analyse des zweiten Datenprojekts von ENIAC. Ziel ist es zu untersuchen, ob Rabatte das nachhaltige Unternehmenswachstum unterstützen oder sich negativ auf Umsatz und Margen auswirken.

Im Zentrum steht eine interne Diskussion zwischen dem Marketing-Team, das Rabatte als Mittel zur Kundengewinnung, -zufriedenheit und -bindung betrachtet, und den Investoren im Vorstand, die kritisch hinterfragen, ob zu hohe Rabatte die angestrebte Positionierung des Unternehmens im Qualitätssegment gefährden.

Geschäftsfragen
Wie viele Produkte werden rabattiert?
Wie hoch sind die Rabatte als Prozentsatz des Produktpreises?
Wie beeinflussen saisonale Faktoren und besondere Termine wie Weihnachten und Black Friday die Verkäufe?
Wie sollten Produkte klassifiziert werden, um Reporting und Analyse zu vereinfachen?
Wie verteilen sich die Produktpreise auf die verschiedenen Kategorien?
Wie könnten Datenerfassung und Datenqualität verbessert werden?
Datenquellen
Die Analyse verwendet vier CSV-Dateien:

Datei	Beschreibung
orders.csv	Eine Zeile pro Bestellung, einschließlich Bestelldatum, bezahltem Gesamtbetrag und Bestellstatus.
orderlines.csv	Eine Zeile pro Produktposition einer Bestellung, einschließlich SKU, Menge, Einzelpreis und Bearbeitungsdatum.
products.csv	Produktkatalog mit SKU, Name, Beschreibung, Basispreis, Aktionspreis, Lagerstatus und Produkttyp.
brands.csv	Zuordnung zwischen dreistelligen Markencodes und Markennamen.
Ergebnisse der explorativen Analyse
Die explorative Phase zeigte erhebliche Probleme bei Datenqualität und Konsistenz in den Quellen. Diese Erkenntnisse haben sowohl die Bereinigungsstrategie als auch die Interpretation der finalen Ergebnisse maßgeblich beeinflusst.

orders.csv
Identifizierte Probleme:

created_date war als Text statt als Datetime-Feld gespeichert.
Bei total_paid fehlten fünf Werte.
total_paid enthielt extreme Werte von bis zu rund 214.747 €.
Die Spalte state enthielt fünf verschiedene Bestellstatus. Nur "Completed" stellte tatsächlich bezahlte, abgeschlossene Verkäufe dar.
Behandlung:

created_date wurde korrekt in das Datetime-Format konvertiert.
Die fünf fehlenden total_paid-Werte wurden entfernt, da sie ausnahmslos zu Pending-Bestellungen gehörten und dort nur 0,035 % dieses Status ausmachten – kein typisches Muster, sondern eine Ausnahme.
Die extremen Werte wurden beibehalten und dokumentiert. Sie traten ausschließlich bei Shopping-Basket-Bestellungen auf und beeinflussten die Analyse des tatsächlichen Umsatzes daher nicht.
Die Umsatzanalyse wurde konsequent auf "Completed"-Bestellungen beschränkt.
orderlines.csv
Identifizierte Probleme:

unit_price war als Text gespeichert.
Rund 36.169 Werte in unit_price enthielten Formatierungsfehler, etwa doppelte Dezimalpunkte (z. B. "1.137.99").
date war als Text statt als Datetime-Feld gespeichert.
product_id war konstant 0 und dadurch unbrauchbar.
product_quantity enthielt extreme Werte von bis zu 999 Einheiten.
Ein negativer Einzelpreis (-119,00 €) wurde gefunden.
865 Zeilen hatten einen unit_price von 0 €, vermutlich kostenlose Artikel.
Differenzen zwischen der Summe der Bestellpositionen und total_paid entsprachen häufig festen Versandkosten, etwa 6,99 €, 4,99 € oder 3,99 €.
Behandlung:

Der Formatierungsfehler mit den doppelten Dezimalpunkten wurde korrigiert, wo der ursprüngliche Wert zuverlässig rekonstruierbar war.
Die Korrektur wurde auf drei unabhängigen Wegen validiert: Abgleich mit products.price, Prüfung der betroffenen Produktnamen und Vergleich mit den Bestellsummen. In 99,97 % der Fälle stimmten die rekonstruierten Werte mit orders.total_paid überein.
Drei nicht rekonstruierbare Preiswerte wurden auf NaN gesetzt. Keiner davon gehörte zu einer abgeschlossenen Bestellung.
Das Vorzeichen des negativen Preises wurde korrigiert, da der Absolutwert exakt dem tatsächlichen Produktpreis entsprach.
Werte von 0 € wurden nicht gelöscht, sondern über einen neuen Indikator is_free_item markiert. 49 dieser kostenlosen Artikel gehörten zu abgeschlossenen Bestellungen und blieben damit für die Preisanalyse relevant.
date wurde ins Datetime-Format konvertiert, die unbrauchbare Spalte product_id wurde entfernt.
Extreme Mengen wurden anhand der Bestellsummen validiert. Die größten Mengen waren mit realen Bestellungen vereinbar, etwa möglichen Großbestellungen, und wurden daher nicht automatisch entfernt.
Das wahrscheinliche Vorhandensein separater Versandkosten, die nicht in einem eigenen Feld erfasst werden, wurde dokumentiert.
products.csv
Identifizierte Probleme:

sku war nicht zuverlässig eindeutig: 8.746 vollständig doppelte Zeilen sowie ein weiterer doppelter SKU-Wert wurden gefunden.
Fehlende Werte traten bei desc (7), price (46) und type (50) auf.
price, promo_price und type waren als Text gespeichert.
542 Werte in price enthielten Formatierungsprobleme, unter anderem doppelte Dezimalpunkte und zu viele Nachkommastellen.
Mehr als 92 % der Werte in promo_price wiesen ähnliche Formatierungsfehler auf.
Behandlung:

Duplikate wurden zweistufig entfernt: zunächst vollständig doppelte Zeilen, danach der verbleibende doppelte SKU-Wert.
Fehlende Produktbeschreibungen wurden mit dem jeweiligen Produktnamen aufgefüllt.
type wurde bewusst nicht aufgefüllt, da die Bedeutung des numerischen Codes unklar blieb.
Zeilen mit fehlendem price wurden entfernt, wenn sich der Wert nicht zuverlässig aus orderlines rekonstruieren ließ.
Die 542 Produkte mit nicht reparierbarem Preisformat wurden entfernt, da der ursprüngliche Wert ohne Informationsverlust nicht wiederherstellbar war.
promo_price wurde komplett von der Rabattberechnung ausgeschlossen, da über 90 % der Spalte beschädigt und damit nicht vertrauenswürdig war.
Wichtige Konsequenz: Rabatte konnten nicht direkt über promo_price berechnet werden. Stattdessen wurden sie rekonstruiert, indem die Katalogpreise mit den tatsächlich in orderlines gezahlten Preisen verglichen wurden.

brands.csv
Identifizierte Probleme:

Bei sechs Markennamen erschienen jeweils zwei unterschiedliche Kurz-Codes: Apple, Bose, Jaybird, Mophie, Startech und Unknown.
Behandlung:

Es wurde keine automatische Korrektur vorgenommen, da die Datenquelle nicht genügend Information enthielt, um den jeweils korrekten Code eindeutig zu bestimmen.
Die Inkonsistenz wurde dokumentiert, der Kurz-Code blieb als Verknüpfungsschlüssel zwischen Marken und Produkten erhalten.
Ansonsten enthielt die Tabelle keine fehlenden Werte oder doppelten Zeilen und wies durchgehend passende Datentypen auf.
Strategie zur Datenqualität
Bei der Datenbereinigung wurde durchgehend der Erhalt realer Verkaufsinformationen einer aggressiven Entfernung ungewöhnlicher Beobachtungen vorgezogen.

Auffällige Werte wurden vor jeder Entfernung oder Änderung zunächst mit verwandten Tabellen, Bestellsummen, Produktnamen, Bestellstatus und der zugrunde liegenden Geschäftslogik abgeglichen.

Der rekonstruierte Rabattprozentsatz wurde wie folgt berechnet:

Rabatt % = (Katalogpreis − tatsächlich bezahlter Preis) / Katalogpreis × 100
Kategorisierungsstrategie
Die Produktkategorien wurden nicht direkt aus der Spalte type in products.csv übernommen, da es sich dabei um einen numerischen Code mit unklarer Bedeutung und 50 fehlenden Werten handelt.

Stattdessen wurden die Kategorien auf Analyseebene nach folgendem Ansatz gebildet:

Primäres Signal: Produktname – Die Kategorien wurden anhand von Textmustern in products.name definiert. Produkte, deren Name beispielsweise mit "iMac" beginnt, wurden der Kategorie iMac zugeordnet, entsprechend für iPhone, Tablet und weitere.

Zusätzliches Signal: Markencode – Die Tabelle brands.csv enthält dreistellige Markencodes, die am Anfang jeder products.sku stehen. Diese Codes wurden als sekundäres Signal genutzt.

Ziel: umsatzorientiert und für das Management verständlich – Die Kategorien machen den Umsatzanteil jeder Kategorie berechenbar, identifizieren die fünf umsatzstärksten Kategorien (zusammen rund 55 % des Gesamtumsatzes) und verwenden verständliche Bezeichnungen statt schwer interpretierbarer numerischer Codes.

Preisheterogenität innerhalb der Kategorien – Jede Kategorie umfasst eine große Preisspanne, vom Einstiegsmodell bis zur professionellen High-End-Variante desselben Produkttyps. Ein einheitlicher Rabattsatz für eine gesamte Kategorie wäre daher zu ungenau.

Hinweis zur Implementierung – Die Kategorisierung wurde bislang nur im Analysecode (Python) umgesetzt, nicht strukturiert in der Datenbank selbst.

Team und Zusammenarbeit
Dieses Projekt wurde von einem dreiköpfigen Team durchgeführt: Navid Modir Khazeni, Georgij Vinnichenko und Mehrnoosh Mohebi Damabi.

Alle drei Teammitglieder haben in allen Projektphasen zusammengearbeitet, unter anderem bei Datenexploration, Datenbereinigung, Analyse, Visualisierung und der Entwicklung von Empfehlungen.

Jedes Teammitglied erstellte zunächst unabhängig eine eigene Analyse. Anschließend wurden die Ergebnisse im Team verglichen, wobei jede und jeder den eigenen Ansatz vorstellte und begründete. Die finale Version wurde gemeinsam ausgewählt: Berücksichtigt wurde der Ansatz, den die Mehrheit als logisch und am besten begründet einstufte.

Wichtige Ergebnisse
Rabatte sind die Norm
91 % der verkauften Artikel wurden unterhalb des Katalogpreises verkauft.
Bei diesen rabattierten Verkäufen lag der durchschnittliche Rabatt bei rund 22,5 %.
Nur 9 % aller Verkäufe erfolgten zum vollen Katalogpreis.
Saisonalität beeinflusst den Umsatz stärker als Rabatte allein
Black Friday erzeugte an einem einzigen Tag einen extremen Umsatzpeak – rund das Zehnfache eines normalen Tagesumsatzes.
Die Zeit vor Weihnachten zeigte über mehrere Wochen hinweg einen langsameren, aber anhaltenderen Anstieg der Verkäufe.
Black Friday nimmt dabei wahrscheinlich einen Teil des klassischen Weihnachtsgeschäfts vorweg, da Kundinnen und Kunden ihre Käufe früher tätigen, um von den besten Rabatten des Jahres zu profitieren.
Wenige Premium-Kategorien tragen einen Großteil des Geschäfts
Die fünf umsatzstärksten Kategorien erzeugten zusammen rund 55 % des Gesamtumsatzes.
iMac und iPhone waren dabei die führenden Kategorien.
Innerhalb dieser Kategorien reichten die Preise von Einstiegsmodellen bis zu professionellen High-End-Varianten desselben Produkttyps.
Rabatt ≠ Umsatztreiber
Kategorie	Umsatzanteil	Durchschnittlicher Rabatt
iMac	Höchster	Niedrig
Tablet	Niedrig	Hoch
iMac erzielte den höchsten Umsatz bei einem vergleichsweise niedrigen Rabatt.
Tablets erhielten höhere Rabatte, trugen aber deutlich weniger zum Umsatz bei.
Ein hoher Rabatt führt also nicht automatisch zu hohem Umsatz.
Methodik und Tools
Python
pandas und NumPy für Datenaufbereitung und Analyse
Matplotlib und Seaborn für Visualisierungen
Jupyter Notebooks für die explorative Analyse
Git und GitHub für die Versionskontrolle
Repository-Struktur
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
Empfehlungen
1. Promotions auf wichtige Termine konzentrieren
Rabatte sollten gezielt auf Black Friday und die Wochen vor Weihnachten konzentriert werden, da dort der nachweislich größte Umsatz-Effekt zu beobachten war – statt sie dauerhaft und breit über das ganze Jahr zu verteilen.

2. Margen in Premium-Kategorien schützen
Aggressive Rabatte auf umsatzstarke Kategorien wie iMac und iPhone sollten vermieden werden. Diese Produkte verkaufen sich bereits bei niedrigeren Rabatten gut, wodurch sich die Marge besser schützen lässt.

3. Datenerfassung und Zuverlässigkeit der Datenpipeline verbessern
Versandkosten separat erfassen, statt sie im Gesamtbetrag zu verstecken.
Produktkategorien von Anfang an konsistent und korrekt speichern.
Die Datenpipeline zwischen Online-Shop und Datenbank verbessern, um künftige Datenbeschädigungen zu vermeiden.
4. Kostendaten für die Margenanalyse ergänzen
Einkaufs- oder Herstellungskosten sollten ergänzt werden, damit sich die Auswirkung auf die Marge pro Kategorie und pro Promotion direkt berechnen lässt.

5. Analyse über mehrere Jahreszyklen wiederholen
Dieselbe Analyse sollte nach dem nächsten Black Friday und der nächsten Weihnachtssaison wiederholt werden, um zu bestätigen, dass es sich bei den beobachteten Mustern um wiederkehrende Effekte handelt und nicht um einmalige Ereignisse.

Limitationen
Der Datensatz umfasst nur einen Jahreszyklus, einschließlich eines Black Fridays und einer Weihnachtsperiode.
Kosten- und Margendaten waren nicht verfügbar. Die Analyse misst daher den Umsatz, nicht den Gewinn.
Das beschädigte Feld promo_price erforderte eine indirekte Rekonstruktion der Rabatte.
Die Daten können nicht abschließend belegen, dass Rabatte die beobachteten saisonalen Verkaufsmuster tatsächlich verursacht haben – ein ursächlicher Zusammenhang ist plausibel, aber nicht zweifelsfrei nachgewiesen.
Einige Inkonsistenzen bei Produkten und Marken konnten dokumentiert, aber nicht zuverlässig korrigiert werden.
Lizenz
Dieses Projekt dient ausschließlich Bildungszwecken.

Autoren
ENIAC Data Analytics Team:  Georgij Vinnichenko, Navid Modir Khazeni, Mehrnoosh Mohebi Damabi

Zuletzt aktualisiert: September 2026
