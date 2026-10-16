# King County Housing EDA – Kunde: Jacob Phillips

Exploratory Data Analysis der Hausverkäufe in King County (Seattle und Umgebung, USA). Das Projekt beantwortet die Frage, welche Immobilien die Wünsche eines Käufers mit unbegrenztem Budget (des fiktiven Kunden **Jacob Phillips**) erfüllen und in welchen Regionen sich diese Objekte am häufigsten finden.

## Auf einen Blick

| | |
| --- | --- |
| **Kunde** | Jacob Phillips (Käufer): unbegrenztes Budget, 4+ Badezimmer oder ein kleineres Haus in der Nähe, großes Grundstück (Tennisplatz & Pool), Golf, historisch, keine Wasserlage |
| **Daten** | King County House Sales, rund 21.600 Verkäufe von Mai 2014 bis Mai 2015 in 70 Postleitzahlgebieten |
| **Quelle** | PostgreSQL-Datenbank, Schema `eda`, zwei Tabellen (`king_county_house_sales`, `king_county_house_details`), per JOIN zusammengeführt |
| **Hypothesen** | H1: Große Grundstücke liegen eher außerhalb der Stadtzentren. H2: Historische Häuser haben im Schnitt weniger Badezimmer als neuere Häuser. H3: Je größer das Grundstück, desto höher der Preis, aber der Zusammenhang ist schwächer als bei der Wohnfläche. |

## Ergebnisse in Kürze

**Insights**

1. **Geografisch:** Große Grundstücke gibt es fast nur außerhalb von Seattle. Im 10-km-Radius um Downtown liegt ihr Anteil unter 0,1 %, ab 20 km über 13 %. Schwerpunkte sind der Osten (Redmond, Woodinville, Issaquah) und Vashon Island.
2. **Historische Häuser haben weniger Bäder:** Median 1,5 gegenüber 2,5 bei neueren Häusern. Renovierte historische Häuser haben deutlich mehr Bäder.
3. **Grundstück ist billig, Wohnfläche ist teuer:** Der Preis hängt vor allem an der Wohnfläche (Spearman ρ ≈ 0,64 gegenüber ≈ 0,08 beim Grundstück). Ein sqft Land kostet bei großen Grundstücken nur etwa ein Neuntel.

**Empfehlungen** für Jacob Phillips stehen in Abschnitt 11 des Notebooks [04_eda.ipynb](04_eda.ipynb). Wichtigste Erkenntnis: Die Schnittmenge aus historisch und großem Grundstück ist mit rund 120 Häusern klein, und nur 3 davon haben bereits 4+ Bäder. Die Badezimmer-Anforderung ist der Engpass, nicht das Grundstück.

## Repository-Struktur

Die Dateien sind in der Reihenfolge nummeriert, in der man sie durcharbeitet.

| Datei / Ordner | Inhalt |
| --- | --- |
| [01_assignment.md](01_assignment.md) | Projektbeschreibung: Daten, Aufgaben, Deliverables und Kundenliste |
| [02_workflow.md](02_workflow.md) | Empfohlener EDA-Workflow von der Datenprüfung bis zur Präsentation |
| [03_fetching_the_data_eda.ipynb](03_fetching_the_data_eda.ipynb) | Verbindung zur PostgreSQL-Datenbank (psycopg2, SQLAlchemy) und Laden der Daten in einen DataFrame |
| [04_eda.ipynb](04_eda.ipynb) | **Haupt-Notebook:** dokumentierte Analyse mit Hypothesen, Bereinigung, Visualisierungen, Tests, Insights und Empfehlungen |
| [column_names.md](column_names.md) | Datenwörterbuch der Spalten |
| [data/](data/) | Ablageort für die CSV-Datei (`EDA_data.csv`). Der Ordner ist getrackt, die Daten selbst sind per `.gitignore` ausgeschlossen. |
| [.env.example](.env.example) | Vorlage für die Datenbank-Zugangsdaten |
| [pyproject.toml](pyproject.toml), [uv.lock](uv.lock) | Projektkonfiguration und gesperrte Abhängigkeiten |

## Methodik

Das Notebook [04_eda.ipynb](04_eda.ipynb) ist wie folgt aufgebaut:

