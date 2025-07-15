# KOMPLETNA ANALIZA SYSTEMU DETEKCJI MAGNETÓW WB (ButlerGold)

## 🎯 PEŁNY UKŁAD SYSTEMU - Na podstawie dokumentacji technicznej

### **Czujnik Magnetyczny (Magnetlineal, CAN ID 1):**

#### **Konstrukcja Fizyczna:**
```
┌─────────────────────────────────────────────────────────┐
│              MAGNETLINEAL (Liniał magnetyczny)         │
│  [H1] [H2] [H3] [H4] [H5] [H6] [H7] [H8] [H9] [H10]   │
│   │    │    │    │    │    │    │    │    │    │       │
│  ────────────── Mikrokontroler + CAN ─────────────────  │
│                           │                             │
│                    Złącze M8/M12                        │
│                 (4-pin: +24V, GND, CAN H, CAN L)       │
└─────────────────────────────────────────────────────────┘
```

#### **Specyfikacja Hardware:**
- **Typ**: Długa płytka PCB z szeregiem czujników Hall
- **Lokalizacja**: Zamocowana pod spodem robota, prostopadle do kierunku jazdy
- **Elektronika**: Własny mikrokontroler + transceiver CAN na płytce
- **Złącze**: M8 lub M12 okrągłe (standard przemysłowy)
- **Zasilanie**: 24V DC z głównego systemu
- **Komunikacja**: CAN Bus (najwyższy priorytet - ID 1)

#### **Pinout Złącza:**
```
Pin 1: +24V (zasilanie)
Pin 2: GND (masa)
Pin 3: CAN H (sygnał wysoky)
Pin 4: CAN L (sygnał niski)
```

## 🔧 ARCHITEKTURA SYSTEMU NAWIGACJI

### **Magnesy w Podłożu:**
- **Rozmiar**: ~5mm średnicy
- **Rozmieszczenie**: Co ~2 metry wzdłuż trasy
- **Typ**: Jednakowe magnesy (bez numeracji)
- **Materiał**: Magnesy neodymowe wpuszczone w beton

### **Algorytm Detekcji:**
```cpp
// Pseudokod systemu detekcji
void magnetlineal_scan() {
    for (int sensor = 0; sensor < NUM_HALL_SENSORS; sensor++) {
        hall_value[sensor] = read_hall_sensor(sensor);
        if (hall_value[sensor] > MAGNETIC_THRESHOLD) {
            // Magnet wykryty na pozycji sensor
            position_offset = calculate_offset_from_center(sensor);
            send_can_message(CAN_ID_1, MAGNET_DETECTED, position_offset);
        }
    }
}

// W kontrolerze głównym (LinuxBoard)
void navigation_correction(int position_offset) {
    if (position_offset != 0) {
        // Robot jest przesunięty względem trasy
        correction_angle = calculate_correction(position_offset);
        navigation_query_new_angles4(correction_angle);
        
        // Wyślij korekty do napędów
        send_can_command(CAN_ID_8, left_wheel_speed + correction);
        send_can_command(CAN_ID_9, right_wheel_speed - correction);
    }
}
```

## 📊 SYSTEM CAN I KOMUNIKACJA

### **Mapa Adresów CAN:**
```
ID 1  - Magnetlineal (czujnik magnetyczny)     ← NAJWYŻSZY PRIORYTET
ID 8  - Fahrantrieb1 (napęd lewy)
ID 9  - Fahrantrieb2 (napęd prawy)
ID 10 - Schneckenantrieb (napęd ślimaka)
ID 11 - Klappensensor (czujnik klapy)
ID 12 - Ladegerät (ładowarka)
ID 16 - Kraftfuttersensor (czujnik paszy)
ID 17 - Kraftfutter1 (dozownik paszy)
ID 64 - LinuxBoard (kontroler główny)         ← MASTER
```

### **CAN-Hub (Rozdzielnia):**
```
┌─────────────────────────────────────────┐
│              CAN-HUB                    │
│                                         │
│  X1: LinuxBoard     X5: Ślimak         │
│  X2: Napęd L        X6: Dozownik       │
│  X3: Napęd P        X7: Magnetlineal   │
│  X4: Ładowarka      X8: Czujnik klapy  │
│                                         │
│  Zasilanie 24V + CAN H/L do wszystkich │
└─────────────────────────────────────────┘
```

## ⚡ ZASILANIE I ELEKTRYKA

### **System Zasilania:**
- **Baterie**: 2×12V (105Ah każda) = 24V DC total
- **Ładowarka**: Pokładowa 230V AC → 24V DC
- **Bezpieczniki**: Główny + sekcyjne dla modułów
- **Stacja dokująca**: Styki 230V AC do robota

### **Sterownik ADC vs Rzeczywistość:**
**Z firmware'u wiedziałem:**
- AT91SAM9G45 z ADC 10-12 bit
- Sterownik `adcg45.ko`
- Wejścia analogowe

