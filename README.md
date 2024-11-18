# MongoDB Local Database Setup and Sharing Guide

Dieses Projekt beschreibt, wie eine lokale MongoDB-Datenbank mit Docker eingerichtet und Daten aus einer CSV-Datei importiert werden können. Außerdem wird erklärt, wie die Datenbank exportiert und an andere weitergegeben werden kann.

## Voraussetzungen
- **Docker**: Für die einfache Einrichtung und Portabilität der MongoDB-Umgebung.
- **Python** (optional): Für den Datenimport und -verarbeitung mit `pandas` und `pymongo`.
- **MongoDB-Tools** (falls Docker nicht verwendet wird): `mongodump` und `mongorestore` für das Exportieren und Importieren.

## 1. MongoDB mit Docker starten
1. Lade und installiere [Docker](https://www.docker.com/get-started), falls noch nicht installiert.
2. Starte eine MongoDB-Instanz mit Docker:
    ```bash
    docker run --name mongo -d -p 27017:27017 mongo
    ```

MongoDB läuft nun auf `localhost` und ist über den Standardport `27017` erreichbar.

## 2. CSV-Daten in MongoDB importieren

1. Stelle sicher, dass `pandas` und `pymongo` installiert sind:
    ```bash
    pip install pandas pymongo
    ```

2. Verwende das folgende Python-Skript, um CSV-Daten in MongoDB zu importieren:

    ```python
    import pandas as pd
    from pymongo import MongoClient

    # CSV-Datei laden
    csv_data = pd.read_csv('path/to/your/csv_file.csv')

    # Verbindung zu MongoDB herstellen
    client = MongoClient('mongodb://localhost:27017/')
    db = client['database_name']  # Ersetze 'database_name' durch deinen Datenbanknamen
    collection = db['collection_name']  # Ersetze 'collection_name' durch den Namen der Sammlung

    # CSV-Daten in JSON umwandeln und in MongoDB einfügen
    collection.insert_many(csv_data.to_dict('records'))
    ```

3. **Anpassen des Skripts**:
    - Ersetze `'path/to/your/csv_file.csv'` mit dem Pfad zu deiner CSV-Datei.
    - Passe `database_name` und `collection_name` an deine Datenbankstruktur an.

## 3. Exportieren der MongoDB-Datenbank

Um die MongoDB-Datenbank für die Weitergabe an andere zu exportieren:

1. Führe den folgenden Befehl aus, um die Datenbank zu exportieren:
    ```bash
    mongodump --db database_name --out /path/to/export
    ```
   - Ersetze `database_name` mit dem Namen deiner Datenbank.
   - Ersetze `/path/to/export` durch den Pfad, in dem der Export gespeichert werden soll.

Dies erstellt ein Verzeichnis im angegebenen Pfad mit einer BSON-Datei, die die gesamte Datenbank enthält.

## 4. Importieren der Datenbank

Der Empfänger kann die exportierte Datenbank wie folgt importieren:

1. Stelle sicher, dass MongoDB läuft.
2. Führe den folgenden Befehl aus, um die Datenbank zu importieren:
    ```bash
    mongorestore --db database_name /path/to/export/database_name
    ```
   - Ersetze `database_name` durch den Namen der zu importierenden Datenbank.
   - Ersetze `/path/to/export/database_name` durch den Pfad zur exportierten Datenbank.

## 5. Docker-Container mit Datenvolumen weitergeben (Optional)

Falls du die Datenbank im Docker-Container weitergeben möchtest, kannst du den Container in ein Image umwandeln und exportieren:

1. **Container in ein Image umwandeln**:
    ```bash
    docker commit container_id your_mongo_image
    ```

2. **Image exportieren**:
    ```bash
    docker save -o mongo_image.tar your_mongo_image
    ```

3. **Image importieren und Container starten**:
    Der Empfänger kann das Image laden und starten:
    ```bash
    docker load -i mongo_image.tar
    docker run -d -p 27017:27017 your_mongo_image
    ```

## 6. Sicherheitshinweise
- Achte darauf, dass keine sensiblen Daten im Image oder Dump enthalten sind, bevor du sie weitergibst.
- Setze Sicherheitsmaßnahmen um, wenn du die Datenbank für den Netzwerkzugriff freigibst.

## Weiterführende Ressourcen
- [MongoDB Documentation](https://docs.mongodb.com/)
- [Docker Documentation](https://docs.docker.com/)

---

