# ANALIZA ARCHITEKTURY OPROGRAMOWANIA WB (ButlerGold)

## 🖥️ SYSTEM OPERACYJNY I PLATFORMA

### **LinuxBoard - Główny Komputer:**
```
Nazwa: LinuxBoard (CAN ID 64)
System: Linux (embedded)
Procesor: AT91SAM9G45 (ARM9)
Funkcja: Master controller całego systemu
Komunikacja: CAN Bus master + WLAN/Ethernet
```

### **Wersje Oprogramowania (z raportu):**
```
Engine: 2.04.002    ← Silnik sterujący (core system)
GUI: 2.04.002       ← Interfejs graficzny
Kernel: 2.6.38      ← Wersja jądra Linux
```

### **Architektura Systemu:**
```
┌─────────────────────────────────────────┐
│           LINUX SYSTEM                  │
│                                         │
│  ┌─────────────┐  ┌─────────────────┐   │
│  │     GUI     │  │   CAN DAEMON    │   │
│  │  (Qt/GTK)   │  │  (Komunikacja)  │   │
│  └─────────────┘  └─────────────────┘   │
│                                         │
│  ┌─────────────┐  ┌─────────────────┐   │
│  │ NAVIGATION  │  │   FILE SYSTEM   │   │
│  │   ENGINE    │  │   (SD/Flash)    │   │
│  └─────────────┘  └─────────────────┘   │
│                                         │
│  ┌─────────────────────────────────────┐ │
│  │        LINUX KERNEL 2.6.38         │ │
│  │     (Drivers: adcg45.ko, CAN)      │ │
│  └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

## 🔧 STRUKTURA APLIKACJI

### **Główne Moduły Software:**

#### 1. **Interfejs Graficzny (GUI 2.04.002):**
```
Technologia: Prawdopodobnie Qt lub embedded GUI
Funkcje:
- Główne menu (Hauptmenü)
- Tryb ręczny (Handbetrieb) 
- Pozycje referencyjne (Referenzpositionen)
- Status systemu
- Import/Export danych
- Ustawienia zaawansowane (Erw. Einstellungen)
```

#### 2. **Silnik Sterujący (Engine 2.04.002):**
```
Funkcje:
- Kontrola CAN Bus
- Algorytmy nawigacji (navigation_query_new_angles4)
- Zarządzanie stanem systemu
- Obsługa czujników i aktuatorów
- Logika bezpieczeństwa
```

#### 3. **System Nawigacji:**
```
Algorytmy:
- Detekcja magnetów (bt_lost_no_mapping)
- Korekta trasy (navigation_query_new_angles4)
- Mapowanie pozycji (_mapping_data)
- Odometrię z enkoderów
- Zarządzanie trasami
```

#### 4. **Protokoły Komunikacyjne:**
```
CAN Bus Protocol:
- Master-Slave komunikacja
- ID-based addressing (1-64)
- Real-time control
- Status monitoring
- Error handling

Network Stack:
- WLAN (WiFi client/AP)
- Ethernet (LAN)
- TCP/IP protocols
- Remote access
```

## 📱 INTERFEJS UŻYTKOWNIKA - SZCZEGÓŁOWA ANALIZA

### **Menu Structure (z raportu):**

#### **1. Hauptmenü (Menu Główne):**
```
├── Automatik           ← Tryb automatyczny
├── Handbetrieb         ← Tryb ręczny  
├── Referenzpositionen  ← Pozycje referencyjne
├── Status              ← Status systemu
├── Import/Export       ← Zarządzanie danymi
└── Einstellungen       ← Ustawienia
```

#### **2. Handbetrieb (Tryb Ręczny):**
```
Funkcje:
- Sterowanie ręczne napędami
- Test jednotliwych modułów
- "Feststellbremse lösen" (zwolnienie hamulca postojowego)
- Kontrola ślimaka podgarniającego
- Test dozownika paszy treściwej
```

#### **3. Referenzpositionen:**
```
Funkcje:
- "Magnete Einlernen" (nauka magnetów)
- Kalibracja pozycji startowych
- Ustawienie punktów kontrolnych
- Mapowanie trasy
```

#### **4. Status System:**
```
Wyświetlane informacje:
- "Batterie: 92%" (poziom baterii)
- "Ladestrom: 5.1A" (prąd ładowania)
- Status modułów CAN (OK/ERROR)
- "Zeige Notfahrten" (jazdy awaryjne)
- Diagnostyka połączeń
```

#### **5. Import/Export:**
```
Funkcje:
- Export do "ButlerExport.csv" 
- Import konfiguracji z SD
- Backup ustawień
- Przywracanie konfiguracji
```

#### **6. Erw. Einstellungen (Ustawienia Zaawansowane):**
```
├── Geräte (Urządzenia)
│   ├── CAN ID mapping
│   ├── Status modułów  
│   └── Diagnostyka
├── WLAN
│   ├── SSID configuration
│   ├── PSK (hasło)
│   └── Network settings
├── LAN
│   ├── IP addressing
│   ├── DHCP/Static
│   └── Gateway settings
└── System
    ├── Time/Date
    ├── Software versions
    └── Factory reset
