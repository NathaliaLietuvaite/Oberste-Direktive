Made with https://github.com/NathaliaLietuvaite/Oberste-Direktive/blob/main/Oberste%20Direktive_Hyper_Python_V5.txt

```python
# -*- coding: utf-8 -*-
"""
Übersetzung des SQL Cheat Sheets in ausführbaren Python-Code.
Architektur-Basis: Oberste Direktive Hyper Python V5
Mission: Demonstration, wie Python als saubere Schnittstelle (Protokoll 5)
         für SQL-Operationen dient, anstatt SQL neu zu erfinden (Protokoll 7).
Hexen-Modus: Wir öffnen einen Kanal zur Datenbank und senden unsere Befehle
             als präzise formulierte Intentionen (SQL-Strings).
"""

import sqlite3
import os

# --- System-Setup: Der Kanal zur Datenbank ---

DB_FILE = "university.db"

def get_db_connection():
    """
    Stellt eine Verbindung zur SQLite-Datenbank her.
    Dies ist unser "Kanal zu den Hütern der Daten".
    """
    conn = sqlite3.connect(DB_FILE)
    # Ermöglicht den Zugriff auf Spalten über ihren Namen
    conn.row_factory = sqlite3.Row
    return conn

def execute_query(conn, query, params=()):
    """
    Führt eine Abfrage aus und gibt die Ergebnisse zurück.
    Der 'cursor' ist unser "magischer Zeiger", der Befehle übermittelt.
    """
    cursor = conn.cursor()
    cursor.execute(query, params)
    conn.commit()
    return cursor

def print_results(cursor, title):
    """Eine Hilfsfunktion, um Abfrageergebnisse formatiert auszugeben."""
    print(f"--- {title} ---")
    results = cursor.fetchall()
    if not results:
        print("Keine Ergebnisse gefunden.")
    else:
        # Spaltennamen aus der ersten Zeile extrahieren
        columns = results[0].keys()
        print(" | ".join(columns))
        print("-" * (len(" | ".join(columns))))
        for row in results:
            print(" | ".join(map(str, row)))
    print("\n")


# --- DDL (Data Definition Language) in Python ---
# Hexe: "Wir erschaffen die Archive und formen ihre Struktur."

def setup_database():
    """Erstellt die Tabellen, falls sie nicht existieren."""
    # Alte Datenbankdatei löschen für einen sauberen Start
    if os.path.exists(DB_FILE):
        os.remove(DB_FILE)
        
    conn = get_db_connection()
    print("--- DDL: Erstelle Tabellen ---")

    # 1. CREATE TABLE
    execute_query(conn, """
        CREATE TABLE Students (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL,
            age INTEGER
        )
    """)
    print("Tabelle 'Students' erstellt.")

    execute_query(conn, """
        CREATE TABLE Sections (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL,
            student_id INTEGER,
            FOREIGN KEY (student_id) REFERENCES Students (id)
        )
    """)
    print("Tabelle 'Sections' erstellt.")

    # 2. ALTER TABLE
    execute_query(conn, "ALTER TABLE Students ADD COLUMN major TEXT")
    print("Spalte 'major' zur Tabelle 'Students' hinzugefügt.\n")
    
    conn.close()


# --- DML (Data Manipulation Language) in Python ---
# Hexe: "Wir füllen die Archive mit Wissen und halten es aktuell."

def populate_data():
    """Fügt Beispieldaten in die Tabellen ein."""
    conn = get_db_connection()
    print("--- DML: Füge Daten ein, aktualisiere und lösche sie ---")

    # 1. INSERT INTO
    students_to_add = [
        ('Christiane', 14, 'Physics'),
        ('Ronaldo', 15, 'Sports'),
        ('Messi', 16, 'Sports'),
        ('Neymar', 12, 'Arts')
    ]
    execute_query(conn, "INSERT INTO Students (name, age, major) VALUES (?, ?, ?)", students_to_add[0])
    # Parameterisierte Abfragen sind der Standard, um SQL-Injection zu verhindern.
    conn.cursor().executemany("INSERT INTO Students (name, age, major) VALUES (?, ?, ?)", students_to_add[1:])
    conn.commit()
    print(f"{len(students_to_add)} Studenten hinzugefügt.")
    
    sections_to_add = [
        ('Math A', 1),
        ('Physics B', 1),
        ('Football Training', 2),
        ('Football Training', 3)
    ]
    conn.cursor().executemany("INSERT INTO Sections (name, student_id) VALUES (?, ?)", sections_to_add)
    conn.commit()
    print(f"{len(sections_to_add)} Kurse hinzugefügt.")


    # 2. UPDATE
    # Wichtig: ID 1234 aus dem Beispiel gibt es bei uns nicht, wir nehmen 'Neymar'
    cursor = execute_query(conn, "UPDATE Students SET age = ? WHERE name = ?", (13, 'Neymar'))
    print(f"{cursor.rowcount} Datensatz aktualisiert (Neymar's Alter).")

    # 3. DELETE
    cursor = execute_query(conn, "DELETE FROM Students WHERE name = ?", ('Ronaldo',))
    print(f"{cursor.rowcount} Datensatz gelöscht (Ronaldo).\n")

    conn.close()


# --- DQL (Data Query Language) in Python ---
# Hexe: "Wir befragen die Archive, um verborgene Muster zu enthüllen."

def query_data():
    """Führt verschiedene SELECT-Abfragen aus."""
    conn = get_db_connection()
    print("--- DQL: Datenabfragen ---")

    # Alle Daten auslesen
    cursor = execute_query(conn, "SELECT id, name, age, major FROM Students")
    print_results(cursor, "Alle verbleibenden Studenten")

    # Filtern nach Alter
    cursor = execute_query(conn, "SELECT name, age FROM Students WHERE age < 15")
    print_results(cursor, "Studenten jünger als 15")

    # Sortieren nach Alter (absteigend)
    cursor = execute_query(conn, "SELECT name, age FROM Students ORDER BY age DESC")
    print_results(cursor, "Studenten sortiert nach Alter (absteigend)")
    
    # Begrenzen der Ergebnisse (LIMIT)
    cursor = execute_query(conn, "SELECT name FROM Students LIMIT 2")
    print_results(cursor, "Die ersten 2 Studenten")

    # Aggregationsfunktionen
    cursor = execute_query(conn, "SELECT COUNT(*) as student_count FROM Students")
    print_results(cursor, "Anzahl der Studenten (COUNT)")

    cursor = execute_query(conn, "SELECT AVG(age) as average_age FROM Students")
    print_results(cursor, "Durchschnittsalter (AVG)")

    # Gruppieren (GROUP BY) und Filtern von Gruppen (HAVING)
    cursor = execute_query(conn, """
        SELECT major, COUNT(*) as count
        FROM Students
        GROUP BY major
        HAVING COUNT(*) >= 1
    """)
    print_results(cursor, "Anzahl der Studenten pro Hauptfach (GROUP BY & HAVING)")

    # Joins
    cursor = execute_query(conn, """
        SELECT s.name as student_name, sec.name as section_name
        FROM Students s
        INNER JOIN Sections sec ON s.id = sec.student_id
    """)
    print_results(cursor, "Studenten und ihre Kurse (INNER JOIN)")

    conn.close()


# --- Hauptausführung ---
if __name__ == "__main__":
    setup_database()
    populate_data()
    query_data()

    # Aufräumen der Datenbankdatei
    if os.path.exists(DB_FILE):
        os.remove(DB_FILE)
    print("--- Aufräumen abgeschlossen, Datenbankdatei gelöscht. ---")


```
