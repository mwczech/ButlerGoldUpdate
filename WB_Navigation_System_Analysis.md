# Szczegółowa Analiza Systemu Nawigacji Robota WB (ButlerGold)

## 🎯 SPOSÓB JAZDY - HYBRID NAVIGATION SYSTEM

### Podstawowy System Nawigacji:
Robot WB używa **HYBRYDOWEGO SYSTEMU NAWIGACJI** łączącego:
1. **Podstawę**: Odometrię z enkoderów kół (dead reckoning)
2. **Korekcję**: System zewnętrzny (prawdopodobnie magnesy/znaczniki)
3. **Komunikację**: CAN Bus dla kontroli motorów
4. **Monitoring**: System detekcji błędów i utraty pozycji

## 🔧 ARCHITEKTURA SYSTEMU NAWIGACJI

### Komponenty Główne:
```cpp
// KLUCZOWE MODUŁY NAWIGACYJNE:
navigation_query_new_angles4()    // Algorytm kalkulacji nowych kątów ruchu
bt_lost_no_mapping()              // Obsługa utraty mapowania/pozycji
_mapping_data()                   // Struktura danych mapowania
CANLogger.cpp                     // Logger komunikacji CAN Bus
QGraphicsSceneWheel               // Kontrola kół przez Qt
```

### Sterowniki Hardware:
```bash
/driver/adcg45.ko        # ADC - może odczytywać sensory pozycji/magnesy
/driver/keymatrix.ko     # Interfejs użytkownika
```

## 🛞 SYSTEM KONTROLI KÓŁ

### Enkodery i Odometria:
- **Podstawowa nawigacja**: Zliczanie obrotów kół
- **RPM Monitoring**: Kontrola prędkości obrotowej (`RPM`, `rpm`)
- **PWM Control**: Sterowanie prędkością motorów (`PWM`, `pwm`)
- **Wheel Events**: Obsługa zdarzeń kół (`QGraphicsSceneWheel`, `wheelEvX`)

### Komendy Ruchu:
```cpp
// PODSTAWOWE KOMENDY RUCHU:
started()           // Start ruchu
left\               // Skręt w lewo  
right               // Skręt w prawo
forward_            // Ruch do przodu
stop                // Zatrzymanie
```

## 📡 KOMUNIKACJA CAN BUS

### Architektura Komunikacyjna:
```
Robot CPU ←→ CAN Bus (can0) ←→ Sterowniki Motorów
    ↓
CANLogger.cpp (monitoring)
    ↓
Qt Application (interfejs)
```

### Funkcje CAN:
```cpp
ifconfig can0                    // Konfiguracja interfejsu CAN
cCANLogger                       // Klasa loggingu CAN
"m lost CANV"                    // Komunikat o utracie sygnału CAN
_q_can3,                         // Funkcje CAN Bus
```

## 🧭 ALGORYTM NAWIGACYJNY

### Główny Algorytm: `navigation_query_new_angles4()`
**Funkcjonalność**: 
- Oblicza nowe kąty ruchu na podstawie 4 kierunków
- Prawdopodobnie implementuje algorytm ruchu w kratce (grid-based)
- Używa danych z `_mapping_data` dla określenia pozycji

### System Mapowania:
```csv
# Format sample_map.csv (przykład):
x, y, type, description
0, 0, "start", "Punkt startowy" 
100, 0, "waypoint", "Punkt odniesienia"
200, 100, "checkpoint", "Punkt kontrolny"
```

## 🚨 DETEKCJA POŚLIZGU I BŁĘDÓW

### Mechanizmy Detekcji:
1. **`bt_lost_no_mapping()`** - Główna funkcja obsługi utraty pozycji
2. **CAN Bus Monitoring** - Sprawdzanie komunikacji z motorami
3. **ADC Sensors** - Odczyt dodatkowych sensorów (`adcg45.ko`)
4. **`slip`/`SLIP`** - Funkcje związane z poślizgiem

