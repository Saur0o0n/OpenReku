<picture>
  <source media="(prefers-color-scheme: dark)" srcset=".github/images/openreku-logo-white.svg">
  <img src=".github/images/openreku-logo-color.svg" alt="OpenReku" width="480">
</picture>

Otwarty projekt sterownika rekuperatora opartego na ESP32 i ESPHome, współpracującego z Home Assistant. Obejmuje elektronikę sterownika oraz konfigurację obsługującą wentylatory, zawory i czujniki.

![OpenReku — sterowanie rekuperatorem w Home Assistant](HomeAssistant/karta-rekuperatora.png)

## Zawartość repozytorium

| Katalog | Zawartość |
| --- | --- |
| [PCB/](PCB/) | Projekt płytki w wersji 1.1: schematy i PCB KiCad, lokalne biblioteki, lista elementów BOM, Gerbery do produkcji. |
| [ESPHome/](ESPHome/) | Konfiguracja `rekuperator.yaml` z obsługą urządzenia i integracją z Home Assistant oraz wzór `secrets.yaml.example` do uzupełnienia własnymi danymi dostępowymi. |
| [HomeAssistant/](HomeAssistant/) | Karta panelu Home Assistant, automatyzacje testu filtrów i korekty bilansu oraz podkład graficzny karty. |

Opis plików i instrukcję otwarcia projektu elektroniki zawiera [PCB/README.md](PCB/README.md). Przygotowanie konfiguracji i pliku sekretów opisano w [ESPHome/README.md](ESPHome/README.md). Sposób dodania karty do panelu opisano w [HomeAssistant/README.md](HomeAssistant/README.md).

## Licencja

Projekt PCB, dokumentacja i własne grafiki są udostępnione na licencji [CC BY-NC 4.0](LICENSES/CC-BY-NC-4.0.txt), a kod i konfiguracje YAML ESPHome oraz Home Assistant — na [PolyForm Noncommercial 1.0.0](LICENSES/PolyForm-Noncommercial-1.0.0.txt).

Możesz je kopiować, modyfikować i udostępniać do celów niekomercyjnych, na warunkach odpowiedniej licencji. Przy udostępnianiu zachowaj informację o autorze **Saur0o0n**, nazwie **OpenReku**, [źródle projektu](https://github.com/Saur0o0n/OpenReku) i licencji; dla materiałów CC BY-NC oznacz również wprowadzone zmiany. Kod zawiera obowiązkową informację `Required Notice:`.

Zakres licencji i wyłączenia dotyczące cudzych materiałów określa [LICENSE](LICENSE). Wykorzystanie komercyjne wykraczające poza udzielone licencje wymaga osobnej zgody właścicieli praw.
