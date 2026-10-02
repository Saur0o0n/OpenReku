# OpenReku — ESPHome

Konfiguracja ESPHome sterownika rekuperatora: ESP32 z frameworkiem ESP-IDF, integracja z Home Assistant przez szyfrowane API, sterowanie wentylatorami i zaworami, odczyty czujników przez multiplekser I²C oraz procedura testu filtrów.

## Pliki

| Plik | Przeznaczenie |
| --- | --- |
| [rekuperator.yaml](rekuperator.yaml) | Kod konfiguracji ESPHome, obejmujący komponenty urządzenia, automatyzacje i fragmenty C++ w lambdach. |
| [secrets.yaml.example](secrets.yaml.example) | Wzór wymaganych danych dostępowych. Zawiera wyłącznie pola do uzupełnienia. |
| `secrets.yaml` | Lokalny plik z własnymi danymi, tworzony na podstawie wzoru. Jest ignorowany przez Git i nie należy go publikować. |
| [README.md](README.md) | Opis zawartości i przygotowania konfiguracji. |

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
