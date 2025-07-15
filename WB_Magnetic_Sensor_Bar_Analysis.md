# Analiza Belki Detekcji Magnetów Robota WB

## ❌ OGRANICZENIA DOKUMENTACJI

### Co **NIE ZNALAZŁEM** w systemie:
- ❌ Schematów elektrycznych
- ❌ Dokumentacji technicznej układu
- ❌ Plików CAD lub rysunków
- ❌ Szczegółowej konfiguracji pinów
- ❌ Bezpośrednich informacji o liczbie czujników
- ❌ Układu mechanicznego belki

## 🔍 CO UDAŁO SIĘ USTALIĆ z Firmware'u

### **Informacje o Sterowniku ADC:**
```
Sterownik: adcg45.ko
Opis: "ADC Driver for AT91SAM9G45"
Autor: Georg Ottinger
Licencja: GPL
Wersja jądra: 2.6.38
Parametr: idle_samplerate (konfigurowalna częstotliwość próbkowania)
```

### **Specyfikacja Hardware:**
```
Procesor: AT91SAM9G45 (ARM9)
ADC: Wbudowany 10-12 bit
Kanały: Wielokanałowy (AT91SAM9G45 ma 8 kanałów ADC)
Rozdzielczość: 1024-4096 poziomów
Funkcje: adcg45_get(), adcg45_get_channelL()
```

### **Parametry Konfiguracyjne:**
```bash
insmod /driver/adcg45.ko idle_samplerate=1000
sc_threshold=40  # Próg detekcji czujników
```

### **Identyfikowane Piny GPIO/ADC:**
```
Porty dostępne:
- PA0-PA7 (Port A)
- PB2 (Port B) 
- PC0-PC2 (Port C)
- PD1, PD3, PD6, PD9 (Port D)
```

## 🔧 REKONSTRUKCJA HIPOTETYCZNA UKŁADU

### **Na podstawie analizy mogę wywnioskować:**

#### 1. **Typ Belki:**
```
┌─────────────────────────────────────────┐
│          BELKA CZUJNIKÓW HALL           │
│  [H1] [H2] [H3] [H4] [H5] [H6] [H7]     │
│   │    │    │    │    │    │    │       │
│  ADC0 ADC1 ADC2 ADC3 ADC4 ADC5 ADC6     │
└─────────────────────────────────────────┘
```

#### 2. **Prawdopodobna Konfiguracja:**
- **3-7 czujników Hall** (analogowych)
- **Rozmieszczenie liniowe** pod robotem
- **Podłączenie do kanałów ADC** AT91SAM9G45
- **Analogowe wyjście** 0-5V

#### 3. **Schemat Połączeń:**
```
Czujnik Hall → Filtr analogowy → ADC Channel → AT91SAM9G45 → Software
     ↓               ↓              ↓           ↓           ↓
   0-5V        Stabilizacja    10-12 bit    DMA/IRQ    Navigation
```

#### 4. **Fizyczna Konstrukcja (hipotetyczna):**
```
Robot WB (widok z dołu):
         [FRONT]
    ┌─────────────┐
    │    MOTOR    │
    │      L      │
    ├─────────────┤
    │ [●][●][●]   │  ← BELKA CZUJNIKÓW
    │ [●][●][●]   │     (pod środkiem robota)
    ├─────────────┤
    │    MOTOR    │
    │      R      │
    └─────────────┘
         [REAR]
```

#### 5. **Charakterystyka Czujników:**
```
Typ: Analogowe czujniki Hall
Zakres: 0-5V (lub 0-3.3V)
Próg: sc_threshold=40 (z 4096 poziomów)
Częstotliwość: idle_samplerate=1000 Hz
Filtracja: Hardware + software
```

## 📋 WNIOSKI z ANALIZY

### **Co prawdopodobnie zawiera belka:**
1. **4-6 czujników Hall** w układzie liniowym
2. **PCB z filtrowaniem analogowym**
3. **Connector do głównej płyty**
4. **Montaż pod środkiem robota**
5. **Ochrona mechaniczna**

### **Funkcje belki:**
```cpp
// Odczyt wszystkich czujników
for (int i = 0; i < NUM_SENSORS; i++) {
    sensor_value[i] = adcg45_get_channelL(i);
    if (sensor_value[i] > sc_threshold) {
        // Magnet wykryty na pozycji i
        process_magnetic_detection(i, sensor_value[i]);
    }
}
```

### **Algorytm detekcji:**
1. **Próbkowanie** wszystkich kanałów ADC
2. **Porównanie z progiem** (sc_threshold=40)
3. **Określenie pozycji** magnetu na belce
4. **Wywołanie korekcji** nawigacji

## ⚠️ UWAGI

**Powyższa rekonstrukcja jest HIPOTETYCZNA** oparta na:
- ✅ Analizie sterowników
- ✅ Specyfikacji AT91SAM9G45
- ✅ Typowych rozwiązaniach robotycznych
- ✅ Logice kodu nawigacyjnego

**Dla dokładnych informacji potrzebne byłoby:**
- 📋 Oryginalna dokumentacja techniczna
- 🔧 Fizyczny dostęp do robota
- 📐 Schematy elektryczne
- 📖 Specyfikacja producenta

**System firmware'u sugeruje zaawansowany, wielokanałowy detektor magnetyczny z dużą precyzją i konfigurowalnością.**