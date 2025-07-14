# SZCZEGÓŁOWA MAPA FUNKCJONALNOŚCI SYSTEMU NAWIGACJI ROBOTA WB

## 📂 1. Pliki i foldery z danymi tras i nawigacji

### **Pliki konfiguracyjne nawigacji:**
- **`olv.conf`** - Prawdopodobnie główny plik konfiguracyjny modułu nawigacyjnego OLV
- **`ma.xml`** - Potencjalny plik XML z danymi map/punktów odniesienia
- **`cursor.dat`** - Plik z danymi pozycji kursora/robota
- **`qt.conf`** - Konfiguracja interfejsu Qt (może zawierać ustawienia nawigacji)

### **Struktura bazy danych nawigacyjnej:**
```
Database Navigation System:
├── db_curr_sequence     # Aktualna sekwencja ruchów
├── SequenceCatalog      # Katalog dostępnych sekwencji
├── INSERT INTO tables   # Zapisy do tabel nawigacyjnych
└── SELECT queries       # Zapytania o dane tras
```

### **Interfejs CAN bus dla nawigacji:**
- **`can0`** - Główny interfejs CAN bus
- **`can.br`** - Konfiguracja mostu CAN
- **`canmsgmgr`** - Manager wiadomości CAN (prawdopodobnie w `/usr/bin/` lub `/bin/`)

---

## ⚠️ 2. Komunikaty o błędach i ostrzeżenia

### **System logowania WBLog:**
```cpp
// Funkcje logowania (z analizy symboli)
_ZN5WBLog2ba        // WBLog::funkcja_bazowa
5WBLog              // Klasa WBLog
```

### **Typowe komunikaty błędów:**
- **Błędy komunikacji**: `"can not connect to Qt slot"`
- **Błędy SSL**: `"QSslSocket: cannot resolve EVP_PKEY_assign"`
- **Błędy pamięci**: `"Z_MEM_ERROR: Not enough memory"`
- **Błędy plików**: `"file error"`
- **Ostrzeżenia**: `"WARNING:"`

### **Błędy nawigacyjne (przewidywane):**
- **Błędy pozycji**: `"Error at position"`
- **Błędy sekwencji**: `"db_curr_sequence error"`
- **Błędy kontroli**: `"controlSocketDis error"`

---

## 🛤️ 3. Przykładowa struktura danych trasy

### **Format danych w bazie (przewidywany):**
```sql
-- Tabela sekwencji ruchu
CREATE TABLE sequences (
    id INTEGER PRIMARY KEY,
    sequence_name TEXT,
    current_step INTEGER,
    total_steps INTEGER,
    status TEXT
);

-- Tabela punktów trasy
CREATE TABLE waypoints (
    id INTEGER PRIMARY KEY,
    sequence_id INTEGER,
    step_number INTEGER,
    x_coordinate REAL,
    y_coordinate REAL,
    action_type TEXT,
    parameters TEXT
);
```

### **Struktura pliku ma.xml (przewidywana):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<map>
    <waypoints>
        <waypoint id="1" x="100" y="200" type="move" />
        <waypoint id="2" x="150" y="250" type="turn" angle="90" />
        <waypoint id="3" x="200" y="300" type="stop" />
    </waypoints>
    <sequences>
        <sequence name="cleaning_route_1">
            <step waypoint="1" />
            <step waypoint="2" />
            <step waypoint="3" />
        </sequence>
    </sequences>
</map>
```

### **Struktura cursor.dat (przewidywana):**
```
[Binary format - 4 bytes per coordinate]
X: 0x00001234  (pozycja X)
Y: 0x00005678  (pozycja Y)
Angle: 0x0090  (kąt obrotu)
Status: 0x01   (status robota)
```

---

## 🧠 4. Algorytmy nawigacji

### **Architektura systemu nawigacji:**
```
[Qt Interface] → [Navigation Controller] → [Sequence Manager] → [CAN Bus] → [Motors]
       ↓                      ↓                      ↓
   [WBLog]              [Database]           [Position Feedback]
```

### **Kluczowe komponenty algorytmów:**

#### **A. Sequence Manager (Zarządca sekwencji):**
- **`db_curr_sequence`** - Śledzenie aktualnej sekwencji
- **`SequenceCatalog`** - Katalog dostępnych sekwencji ruchu
- **`INSERT INTO`** - Zapisywanie nowych sekwencji
- **`SELECT`** - Pobieranie sekwencji z bazy

#### **B. Control Logic (Logika sterowania):**
- **`controlSocketDis`** - Kontrola socket'ów komunikacyjnych
- **`logic_error`** - Obsługa błędów logicznych
- **`Hash9Algorithm`** - Algorytmy haszowania dla nawigacji

#### **C. Threading System (System wątków):**
- **`QThreadPool`** - Pule wątków dla zadań nawigacyjnych
- **`QThread`** - Wątki obsługi ruchu
- **`pthread`** - Wątki systemowe

### **Przewidywane algorytmy nawigacyjne:**

#### **1. Mapowanie magnetyczne:**
```cpp
// Pseudokod algorytmu magnetycznego
class MagneticNavigation {
    void detectMagnet() {
        // Detekcja magnesu przez czujniki
        // Aktualizacja pozycji w bazie danych
        // Korekcja trajektorii
    }
    