1. **Setup und Daten laden:** CSV aus dem JOIN der beiden Tabellen. Die Spalten `big_lot`, `historic`, `renovated` und `bath_4plus` werden bereits in der SQL-Abfrage berechnet.
2. **Daten verstehen:** deskriptive Statistik, fehlende Werte, kontinuierliche und kategoriale Features.
3. **Daten explorieren:** Verteilungen, Schiefe, Seltenheit der kundenrelevanten Merkmale.
4. **Daten bereinigen:** `yr_renovated` korrigiert (Faktor 10), Duplikate entfernt (jeweils der letzte Verkauf pro Haus), Ausreißer bewusst behalten, neues Feature „Distanz zu Downtown Seattle“ (Haversine). Insgesamt gingen nur rund 0,8 % der Zeilen verloren.
5. **Hypothesen H1–H3:** Karten, Verteilungen, Korrelationsmatrix, Spearman-Korrelation und Mann-Whitney-U-Test.
6. **Insights und Empfehlungen** für den Kunden.

### Annahmen

- **Großes Grundstück:** `sqft_lot` ≥ 40.000 sqft (≈ 3.700 m²)
- **Historisch:** gebaut vor 1950
- **Renoviert:** `yr_renovated` > 0
- **Stadtzentrum:** Downtown Seattle (47.6062, -122.3321)
- **Budget:** unbegrenzt, daher werden teure Häuser nicht als Ausreißer entfernt

## Setup

> [!NOTE]
> Text in spitzen Klammern wie `<repo-name>` ist ein Platzhalter und wird samt Klammern durch den eigenen Wert ersetzt.

1. **Repository klonen**

   ```bash
   git clone git@github.com:<your-username>/<repo-name>.git
   cd <repo-name>
   ```

2. **Abhängigkeiten installieren** (erstellt die virtuelle Umgebung in `.venv/`, benötigt Python ≥ 3.13)

   ```bash
   uv sync
   ```

   Zusätzliche Pakete lassen sich mit `uv add <package-name>` ergänzen (das Notebook nutzt z. B. `scipy`). Danach `pyproject.toml` und `uv.lock` committen.

3. **Datenbank-Zugangsdaten eintragen**

   ```bash
   cp .env.example .env
   ```

   In `.env` die Zugangsdaten der King-County-Datenbank eintragen (dieselben wie in DBeaver).

   > [!CAUTION]
   > `.env` enthält Geheimnisse und darf nie committet werden. Sie steht bereits in der `.gitignore`.

4. **Daten laden:** Zuerst in DBeaver das Schema `eda` ansehen und den JOIN der beiden Tabellen entwickeln. Danach die Abfrage in [03_fetching_the_data_eda.ipynb](03_fetching_the_data_eda.ipynb) verwenden und die CSV als `data/EDA_data.csv` speichern.

5. **Notebooks öffnen:** VS Code im Projektordner starten (`code .`) und die mit `uv sync` erzeugte Umgebung als Kernel wählen. [04_eda.ipynb](04_eda.ipynb) liest `data/EDA_data.csv`.

## Präsentation

Die Präsentation für nicht-technische Kunden dauert 10 Minuten und umfasst 10 Folien.

- [Praesentation_Jacob_Phillips.pdf](Praesentation_Jacob_Phillips.pdf): PDF-Export der Folien
- [Praesentation_Jacob_Phillips.pptx](Praesentation_Jacob_Phillips.pptx): PowerPoint-Datei mit Sprechernotizen
- [Praesentation_Leitfaden.md](Praesentation_Leitfaden.md): Stichpunkt-Leitfaden mit Zeitplan und möglichen Fragen

## Referenzen

- [House Sales in King County (Kaggle)](https://www.kaggle.com/datasets/harlfoxem/housesalesprediction): Quelldatensatz
- [Pandas User Guide](https://pandas.pydata.org/docs/user_guide/index.html)
- [Seaborn Tutorial](https://seaborn.pydata.org/tutorial.html)
- [SQLAlchemy Dokumentation](https://docs.sqlalchemy.org/en/20/)
- [EDA Checklist](https://github.com/neuefische/datascience-infographics/blob/main/EDA_Checklist.md)
- [Detailed EDA with Python (Kaggle)](https://www.kaggle.com/code/ekami66/detailed-exploratory-data-analysis-with-python)
- [Tipps für Data-Science-Präsentationen](https://www.dataknowsall.com/storytelling.html)