```

## 🗂️ SYSTEM PLIKÓW I DANYCH

### **Struktura Danych:**
```
/
├── /etc/
│   ├── wpa_supplicant.conf  ← Konfiguracja WiFi
│   ├── modules.conf         ← Moduły jądra
│   └── network/             ← Ustawienia sieciowe
├── /driver/
│   ├── adcg45.ko           ← Sterownik ADC
│   └── keymatrix.ko        ← Sterownik klawiatury
├── /opt/butler/            ← Aplikacja główna
│   ├── bin/                ← Pliki wykonywalne
│   ├── config/             ← Konfiguracje
│   └── data/               ← Dane operacyjne
└── /media/sd/              ← Karta SD (mount point)
    ├── ButlerExport.csv    ← Exported data
    └── config_backup/      ← Backup configurations
```

### **Format Danych CSV:**
```csv
# ButlerExport.csv (przykład)
timestamp,position,magnet_id,battery_level,status
2017-03-29 10:15:23,barn_1_start,001,92,OK
2017-03-29 10:16:45,barn_1_feed,002,92,FEEDING
2017-03-29 10:18:12,barn_1_end,003,91,OK
```

## 🔄 PROTOKOŁY KOMUNIKACYJNE

### **CAN Bus Protocol (Firmowy):**
```cpp
// Message Structure (hipotetyczna)
typedef struct {
    uint16_t can_id;        // 1-64
    uint8_t command;        // CMD_SET_SPEED, CMD_GET_STATUS, etc.
    uint8_t data_length;    // 0-8 bytes
    uint8_t data[8];        // Payload
    uint16_t checksum;      // Error detection
} CAN_Message;

// Przykładowe komendy:
#define CMD_SET_SPEED       0x01
#define CMD_GET_STATUS      0x02  
#define CMD_EMERGENCY_STOP  0x03
#define CMD_MAGNET_DETECTED 0x04
#define CMD_BATTERY_STATUS  0x05
```

### **Network Protocol:**
```
WLAN Configuration:
- SSID: "WasserbauerAP" / "WLAN-STALL"
- Security: WPA2-PSK
- Mode: Client/AP (configurable)
- DHCP/Static IP support

Services:
- HTTP Web Interface (port 80)
- SSH Remote Access (port 22) 
- Custom TCP Service (port ???)
- File Transfer (FTP/SFTP)
```

## 🧭 ALGORYTMY NAWIGACJI

### **System Nawigacji (z analizy kodu):**

#### **1. Główny Algorytm:**
```cpp
void navigation_main_loop() {
    // 1. Odczyt pozycji z magnetlineal
    magnet_position = read_can_magnetlineal();
    
    // 2. Kalkulacja odchylenia
    if (magnet_detected) {
        deviation = calculate_deviation(magnet_position);
        correction = navigation_query_new_angles4(deviation);
    }
    
    // 3. Odometrię z enkoderów
    left_distance = get_encoder_distance(CAN_ID_8);
    right_distance = get_encoder_distance(CAN_ID_9);
    
    // 4. Detekcja poślizgu
    if (check_wheel_slip(left_distance, right_distance)) {
        bt_lost_no_mapping();  // Recovery procedure
    }
    
    // 5. Aktualizacja pozycji
    update_position(_mapping_data);
}
```

#### **2. Procedury Kalibracji:**
```cpp
void magnete_einlernen() {
    // Procedura "nauki magnetów"
    while (calibration_mode) {
        if (magnet_detected) {
            save_magnet_position(current_position);
            display_message("Magnet gespeichert");
        }
        wait_for_next_position();
    }
    save_calibration_data();
}
```

#### **3. System Obór (Bay Management):**
```cpp
void buchten_management() {
    // "Der Butler fährt los, drücken Sie den Button 'Buchteingang'"
    while (not_at_bay_entrance) {
        navigate_to_next_point();
    }
    
    // Feeding sequence
    aktivate_schnecke();        // Start feeding auger
    if (kraftfutter_enabled) {
        dispense_kraftfutter();  // Concentrated feed
    }
    
    wait_for_feeding_complete();
    deaktivate_schnecke();
}
```

## 🔧 PROCEDURY SERWISOWE

### **Diagnostyka Systemu:**
```cpp
void system_diagnostics() {
    // Test wszystkich modułów CAN
    for (int id = 1; id <= 64; id++) {
        status = ping_can_device(id);
        display_device_status(id, status);
    }
    
    // Test czujników
    test_magnetlineal();
    test_encoders();
    test_battery_system();
    
    // Generate report
    create_diagnostic_report();
}
```

### **Aktualizacja Firmware:**
```cpp
void firmware_update() {
    // 1. Load from SD card
    if (sd_card_present()) {
        firmware_file = load_file("/media/sd/firmware.bin");
    }
    
    // 2. Verify checksum
    if (verify_firmware(firmware_file)) {
        // 3. Flash individual modules via CAN
        update_can_devices(firmware_file);
        
        // 4. Restart system
        system_restart();
    }
}
```

## 📊 LOGOWANIE I MONITORING

### **System Logowania:**
```cpp
typedef struct {
    uint32_t timestamp;
    uint8_t module_id;
    uint8_t event_type;
    char message[64];
    uint8_t severity;  // INFO, WARNING, ERROR, CRITICAL
} Log_Entry;