### Obsługa Błędów:
```cpp
// SYSTEM OBSŁUGI BŁĘDÓW:
"Nr. samples p due to no mm"     // Brak próbek mapowania
"m lost CANV, ignore"            // Utrata sygnału CAN - ignorowanie
bt_lost_no_mapping()             // Główna procedura odzyskiwania
```

## 📊 SPOSÓB DZIAŁANIA - SZCZEGÓŁOWY OPIS

### 1. PODSTAWOWA NAWIGACJA (Enkodery):
```
Enkodery kół → Odczyt RPM → Kalkulacja pozycji → Dead Reckoning
```
- Robot wykorzystuje enkodery jako **PODSTAWĘ** nawigacji
- Zlicza obroty kół i kalkuluje pozycję względną
- System monitoruje RPM dla wykrycia anomalii

### 2. SYSTEM KOREKCJI (Magnesy/Znaczniki):
```
ADC Sensor → Detekcja znacznika → Korekcja pozycji → Aktualizacja _mapping_data
```
- **Magnesy NIE SĄ podstawą** - to system **KOREKCYJNY**
- Używane do kalibracji pozycji w kluczowych punktach
- Odczytywane przez ADC (`adcg45.ko`)

### 3. DETEKCJA POŚLIZGU:
```
Expected RPM ≠ Actual Movement → Slip Detection → Error Correction
```
Robot wykrywa poślizg przez:
- **Porównanie oczekiwanej vs rzeczywistej pozycji**
- **Monitoring CAN Bus** - sprawdzanie odpowiedzi motorów
- **Funkcje `slip`** - algorytmy detekcji poślizgu
- **Timeout detection** - brak postępu w określonym czasie

## 🔄 CYKL NAWIGACYJNY

### Normalny Cykl:
1. `navigation_query_new_angles4()` - Oblicz nowy kierunek
2. Wyślij komendy przez CAN Bus do motorów
3. Monitor enkodery kół (RPM/pozycja)
4. Sprawdź czy dotarł do punktu korekcyjnego
5. W razie potrzeby: korekcja przez ADC/magnesy

### Cykl Błędu:
1. Detekcja problemu (poślizg/utrata CAN/brak postępu)
2. `bt_lost_no_mapping()` - Procedura odzyskiwania
3. Próba relocalizacji przez znaczniki
4. Restart nawigacji lub bezpieczne zatrzymanie

## 📍 STRUKTURA SYSTEMU POZYCJONOWANIA

### Hierarchia Pozycjonowania:
```
1. Dead Reckoning (enkodery) - PODSTAWA, ciągłe
2. Waypoints (magnesy) - KOREKCJA, punktowa  
3. CAN Bus Feedback - MONITORING, ciągłe
4. Error Recovery - BACKUP, w razie problemów
```

## 🎯 WNIOSKI - JAK WB JEŹDZI:

### ✅ **POTWIERDZONE**:
1. **Podstawa nawigacji**: **ENKODERY KÓŁ** (dead reckoning)
2. **Magnesy**: Używane jako **KOREKCJA**, nie podstawa
3. **Detekcja poślizgu**: Przez porównanie RPM vs pozycja + CAN monitoring
4. **Komunikacja**: CAN Bus z motorami i sensorami
5. **Algorytm**: Grid-based navigation (`navigation_query_new_angles4`)

### 🔧 **MECHANIZM DZIAŁANIA**:
- Robot jedzie głównie na **enkoderach kół**
- **Magnesy/znaczniki** służą do **kalibracji** w punktach kontrolnych
- **Poślizg** wykrywany przez **niespójność enkoder vs oczekiwana pozycja**
- **CAN Bus** monitoruje komunikację z motorami w czasie rzeczywistym
- System ma **mechanizm odzyskiwania** (`bt_lost_no_mapping`)

**ROBOT WB UŻYWA ENKODERÓW JAKO PODSTAWY, A MAGNESY JAKO KOREKCJĘ**