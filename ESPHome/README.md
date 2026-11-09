# OpenReku — ESPHome

Konfiguracja ESPHome sterownika rekuperatora: ESP32 z frameworkiem ESP-IDF, integracja z Home Assistant przez szyfrowane API, sterowanie wentylatorami i zaworami, odczyty czujników przez multiplekser I²C oraz procedura testu filtrów.

## Pliki

| Plik | Przeznaczenie |
| --- | --- |
| [rekuperator.yaml](rekuperator.yaml) | Kod konfiguracji ESPHome, obejmujący komponenty urządzenia, automatyzacje i fragmenty C++ w lambdach. |
| [secrets.yaml.example](secrets.yaml.example) | Wzór wymaganych danych dostępowych. Zawiera wyłącznie pola do uzupełnienia. |
| `secrets.yaml` | Lokalny plik z własnymi danymi, tworzony na podstawie wzoru. |

## Przygotowanie

1. Umieść `rekuperator.yaml` w katalogu konfiguracji ESPHome.
2. Skopiuj `secrets.yaml.example` jako `secrets.yaml` do tego samego katalogu. Jeśli masz już plik sekretów, dodaj do niego wymagane wpisy zamiast go nadpisywać.
3. Uzupełnij nazwę i hasło sieci Wi-Fi, klucz szyfrowania API oraz hasło awaryjnego punktu dostępowego. Wszystkie cztery odwołania `!secret` muszą mieć odpowiadający wpis w lokalnym pliku sekretów.
4. Klucz API musi zawierać 32 losowe bajty zakodowane w Base64. Można wygenerować go lokalnie poleceniem `openssl rand -base64 32`. Hasło awaryjnego AP ustaw na 8–63 znaki. Wartości `UZUPELNIJ_...` z pliku wzorcowego trzeba zastąpić własnymi danymi.
5. Sprawdź konfigurację w swoim środowisku ESPHome, np. poleceniem `esphome config rekuperator.yaml`, przed kompilacją i wgraniem.

Nazwy wpisów w `secrets.yaml`:

- `wifi_ssid` — nazwa sieci Wi-Fi;
- `wifi_password` — hasło sieci Wi-Fi;
- `rekuperator_api_encryption_key` — klucz szyfrowania API ESPHome/Home Assistant;
- `rekuperator_fallback_ap_password` — hasło awaryjnego punktu dostępowego urządzenia.

Parametry w `substitutions`, przypisania GPIO, kanały I²C i kalibracje odpowiadają instalacji autora; przed użyciem z własnym urządzeniem należy je porównać z jego połączeniami. Konfiguracja pomija kanał 2 multipleksera ze względu na uszkodzony styk w tej instalacji.

## Punkty pomiarowe

- **Czerpnia** — powietrze z zewnątrz, przed wymiennikiem.
- **Nawiew** — powietrze kierowane do domu, za wymiennikiem.
- **Wywiew** — powietrze wracające z domu, przed wymiennikiem.
- **Wyrzutnia** — powietrze usuwane na zewnątrz, za wymiennikiem.

W nazwach pomiarów wentylatorów i przepływu **Wlot** oznacza tor nawiewny (`fan2`), a **Wylot** tor wywiewny (`fan1`). Identyfikatory poniżej są ID ESPHome; identyfikatory encji w HA mogą mieć dodatkowy prefiks.

## Sensory