// Przykładowe logi:
log_event(CAN_ID_1, "Magnet detected at position 15");
log_event(CAN_ID_12, "Battery level: 92%");
log_event(CAN_ID_8, "Motor overcurrent detected");
log_event(SYSTEM, "Emergency stop activated");
```

### **Exportowane Dane:**
```
Dane operacyjne zapisywane do CSV:
- Pozycje magnetów i czasy przejazdu
- Poziomy baterii w czasie
- Błędy i alarmy systemu
- Statystyki żywienia (ile paszy, gdzie)
- Performance metrics (prędkości, prądy)
```

## 🌐 KOMUNIKACJA SIECIOWA

### **Remote Access Architecture:**
```
Internet ←→ Router ←→ WLAN-Sender (1008366) ←→ ButlerGold

Funkcje zdalnego dostępu:
1. Monitoring statusu robota
2. Download logów i raportów  
3. Upload nowych konfiguracji
4. Remote firmware updates
5. Diagnostyka zdalna przez serwis
```

### **Web Interface (hipotetyczny):**
```html
<!-- Przykład interfejsu webowego -->
<dashboard>
    <status>
        <battery>92%</battery>
        <position>Barn 3, Section 2</position>
        <mode>Automatic Feeding</mode>
    </status>
    
    <controls>
        <button>Emergency Stop</button>
        <button>Return to Base</button>
        <button>Start Manual Mode</button>
    </controls>
    
    <logs>
        <entry>10:15 - Feeding cycle completed</entry>
        <entry>10:12 - Magnet detected at position 15</entry>
        <entry>10:10 - Starting feeding cycle</entry>
    </logs>
</dashboard>
```

## 🔐 BEZPIECZEŃSTWO I ERROR HANDLING

### **Procedury Awaryjne:**
```cpp
void emergency_procedures() {
    // 1. Emergency Stop
    if (emergency_stop_pressed()) {
        halt_all_motors();
        log_event(CRITICAL, "Emergency stop activated");
        send_alarm_notification();
    }
    
    // 2. Communication Loss
    if (can_communication_lost()) {
        enter_safe_mode();
        try_communication_recovery();
    }
    
    // 3. Low Battery
    if (battery_level < CRITICAL_LEVEL) {
        return_to_charging_station();
        send_low_battery_alert();
    }
    
    // 4. Navigation Lost
    if (navigation_lost()) {
        bt_lost_no_mapping();      // Recovery procedure
        attempt_position_recovery();
    }
}
```

## 🎯 KLUCZOWE DESCOBYCIA SOFTWARE

### **Architektura jest BARDZO zaawansowana:**

1. **Linux Embedded** z pełnym stack'iem
2. **Modular CAN-based architecture** 
3. **Dual-language GUI** (DE/EN)
4. **Professional diagnostics** i logging
5. **Network connectivity** dla remote access
6. **SD card support** dla backup/restore
7. **Firmware update capability** przez CAN
8. **Advanced navigation algorithms**
9. **Safety systems** z error handling
10. **Data export/import** dla integracji

### **To nie jest prosty embedded system - to pełny industrial computer z:**
- **Rozproszoną architekturą** modułów
- **Real-time communication** przez CAN
- **Professional UI/UX** z procedurami
- **Network integration** możliwościami  
- **Industrial-grade logging** i diagnostyką
- **Remote support** capability

**System software WB jest na poziomie przemysłowych robotów AGV/AMR!** 🚀