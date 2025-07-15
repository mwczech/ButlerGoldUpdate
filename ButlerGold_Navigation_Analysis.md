# Analiza Systemu Nawigacji Robota ButlerGold

## 1. Pliki i Katalogi Związane z Nawigacją

### Pliki Systemowe Główne:
- `rootfs.img` - Obraz systemu plików zawierający kod nawigacyjny (59MB)
- `linux.bin` - Obraz jądra systemu (2.8MB)
- `update` - Plik aktualizacji systemu (6.3MB)
- `config.sh` - Skrypt konfiguracyjny systemu

### Pliki Konfiguracyjne:
- `/etc/wpa_supplicant.conf` - Konfiguracja sieci WiFi
- `qt.conf` - Konfiguracja środowiska Qt
- `/etc/modules.conf` - Konfiguracja modułów jądra
- `sample_map.csv` - Przykładowa mapa danych nawigacyjnych

### Pliki Związane z Trasami i Mapowaniem:
- `qpainterpath.cpl` - Biblioteka rysowania ścieżek
- `xmapfilter.cpg` - Filtr map/danych mapowych
- `sample_map.csv` - Przykładowa mapa z danymi nawigacyjnymi

## 2. Struktura Typowego Pliku Trasy

### Przykład pliku `sample_map.csv`:
```csv
# Przykładowa struktura pliku mapy/trasy
# Format: x, y, type, description
0, 0, "start", "Punkt startowy"
100, 0, "waypoint", "Punkt pośredni 1"
200, 100, "waypoint", "Punkt pośredni 2"
300, 200, "end", "Punkt końcowy"
```

### Formaty Danych:
- **CSV** - Główny format dla map i tras
- **XML** - Pliki konfiguracyjne (.xml)
- **CONF** - Pliki konfiguracyjne (.conf)

## 3. Logika Nawigacji - Kluczowe Funkcje

### Funkcje Nawigacyjne:
```cpp
// Kluczowe funkcje nawigacyjne znalezione w systemie:
navigation_query_new_angles4()     // Obliczanie nowych kątów nawigacji
bt_lost_no_mapping()               // Obsługa utraty mapowania
_mapping_data()                    // Dane mapowania
Y_MAPPING_free()                   // Zwalnianie danych mapowania
```

### Funkcje Ścieżek i Pozycjonowania:
```cpp
path_get()                         // Pobieranie ścieżki
path_get_dirname()                 // Pobieranie katalogu ścieżki
path_get_basename()                // Pobieranie nazwy pliku ścieżki
setPosition()                      // Ustawianie pozycji
```

### Funkcje Ruchu:
```cpp
move()                             // Podstawowa funkcja ruchu
moveGroup()                        // Ruch grupowy
moveToThread()                     // Przenoszenie do wątku
```

## 4. Kluczowe Algorytmy i Mechanizmy

### Algorytmy Nawigacyjne:
1. **System Mapowania**: 
   - Wykorzystuje struktury danych `_mapping_data`
   - Obsługuje utratę mapowania przez `bt_lost_no_mapping()`
   - Zwalnia pamięć przez `Y_MAPPING_free()`

2. **Obliczanie Kątów**:
   - Funkcja `navigation_query_new_angles4()` - oblicza nowe kąty nawigacji
   - Prawdopodobnie implementuje algorytm rotacji w 4 kierunkach

3. **Zarządzanie Ścieżkami**:
   - System plików ścieżek z funkcjami `path_get_*`
   - Obsługa błędów przez `g_path_get_dirname()`

### Mechanizmy Sterowania:
```cpp
// Kluczowe mechanizmy do sklonowania:
1. navigation_query_new_angles4() - Algorytm kalkulacji kątów
2. _mapping_data struktura - System przechowywania map
3. bt_lost_no_mapping() - Mechanizm odzyskiwania po utracie
4. Funkcje path_get_* - System zarządzania ścieżkami
```

## 5. Architektura Systemu

### Komponenty Główne:
- **Moduł Nawigacji**: `navigation_query_new_angles4`
- **Moduł Mapowania**: `_mapping_data`, `bt_lost_no_mapping`
- **Moduł Ścieżek**: `path_get_*` funkcje
- **Moduł Pozycjonowania**: `setPosition`, funkcje pozycjonujące

### Komunikacja:
- **Routing**: Wykorzystuje `iproute2` dla komunikacji sieciowej
- **Konfiguracja**: Pliki `.conf` i `.xml`
- **Błędy**: System obsługi błędów routing ("Unspecified source routeing error")

## 6. Kluczowe Mechanizmy do Sklonowania

### Priorytet 1 - Krytyczne:
1. **`navigation_query_new_angles4()`** - Podstawowy algorytm nawigacji
2. **`_mapping_data` struktura** - System przechowywania map
3. **`bt_lost_no_mapping()`** - Mechanizm odzyskiwania nawigacji

### Priorytet 2 - Ważne:
1. **System plików CSV** - Format `sample_map.csv`
2. **Funkcje `path_get_*`** - Zarządzanie ścieżkami
3. **Funkcje `move*()`** - Sterowanie ruchem

### Priorytet 3 - Pomocnicze:
1. **Filtr map** - `xmapfilter.cpg`
2. **Rysowanie ścieżek** - `qpainterpath.cpl`
3. **Konfiguracja Qt** - `qt.conf`

## 7. Architektura Techniczna

### Środowisko:
- **OS**: Linux (embedded) z systemem UBI
- **Framework**: Qt (interfejs użytkownika)
- **Język**: C/C++ (głównie C++)
- **Rozmiar**: ~68MB całkowitego obrazu systemu

### Struktury Danych:
```cpp
// Prawdopodobne struktury danych:
struct mapping_data {
    // Dane mapowania
    coordinates_t* waypoints;
    int num_waypoints;
    // ... inne pola
};

struct navigation_state {
    // Stan nawigacji
    float current_angle;
    position_t current_pos;
    // ... inne pola
};
```

### Protokoły Komunikacyjne:
- **WiFi**: Konfiguracja przez `wpa_supplicant.conf`
- **Network Routing**: Wykorzystanie `iproute2`
- **Error Handling**: System obsługi błędów routingu

## 8. Wnioski

Robot ButlerGold wykorzystuje:
- **Embedded Linux** z systemem plików UBI
- **Qt Framework** dla interfejsu
- **System nawigacji oparty na kątach** (`navigation_query_new_angles4`)
- **Mapowanie CSV** dla tras i punktów
- **Mechanizm odzyskiwania** po utracie mapowania
- **Modułową architekturę** z oddzielnymi komponentami

Kluczowe jest zrozumienie algorytmu `navigation_query_new_angles4()` oraz struktury danych `_mapping_data` dla pełnego sklonowania funkcjonalności nawigacyjnej.