| Grupa | Nazwy / ID ESPHome | Co przedstawia |
| --- | --- | --- |
| Temperatura | „Czerpnia / Nawiew / Wywiew / Wyrzutnia - Temperatura”; `czerpnia_temp`, `nawiew_temp`, `wywiew_temp`, `wyrzutnia_temp` | Temperatura powietrza w °C z czterech czujników SHT4x. Odczyt co 30 s. |
| Wilgotność względna | „Czerpnia / Nawiew / Wywiew / Wyrzutnia - Wilgotność”; `czerpnia_hum`, `nawiew_hum`, `wywiew_hum`, `wyrzutnia_hum` | Wilgotność względna w %, mierzona przez SHT4x co 30 s. |
| Zawartość wilgoci | „Czerpnia / Nawiew / Wywiew / Wyrzutnia - Wilgotnosc bezwzględna”; `czerpnia_hum_abs`, `nawiew_hum_abs`, `wywiew_hum_abs`, `wyrzutnia_hum_abs` | Gramy pary wodnej na kilogram suchego powietrza (g/kg), obliczane co 30 s z temperatury i wilgotności względnej. Mimo nazwy encji jest to stosunek mas, nie g/m³. Obliczenia przyjmują ciśnienie `air_pressure_pa`, domyślnie 101325 Pa. |
| Ciśnienie różnicowe | „Wlot / Wylot Venturi Differential Pressure”; `wlot_venturi_dp`, `wylot_venturi_dp` | Różnica ciśnień na zwężkach Venturiego z SDP810, w YAML w hPa. Odczyt co 2 s; małe wartości w martwej strefie są zerowane. Podstawa obliczeń przepływu. |
| Gęstość powietrza | „Wlot / Wylot - gęstość powietrza”; `wlot_air_density`, `wylot_air_density` | Gęstość w kg/m³, wyliczana co 30 s z temperatury nawiewu lub wywiewu oraz przyjętego ciśnienia atmosferycznego. |
| Prędkość powietrza | „Wlot / Wylot - prędkość powietrza”; `wlot_air_velocity`, `wylot_air_velocity` | Obliczona średnia prędkość w kanale w m/s. Wykorzystuje różnicę ciśnień, gęstość powietrza i geometrię zwężki; aktualizacja co 2 s. |
| Przepływ powietrza | „Wlot / Wylot - przepływ powietrza”; `wlot_airflow`, `wylot_airflow` | Strumień objętości w m³/h, obliczany z prędkości i przekroju kanału co 2 s. Filtr medianowy i średnia wykładnicza wygładzają wskazania. Te wartości służą też synchronizacji przepływów. |
| Obroty wentylatorów | „Wlot RPM” (`fan2_rpm`), „Wylot RPM” (`fan1_rpm`) | Prędkość obrotowa z impulsów tachometrycznych, w obr./min; odczyt co 5 s. |
| Wyniki testu przepływu | „Wlot / Wylot - przepływ na 1% PWM (test)”; `filter_test_wlot_flow_per_pwm`, `filter_test_wylot_flow_per_pwm` | Średni przepływ podczas testu podzielony przez nastawę PWM w punktach procentowych, w m³/h/%. Wynik służy do porównywania kolejnych testów przy tej samej nastawie. |
| Wyniki testu obrotów | „Wlot / Wylot - RPM na 1% PWM (test)”; `filter_test_wlot_rpm_per_pwm`, `filter_test_wylot_rpm_per_pwm` | Średnie obroty podczas testu podzielone przez nastawę PWM, w rpm/%. Pomagają porównać zachowanie obu wentylatorów i zmiany w czasie. |
| Tryb serwisowy | „Tryb serwisowy”; `service_mode` (sensor binarny) | Informuje, że trwa procedura serwisowa i sterowanie użytkownika jest zablokowane. Jest tylko odczytem, nie przełącznikiem. Restart ESP zeruje ten stan. |

Cztery wyniki testowe i tryb serwisowy mają w HA kategorię diagnostyczną. Wyniki są publikowane po zakończeniu testu, zapisywane w pamięci flash i odtwarzane po restarcie. Przed pierwszym testem nie mają wartości.

Statystyki długoterminowe są włączone dla temperatur, wilgotności względnej i zawartości wilgoci, ciśnień różnicowych, przepływów, RPM oraz czterech wyników testowych. Gęstość i prędkość powietrza nie mają obecnie `state_class: measurement`.

## Przełączniki i sterowanie

| Element w HA | ID ESPHome | Działanie |
| --- | --- | --- |
| Zasilanie rekuperatora | `recuperator_power` | Włącza zasilanie podzespołów i zleca zamknięcie bypassu. Przy wyłączeniu zatrzymuje wentylatory, wyłącza przekaźniki siłownika, a następnie odcina zasilanie podzespołów. Sam ESP pozostaje dostępny. |
| Wlot PWM | `fan2_pwm` | Włączanie/wyłączanie wentylatora nawiewnego i nastawa jego prędkości. Przy synchronizacji jest wentylatorem nadrzędnym. |
| Wylot PWM | `fan1_pwm` | Włączanie/wyłączanie wentylatora wywiewnego i nastawa jego prędkości. Polecenia użytkownika są ignorowane, gdy wybrano synchronizację PWM lub przepływu. |
| Synchronizacja wentylatorów | `fan_sync_mode` | **Brak** — niezależne sterowanie; **Moc PWM** — wylot naśladuje stan i nastawę wlotu; **Przepływ powietrza** — regulator dobiera PWM wylotu do przepływu wlotu z uwzględnieniem korekty bilansu. |
| Korekta bilansu Wlot-Wylot | `airflow_balance_offset` | Korekta w m³/h dla synchronizacji przepływu. Cel dla wylotu to przepływ wlotu minus korekta, więc dodatnia wartość zmniejsza zadany wywiew. Zakres −50…420 m³/h, krok 5 m³/h. |
| By-pass | `bypass` | Ręczne polecenie otwarcia lub zamknięcia obejścia wymiennika. Stan odzwierciedla zadane położenie, bez czujnika potwierdzającego rzeczywistą pozycję siłownika. W trybie automatycznym regulator może później zmienić ręcznie zadaną pozycję. |
| Tryb sterowania bypassem | `bypass_control_mode` | **Ręczne sterowanie** — bez automatycznych zmian; **Automatyczne grzanie** — otwarcie, gdy czerpnia jest cieplejsza od wywiewu; **Automatyczne chłodzenie** — otwarcie, gdy czerpnia jest chłodniejsza od wywiewu. Obowiązują opisane poniżej próg i zwłoka. Po restarcie wraca tryb ręczny. |
| Restart rekuperatora | `restart_rekuperatora` | Przycisk restartu ESP, dostępny także przy wyłączonym zasilaniu podzespołów i podczas serwisu. W HA znajduje się w kategorii konfiguracji. |