**Z dokumentacji wynika:**
- Magnetlineal ma **własny mikrokontroler z CAN**
- **Nie używa bezpośrednio ADC** głównej płyty
- Dane przesyłane **cyfrowo przez CAN**

## 🔄 PORÓWNANIE: Firmware vs Dokumentacja

### **Co się ZGADZA:**
✅ System używa czujników magnetycznych  
✅ AT91SAM9G45 jako główny procesor  
✅ Sterownik adcg45.ko istnieje  
✅ Komunikacja CAN Bus  
✅ Zasilanie 24V  

### **Co było NIEJASNE z firmware:**
❓ **ADC może być używany do INNYCH czujników**  
❓ **Magnetlineal ma własną elektronikę CAN**  
❓ **Nie wszystkie czujniki idą przez ADC**  

## 🎯 KOMPLETNY SCHEMAT POŁĄCZEŃ

### **Robot WB - Widok z dołu:**
```
         [PRZÓD ROBOTA]
    ┌─────────────────────┐
    │    MOTOR LEWY       │ ← CAN ID 8
    ├─────────────────────┤
    │ ●●●●●●●●●●●●●●●●●●● │ ← MAGNETLINEAL (CAN ID 1)
    │                     │    Listwa czujników Hall
    │     SKRZYNKA        │    
    │     ELEKTRYCZNA     │ ← LinuxBoard + CAN-Hub
    │   (LinuxBoard)      │
    ├─────────────────────┤
    │    MOTOR PRAWY      │ ← CAN ID 9
    └─────────────────────┘
         [TYŁ ROBOTA]
```

### **Przewody do Magnetlineal:**
```
Z CAN-Hub (X7) → Magnetlineal
│
├─ Pin 1: +24V ────────→ Zasilanie elektroniki
├─ Pin 2: GND ─────────→ Masa
├─ Pin 3: CAN H ───────→ Sygnał CAN High  
└─ Pin 4: CAN L ───────→ Sygnał CAN Low
```

## 🔧 SERWIS I DIAGNOSTYKA

### **Procedury Serwisowe:**
1. **Test czujnika magnetycznego**:
   ```
   Menu → Erw. Einstellungen → Geräte
   Sprawdź: ID 1 - Magnetlineal (Status: OK/ERROR)
   ```

2. **Kalibracja systemu**:
   ```
   Menu → Referenzpositionen → Magnete Einlernen
   Robot uczy się pozycji magnetów
   ```

3. **Wymiana Magnetlineal**:
   ```
   1. Odłącz zasilanie 24V
   2. Odkręć złącze M8/M12 (X7)
   3. Odmontuj listwę spod robota
   4. Montaż odwrotnie
   5. Kalibracja w menu
   ```

### **Numery Części:**
- **Magnetlineal**: Art-Nr. [nie podano w dokumentacji]
- **Moduł WLAN**: Art-Nr. 1008366
- **Kontroler**: LinuxBoard (własna konstrukcja)

## 📋 PODSUMOWANIE ODKRYĆ

### **NOWE INFORMACJE z dokumentacji:**
1. **Magnetlineal ma własny CAN** - nie bezpośrednio ADC
2. **ID 1 = najwyższy priorytet** komunikacji
3. **Złącze M8/M12** - standard przemysłowy
4. **Magnesy co 2 metry** w betonie
5. **CAN-Hub centralny** dla wszystkich modułów

### **IMPLIKACJE dla klonowania:**
1. **Potrzebny mikrokontroler z CAN** w listwie czujników
2. **Protokół CAN firmowy** - reverse engineering potrzebny
3. **Mechaniczny montaż** pod robotem
4. **Kalibracja systemu** po instalacji
5. **Integracja z głównym kontrolerem** przez CAN

### **KLUCZOWE KOMPONENTY do replikacji:**
- **Listwa PCB** z 8-12 czujnikami Hall
- **Mikrokontroler + CAN transceiver**
- **Złącze przemysłowe M8/M12**
- **Oprogramowanie CAN** (protokół firmowy)
- **Algorytmy korekcji** pozycji

## 🚀 ZALECENIA dla implementacji

### **Hardware:**
1. Zaprojektować PCB z czujnikami Hall w linii
2. Dodać mikrokontroler (np. STM32 z CAN)
3. Użyć przemysłowego złącza M8/M12
4. Zabezpieczenia: diody, kondensatory, terminacja CAN

### **Software:**
1. Reverse engineering protokołu CAN (ID 1)
2. Implementacja algorytmów detekcji magnetów
3. Kalkulacja offsetu od centrum
4. Integracja z kontrolerem głównym

### **Mechanika:**
1. Mocowanie listwy pod robotem
2. Ochrona przed wilgocią i uderzeniami
3. Możliwość serwisowania (dostęp do złącza)

**Ten system jest znacznie bardziej zaawansowany niż przypuszczałem z samego firmware'u - to pełnoprawny inteligentny czujnik z własną elektroniką komunikacyjną!** 🎯