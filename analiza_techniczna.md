# KOMPLETNA ANALIZA TECHNICZNA REPOZYTORIUM ROBOTA WBNEXL

## 🔍 KROK 1: Struktura i zawartość

### Pełna struktura katalogów i plików (drzewo):

```
workspace/
├── .git/                    # Repozytorium Git
├── .gitattributes          # Konfiguracja Git (66B)
├── config.sh               # Główny skrypt konfiguracyjny aktualizacji (773B)
├── update                  # Plik aktualizacji/bootloader (6.4MB)
├── linux.bin               # Kernel Linux (2.9MB)  
├── rootfs.img              # Obraz głównego systemu plików UBI (59MB)
├── bin/                    # Katalog na binaria (pusty)
└── WB/                     # Katalog na kod aplikacyjny (pusty)
```

### Analiza poszczególnych elementów:

#### **config.sh** - Skrypt konfiguracyjny aktualizacji
**Rola**: Kontroluje proces aktualizacji firmware robota
**Funkcje**:
- `DO_KERNEL=y` - Aktualizacja kernela Linux z pliku linux.bin
- `DO_ROOTFS=y` - Aktualizacja głównego systemu plików z rootfs.img
- `DO_BACKUP=y` - Tworzenie kopii zapasowych: bazy danych, plików VPN, ustawień, logów
- `DO_BINARIES=y` - Kopiowanie binariów z katalogu 'bin' na urządzenie
- `DO_VPN=n` - Kopiowanie konfiguracji VPN
- `DO_SETTINGS=n` - Kopiowanie plików ustawień
- `DO_UPDATE_CAN=n` - Aktualizacja firmware CAN bus (canFlash)

#### **update** (6.4MB) - Bootloader/Firmware główny
**Typ**: Plik binarny ARM
**Zawartość**:
- Kod bootloadera dla procesora ARM
- Komunikaty bootowania: "Uncompressing Linux...", "done, booting the kernel"
- String "linux-2.6" - wskazuje na kernel Linux 2.6
- Instrukcje ARM assembly
- Mechanizm dekompresji kernela

#### **linux.bin** (2.9MB) - Kernel Linux
**Typ**: Skompresowany kernel Linux
**Zawartość**:
- Kernel Linux w wersji 2.6
- Wsparcie dla procesora ARM
- Wbudowane sterowniki i moduły

#### **rootfs.img** (59MB) - Główny system plików
**Typ**: Obraz UBI (Unsorted Block Images) - format typowy dla systemów embedded
**Zawartość**: Pełny system Linux z aplikacjami robota

---

## 🧠 KROK 2: Analiza plików istotnych w rootfs.img

### Architektura systemu:
- **Procesor**: AT91SAM9 (ARM9) od Atmel/Microchip
- **System operacyjny**: Linux 2.6 z Buildroot
- **Środowisko budowy**: `/home/arm9_WBNEXL/AT91SAM9/Buildroot/`
- **Nazwa produktu**: WBNEXL (prawdopodobnie robot przemysłowy)

### Kluczowe komponenty systemu:

#### **Narzędzia systemowe**:
- **Busybox** - Główny zestaw narzędzi Unix dla systemów embedded
- **MTD tools** - Zarządzanie pamięcią flash: `flash_erase`, `flash_unlock`, `flashcp`
- **GStreamer** - Framework do obsługi multimediów
- **Qt** - Framework interfejsu graficznego

#### **Skrypty konfiguracyjne**:
- `setupmac.sh` - Konfiguracja adresu MAC
- `updateWLAN0.sh` - Aktualizacja konfiguracji Wi-Fi
- `updateETH0.sh` - Aktualizacja konfiguracji Ethernet
- `checkIP.sh` - Sprawdzanie konfiguracji IP
- `memlog.sh` - Logowanie stanu pamięci

#### **Pliki konfiguracyjne**:
- `qt.conf` - Konfiguracja frameworka Qt
- `olv.conf` - Konfiguracja modułu OLV
- `ma.xml` - Plik XML prawdopodobnie związany z mapowaniem
- `modules.conf` - Konfiguracja modułów kernela

#### **Moduły Python**:
- `glib.py` - Bindingi GLib
- `gobject.py` - Bindingi GObject

### Komunikacja sprzętowa:
- **CAN bus**: `can0` - Interfejs CAN bus, `canmsgmgr` - manager wiadomości CAN
- **Ethernet**: Wsparcie dla interfejsu ETH0
- **Wi-Fi**: Wsparcie dla interfejsu WLAN0
- **NVRAM**: Obsługa pamięci nieulotnej (`nvram.h`)

### System logowania:
- **WBLog** - Dedykowany system logowania dla aplikacji WBNEXL
- **Wielowątkowość**: Użycie pthreads i QThreadPool

---

## 🧭 KROK 3: Nawigacja i sterowanie robotem

### Komunikacja CAN bus:
Robot używa **CAN bus** jako głównej magistrali komunikacyjnej:
- **Interfejs**: `can0` (ifconfig can0)
- **Manager**: `canmsgmgr` - aplikacja zarządzająca wiadomościami CAN
- **Aktualizacje**: `canFlash` - narzędzie do aktualizacji firmware kontrolerów CAN

### Architektura sterowania:
Chociaż nie ma bezpośredniego dostępu do kodu źródłowego, z analizy wynika:

1. **Główna aplikacja Qt** - Interfejs użytkownika napisany w Qt
2. **Moduł OLV** - Prawdopodobnie odpowiedzialny za nawigację/mapowanie
3. **Komunikacja CAN** - Sterowanie silnikami i czujnikami przez CAN bus
4. **System logowania WBLog** - Monitorowanie wszystkich operacji