Przy wyłączonym rekuperatorze polecenia sterowania są ignorowane poza włączeniem zasilania i restartem ESP. Podczas serwisu zablokowane jest również sterowanie zasilaniem; restart ESP pozostaje dostępny.

### Automatyka bypassu

Regulator co 5 s porównuje temperatury czerpni i wywiewu. Parametry są w `globals` i nie są wystawione do HA:

- `bypass_temperature_threshold` — próg różnicy temperatur, domyślnie 0,5°C;
- `bypass_hold_seconds` — wymagany czas utrzymania warunku, domyślnie 300 s.

Przy grzaniu różnica czerpnia minus wywiew ≥0,5°C prowadzi do otwarcia, a ≤−0,5°C do zamknięcia. Przy chłodzeniu kierunki są odwrotne. Pomiędzy progami pozycja nie zmienia się. Warunek musi trwać nieprzerwanie przez 5 minut; jego zanik zeruje odliczanie. Automatyka jest wstrzymana przy wyłączonym rekuperatorze, w serwisie i przy nieprawidłowych odczytach temperatury.

### Elementy wewnętrzne

Encje z `internal: true` realizują polecenia i nie są wystawiane do HA:

| Grupa / ID | Rola |
| --- | --- |
| `internal_recuperator_main_relay` | Fizyczny przekaźnik zasilania podzespołów. |
| `internal_recuperator_power` | Sekwencja włączania i wyłączania podzespołów, wywoływana przez publiczny przełącznik zasilania. |
| `belimo_open_relay`, `belimo_close_relay` | Przekaźniki obu kierunków ruchu bypassu z blokadą jednoczesnego włączenia. Czas sterowania wynosi domyślnie 88 s (`belimo_pulse_time`). |
| `internal_fan1_pwm`, `internal_fan2_pwm` | Właściwe sterowanie wentylatorami, używane przez publiczne encje, synchronizację i test serwisowy. |
| `internal_fan_sync_mode`, `internal_airflow_balance_offset` | Wewnętrzny wybór synchronizacji oraz nastawa korekty bilansu. |
| `internal_bypass` | Sterowanie ruchem bypassu używane przez publiczną encję, automatykę i test. |

## Test serwisowy

Akcja API `run_filter_test` jest dostępna w HA jako `esphome.rekuperator_run_filter_test`. Harmonogram ustawia się w HA; ESPHome nie uruchamia testu samodzielnie o określonej godzinie.

Procedura zapamiętuje ustawienia wentylatorów, synchronizacji i bypassu, włącza serwis, wyłącza synchronizację i uruchamia oba wentylatory z `filter_test_pwm` (domyślnie 30%). Wymusza zamknięcie bypassu, czeka na jego ruch z marginesem 2 s, a następnie przez 4 minuty na stabilizację. Zbiera 12 odczytów co 5 s, wylicza cztery wyniki, zapisuje je i przywraca poprzednie ustawienia. Nie czeka na zakończenie ruchu przywracanego bypassu przed wyjściem z serwisu.

Test nie ruszy przy wyłączonym zasilaniu, aktywnym serwisie lub nastawie testowej PWM poza zakresem 1–100. Ponowne wywołanie podczas trwającego testu nie restartuje procedury. Próbki nie są odrzucane; nieprawidłowe odczyty mogą dać nieprawidłowy wynik. Współczynniki odnoszą się do procentowej nastawy wentylatora, nie do surowego wypełnienia wyjścia LEDC.