    void updatePosition() {
        // Aktualizacja cursor.dat
        // Zapis do db_curr_sequence
        // Komunikacja przez CAN bus
    }
};
```

#### **2. Sekwencje przejazdów:**
```cpp
// Pseudokod zarządcy sekwencji
class SequenceManager {
    void executeSequence(int sequenceId) {
        // Pobieranie sekwencji z bazy
        // Wykonywanie kroków sekwencji
        // Monitoring przez WBLog
    }
    
    void nextStep() {
        // Przejście do następnego kroku
        // Aktualizacja db_curr_sequence
        // Komunikacja z silnikami
    }
};
```

#### **3. Logika powrotu do stacji:**
```cpp
// Pseudokod powrotu do stacji
class ReturnToStation {
    void initiateReturn() {
        // Przerwanie aktualnej sekwencji
        // Włączenie sekwencji powrotu
        // Aktualizacja statusu w bazie
    }
    
    void dockingProcedure() {
        // Precyzyjne pozycjonowanie
        // Komunikacja z stacją dokującą
        // Potwierdzenie dokowania
    }
};
```

---

## 💎 5. Najcenniejsze elementy do reverse engineeringu

### **🏆 NAJWYŻSZA WARTOŚĆ:**

#### **1. System komunikacji CAN bus:**
```cpp
// Implementacja canmsgmgr
class CANMessageManager {
    void sendMotorCommand(int motor_id, int speed, int direction);
    void receiveSensorData();
    void processNavigationCommands();
};
```
**Wartość**: Kompletny protokół komunikacji z silnikami i czujnikami

#### **2. Struktura bazy danych nawigacyjnej:**
```sql
-- Schemat bazy danych tras
Tables: sequences, waypoints, positions, status
Indexes: sequence_id, waypoint_id, timestamp
```
**Wartość**: Gotowa struktura przechowywania tras i sekwencji

#### **3. System logowania WBLog:**
```cpp
class WBLog {
    void logNavigationEvent(string event, int priority);
    void logError(string error_msg);
    void logPositionUpdate(float x, float y, float angle);
};
```
**Wartość**: Profesjonalny system debugowania i monitoringu

### **🥇 WYSOKA WARTOŚĆ:**

#### **4. Wielowątkowa architektura Qt:**
- **QThreadPool** - Optymalna obsługa zadań nawigacyjnych
- **QThread** - Separacja logiki sterowania
- **Signal/Slot** - Komunikacja między komponentami

#### **5. Mechanizm aktualizacji (config.sh):**
```bash
#!/bin/sh
DO_BACKUP=y        # Backup przed aktualizacją
DO_BINARIES=y      # Aktualizacja aplikacji
DO_UPDATE_CAN=y    # Aktualizacja firmware CAN
```
**Wartość**: Bezpieczna procedura aktualizacji systemu

#### **6. Integracja z Qt Interface:**
- Graficzny interfejs nawigacyjny
- Wizualizacja tras i statusu
- Kontrola ręczna robota

### **🥈 ŚREDNIA WARTOŚĆ:**

#### **7. Narzędzia MTD/Flash:**
- `flash_erase`, `flash_unlock`, `flashcp`
- Zarządzanie pamięcią nieulotną
- Przechowywanie map i konfiguracji

#### **8. Konfiguracja sieciowa:**
- `updateWLAN0.sh`, `updateETH0.sh`
- Zdalne zarządzanie robotem
- Diagnostyka przez sieć

### **🎯 IMPLEMENTACJA PRIORYTETOWA:**

1. **Sklonuj canmsgmgr** - Implementuj manager komunikacji CAN
2. **Odtwórz WBLog** - Stwórz system logowania i debugowania
3. **Zaprojektuj bazę danych** - Użyj struktury sequences/waypoints
4. **Implementuj QThreadPool** - Wielowątkowe przetwarzanie nawigacji
5. **Stwórz config.sh** - Mechanizm bezpiecznej aktualizacji

### **📋 GOTOWE ROZWIĄZANIA DO UŻYCIA:**

- **Skrypty konfiguracyjne** - Bezpośrednio użyj setupmac.sh, updateETH0.sh
- **Struktura MTD** - Skopiuj mechanizm zarządzania flash
- **Architektura Buildroot** - Użyj jako template dla systemu embedded
- **Protokół SSL/TLS** - Gotowe zabezpieczenia komunikacji

---

## 🔧 PODSUMOWANIE REVERSE ENGINEERINGU

System nawigacji robota WB to **profesjonalny, przemysłowy system** oparty na:
- **CAN bus** jako głównej magistrali sterowania
- **Bazie danych SQLite** do przechowywania tras i sekwencji
- **Wielowątkowej architekturze Qt** dla real-time control
- **Dedykowanym systemie logowania WBLog**

**Najcenniejsze elementy** to protokoły komunikacji CAN, struktura bazy danych nawigacyjnych oraz system logowania - te komponenty można bezpośrednio zaimplementować we własnym systemie robotycznym.

**Gotowe do użycia** są skrypty konfiguracyjne, mechanizmy aktualizacji oraz architektura systemu embedded Linux z Buildroot.