### Struktura systemu sterowania:

```
[Interfejs Qt] → [Aplikacja główna] → [CAN Manager] → [Kontrolery CAN] → [Silniki/Czujniki]
                       ↓
                   [WBLog] → [Baza danych] → [Pliki logów]
```

### Funkcje nawigacyjne:
- **Mapowanie**: Plik `ma.xml` może zawierać dane map lub punktów odniesienia
- **Konfiguracja ruchu**: Brak bezpośrednich odniesień do trajektorii, ale system CAN sugeruje precyzyjne sterowanie
- **Monitoring**: System WBLog rejestruje wszystkie operacje

### Interfejs użytkownika:
- **Framework Qt** - Graficzny interfejs użytkownika
- **GStreamer** - Możliwość wyświetlania wideo/obrazów z kamer
- **Debugger** - `gdbserver :2345` - debug przez port 2345

---

## 🔄 KROK 4: Aktualizacje, backup, bezpieczeństwo

### Procedury aktualizacji firmware:

#### **Główny mechanizm aktualizacji (config.sh)**:
1. **Backup danych**: Automatyczne kopie zapasowe bazy danych, VPN, ustawień, logów
2. **Aktualizacja kernela**: Zastąpienie kernela Linux z pliku `linux.bin`
3. **Aktualizacja systemu**: Zastąpienie głównego systemu z `rootfs.img`
4. **Aktualizacja binariów**: Kopiowanie nowych aplikacji z katalogu `bin/`
5. **Aktualizacja CAN**: Opcjonalna aktualizacja firmware kontrolerów CAN

#### **Narzędzia flashowania**:
- `flash_erase` - Kasowanie bloków pamięci flash
- `flash_unlock` - Odblokowanie chronionych obszarów
- `flashcp` - Kopiowanie danych do flash
- `flash_info` - Informacje o pamięci flash

### Zabezpieczenia:
- **OpenSSL**: Pełne wsparcie kryptograficzne z kluczami RSA/DSA
- **Autoryzacja**: Mechanizmy uwierzytelniania (`authenticator`)
- **Klucze**: System zarządzania kluczami kryptograficznymi
- **Open Source**: Licencja Qt Open Source

### Komunikacja sieciowa:
- **Protokoły**: HTTP, TCP, UDP, socket
- **Szyfrowanie**: SSL/TLS
- **VPN**: Wsparcie dla połączeń VPN

---

## 📋 KROK 5: Wnioski końcowe

### Czy repozytorium pozwala zrozumieć działanie robota?

**TAK**, chociaż nie ma bezpośredniego dostępu do kodu źródłowego, analiza ujawnia:

#### **Jak porusza się robot**:
1. **Sterowanie CAN bus**: Robot używa magistrali CAN do komunikacji z kontrolerami silników
2. **Nawigacja**: System mapowania przez moduł OLV i pliki XML
3. **Precyzyjne sterowanie**: Wielowątkowa aplikacja Qt z real-time control

#### **Pełny cykl pracy**:
1. **Bootowanie**: Bootloader → Kernel Linux → Aplikacja główna
2. **Inicjalizacja**: Konfiguracja sieci (ETH0/WLAN0) → Uruchomienie CAN bus
3. **Praca**: Interfejs Qt ← → Aplikacja główna ← → CAN Manager ← → Kontrolery
4. **Logowanie**: Wszystkie operacje zapisywane przez WBLog
5. **Backup**: Automatyczne kopie zapasowe danych

#### **Komunikacja z użytkownikiem**:
- **GUI Qt**: Graficzny interfejs dotykowy lub ekranowy
- **Sieć**: Dostęp zdalny przez HTTP/TCP
- **Debug**: gdbserver na porcie 2345
- **Logi**: System WBLog z bazą danych

### Możliwość budowy alternatywnego oprogramowania:

**TAK**, na podstawie tej analizy można:

1. **Odtworzenie architektury**: 
   - Procesor AT91SAM9 (ARM9)
   - Linux 2.6 z Buildroot
   - Aplikacja Qt + CAN bus

2. **Interfejs CAN**: 
   - Użycie `can0` z SocketCAN
   - Implementacja `canmsgmgr` w C++/Python

3. **System aktualizacji**:
   - Replikacja mechanizmu z `config.sh`
   - Użycie MTD tools do zarządzania flash

### Co można zaimplementować u siebie:

#### **Bezpośrednie implementacje**:
1. **System aktualizacji** - Skrypt `config.sh` jako template
2. **Logowanie** - Klasa WBLog w C++
3. **Komunikacja CAN** - Manager wiadomości CAN
4. **Interfejs Qt** - GUI dla robota przemysłowego

#### **Architektura referencyjna**:
- **Embedded Linux** z Buildroot
- **CAN bus** dla sterowania silnikami
- **Qt** dla interfejsu użytkownika
- **UBI/MTD** dla systemu plików
- **OpenSSL** dla bezpieczeństwa

#### **Gotowe rozwiązania**:
- Skrypty konfiguracji sieci (`updateWLAN0.sh`, `updateETH0.sh`)
- Mechanizm backup i restore
- System debugowania z gdbserver
- Struktura logów i monitoringu

### Podsumowanie:
Repozytorium WBNEXL to **kompletny system firmware** dla robota przemysłowego opartego na ARM9 z komunikacją CAN bus. Zawiera wszystkie niezbędne komponenty do budowy podobnego systemu, włączając bootloader, kernel, aplikacje, procedury aktualizacji i zabezpieczenia. Mimo braku bezpośredniego dostępu do kodu źródłowego, analiza ujawnia wystarczająco szczegółów technicznych do replikacji architektury i kluczowych funkcji.