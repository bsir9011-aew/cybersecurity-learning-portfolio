# Lab 04 - Registry Persistence Detection

## Cel laboratorium

Celem laboratorium było wykrycie mechanizmu persistence utworzonego przez modyfikację rejestru Windows.

Analiza obejmowała:

* identyfikację procesu zmieniającego rejestr,
* analizę klucza `Run`,
* sprawdzenie ustawionej wartości,
* ustalenie, jaki plik miał być uruchamiany,
* potwierdzenie automatycznego uruchomienia payloadu po zalogowaniu użytkownika,
* odtworzenie pełnej osi czasu zdarzeń.

## Środowisko

Laboratorium zostało wykonane w interaktywnym środowisku HTML imitującym analizę logów:

* Sysmon,
* Windows Security.

Wszystkie dane były syntetyczne i zostały przygotowane wyłącznie do nauki analizy defensywnej.

## Czym jest persistence

Persistence oznacza mechanizm pozwalający atakującemu utrzymać dostęp do systemu.

Proces uruchomiony jednorazowo może zakończyć działanie po:

* wylogowaniu użytkownika,
* restarcie komputera,
* zakończeniu procesu,
* wyłączeniu systemu.

Dlatego atakujący może utworzyć mechanizm, który ponownie uruchomi payload.

W tym laboratorium wykorzystano:

```text
klucz rejestru Run
```

Program wpisany do tego klucza może zostać uruchomiony automatycznie po zalogowaniu użytkownika.

## Analizowane Event ID

| Event ID | Źródło           | Znaczenie                                |
| -------- | ---------------- | ---------------------------------------- |
| `1`      | Sysmon           | utworzenie procesu                       |
| `12`     | Sysmon           | utworzenie lub usunięcie klucza rejestru |
| `13`     | Sysmon           | ustawienie wartości rejestru             |
| `14`     | Sysmon           | zmiana nazwy klucza lub wartości         |
| `4624`   | Windows Security | udane logowanie użytkownika              |

Najważniejszym zdarzeniem w tym laboratorium był:

```text
Sysmon Event ID 13
```

Zdarzenie to pokazało ustawienie konkretnej wartości rejestru.

## Klucze Run i RunOnce

Najczęściej analizowane lokalizacje autostartu:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

### HKCU

```text
HKEY_CURRENT_USER
```

Dotyczy aktualnego użytkownika.

Program zapisany w tym miejscu zwykle uruchomi się po zalogowaniu tego użytkownika.

### HKLM

```text
HKEY_LOCAL_MACHINE
```

Dotyczy całego komputera.

Zmiana tego obszaru może wymagać wyższych uprawnień.

### Run

Program może być uruchamiany przy każdym logowaniu.

### RunOnce

Program zazwyczaj uruchamia się tylko jeden raz, a następnie wpis może zostać usunięty.

## Scenariusz

Na hoście:

```text
WS-SALES-09
```

użytkownik:

```text
pawel.nowak
```

uruchomił podejrzany plik:

```text
C:\Users\pawel.nowak\AppData\Roaming\update.exe
```

Plik uruchomił narzędzie:

```text
reg.exe
```

Następnie utworzył wpis w kluczu autostartu:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

Wartość rejestru otrzymała nazwę:

```text
WindowsUpdate
```

i wskazywała na:

```text
C:\Users\pawel.nowak\AppData\Roaming\update.exe
```

## Główny łańcuch zdarzeń

```text
powershell.exe
    ↓
update.exe
    ↓
reg.exe
    ↓
klucz CurrentVersion\Run
    ↓
wartość WindowsUpdate
    ↓
logowanie użytkownika
    ↓
automatyczny start update.exe
    ↓
cmd.exe /c whoami /groups
```

## Proces modyfikujący rejestr

Proces:

```text
reg.exe
```

jest legalnym narzędziem systemu Windows.

Może służyć do:

* odczytywania rejestru,
* dodawania kluczy,
* ustawiania wartości,
* usuwania wpisów,
* eksportowania i importowania danych rejestru.

Samo użycie `reg.exe` nie oznacza ataku.

Podejrzany był kontekst:

```text
nieznany update.exe
→ uruchamia reg.exe
→ dodaje siebie do autostartu
```

## Polecenie modyfikujące rejestr

W laboratorium użyto polecenia:

```cmd
reg.exe add HKCU\Software\Microsoft\Windows\CurrentVersion\Run /v WindowsUpdate /t REG_SZ /d C:\Users\pawel.nowak\AppData\Roaming\update.exe /f
```

## Znaczenie przełączników reg.exe

| Element  | Znaczenie                               |
| -------- | --------------------------------------- |
| `add`    | dodanie lub zmiana wpisu rejestru       |
| `/v`     | nazwa wartości                          |
| `/t`     | typ danych                              |
| `REG_SZ` | zwykła wartość tekstowa                 |
| `/d`     | dane zapisane w wartości                |
| `/f`     | wykonanie bez dodatkowego potwierdzenia |

W tym przypadku:

```text
/v WindowsUpdate
```

oznaczało nazwę wartości.

```text
/d C:\Users\pawel.nowak\AppData\Roaming\update.exe
```

wskazywało plik, który miał być uruchamiany automatycznie.

## Analiza Sysmon Event ID 12

Sysmon Event ID `12` może informować o:

* utworzeniu klucza,
* usunięciu klucza,
* otwarciu lub utworzeniu obiektu rejestru.

W laboratorium zdarzenie dotyczyło:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

Samo wystąpienie Event ID `12` nie oznacza ataku.

W systemie legalne programy często tworzą lub modyfikują klucze rejestru.

## Analiza Sysmon Event ID 13

Sysmon Event ID `13` oznacza ustawienie wartości rejestru.

W analizowanym przypadku:

```text
TargetObject:
HKCU\Software\Microsoft\Windows\CurrentVersion\Run\WindowsUpdate
```

Dane wartości:

```text
C:\Users\pawel.nowak\AppData\Roaming\update.exe
```

Proces wykonujący zmianę:

```text
C:\Windows\System32\reg.exe
```

Proces nadrzędny:

```text
C:\Users\pawel.nowak\AppData\Roaming\update.exe
```

To zdarzenie było kluczowym dowodem utworzenia mechanizmu persistence.

## Analiza Sysmon Event ID 14

Sysmon Event ID `14` może informować o zmianie nazwy:

* klucza rejestru,
* wartości rejestru.

W laboratorium zdarzenie potwierdziło nazwę wartości:

```text
WindowsUpdate
```

Nazwa próbowała wyglądać jak legalny element systemu lub aktualizacji.

Nazwa wpisu nie świadczy jednak o jego bezpieczeństwie.

Należy sprawdzić przede wszystkim:

* proces wykonujący zmianę,
* ścieżkę uruchamianego pliku,
* podpis cyfrowy,
* hash pliku,
* proces nadrzędny,
* aktywność po zalogowaniu.

## Potwierdzenie persistence

Samo utworzenie wpisu w rejestrze nie wystarcza do potwierdzenia, że mechanizm zadziałał.

W laboratorium wystąpiło:

```text
Event ID 4624
```

czyli udane logowanie użytkownika.

Kilka sekund później:

```text
explorer.exe → update.exe
```

Plik został uruchomiony z wpisu:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run\WindowsUpdate
```

Oznacza to, że mechanizm persistence rzeczywiście zadziałał.

## Oś czasu incydentu

```text
10:00:05
PowerShell uruchamia update.exe
        ↓
10:00:12
update.exe uruchamia reg.exe
        ↓
10:00:13
Sysmon 12
Utworzenie lub otwarcie klucza Run
        ↓
10:00:14
Sysmon 13
Ustawienie wartości WindowsUpdate
        ↓
C:\Users\pawel.nowak\AppData\Roaming\update.exe
        ↓
10:00:15
Sysmon 14
Potwierdzenie nazwy wartości
        ↓
10:15:00
Event ID 4624
Użytkownik loguje się do systemu
        ↓
10:15:03
explorer.exe uruchamia update.exe
        ↓
10:15:09
update.exe uruchamia cmd.exe
        ↓
cmd.exe /c whoami /groups
```

## Dalsza aktywność payloadu

Po automatycznym uruchomieniu plik wykonał:

```cmd
cmd.exe /c whoami /groups
```

Polecenie:

```text
whoami /groups
```

pokazuje grupy, do których należy aktualny użytkownik.

Może pomóc ustalić:

* czy użytkownik jest administratorem,
* do jakich grup domenowych należy,
* jakie role posiada,
* jakie zasoby może potencjalnie otworzyć.

Polecenie jest legalne, ale w tym kontekście może wskazywać na rozpoznanie uprawnień po utrzymaniu dostępu.

## Wyniki analizy

| Element                    | Wynik                         |
| -------------------------- | ----------------------------- |
| Podejrzany host            | `WS-SALES-09`                 |
| Użytkownik                 | `pawel.nowak`                 |
| Proces zmieniający rejestr | `reg.exe`                     |
| Proces nadrzędny           | `update.exe`                  |
| Klucz                      | `HKCU\...\CurrentVersion\Run` |
| Nazwa wartości             | `WindowsUpdate`               |
| Payload                    | `update.exe`                  |
| Najważniejszy Event ID     | `13`                          |
| Priorytet                  | wysoki                        |

## Najważniejsze czerwone flagi

```text
update.exe uruchamia reg.exe
```

Nietypowy plik użytkownika modyfikuje rejestr.

```text
reg.exe zmienia klucz Run
```

Możliwa próba utworzenia autostartu.

```text
wartość nazywa się WindowsUpdate
```

Nazwa może próbować udawać legalną aktualizację.

```text
payload znajduje się w AppData\Roaming
```

Katalog użytkownika jest często wykorzystywany przez legalne programy, ale również przez malware.

```text
update.exe uruchamia się ponownie po logowaniu
```

Mechanizm persistence został potwierdzony.

```text
update.exe uruchamia whoami /groups
```

Możliwe rozpoznanie uprawnień użytkownika.

## Ocena ryzyka

```text
Priorytet: wysoki
```

O wysokim priorytecie zdecydował pełny łańcuch:

```text
podejrzany plik
→ modyfikacja klucza Run
→ autostart
→ ponowne uruchomienie po logowaniu
→ rozpoznanie uprawnień
```

Pole `risk` widoczne w laboratorium było polem treningowym.

W prawdziwym środowisku priorytet może wynikać z:

* reguły SIEM,
* alertu EDR,
* korelacji zdarzeń,
* reputacji pliku,
* klasyfikacji analityka.

## Rekomendowane działania

* odizolowanie hosta `WS-SALES-09` od sieci,
* zabezpieczenie wartości rejestru przed usunięciem do czasu zebrania dowodów,
* eksport podejrzanego klucza rejestru,
* usunięcie wartości `WindowsUpdate` po zabezpieczeniu materiału,
* zakończenie procesu `update.exe`,
* zabezpieczenie pliku do analizy,
* obliczenie pełnego hasha SHA-256,
* sprawdzenie podpisu cyfrowego,
* analiza procesu PowerShell,
* sprawdzenie połączeń sieciowych payloadu,
* wyszukanie identycznego wpisu na innych hostach,
* analiza pozostałych kluczy `Run` i `RunOnce`,
* sprawdzenie innych mechanizmów persistence,
* reset poświadczeń użytkownika w przypadku potwierdzenia incydentu.

## Dodatkowe dane do sprawdzenia

W rzeczywistym środowisku należałoby przeanalizować:

* pełną linię polecenia PowerShell,
* PowerShell Event ID `4104`,
* Sysmon Event ID `3`,
* Sysmon Event ID `11`,
* hash i podpis pliku,
* datę utworzenia pliku,
* połączenia DNS i proxy,
* zadania zaplanowane,
* usługi Windows,
* foldery autostartu,
* inne klucze rejestru persistence,
* aktywność tego samego użytkownika na innych hostach.

## Czego się nauczyłem

* czym jest persistence,
* jak działa klucz rejestru `Run`,
* jaka jest różnica między `Run` i `RunOnce`,
* znaczenia Sysmon Event ID `12`, `13` i `14`,
* analizowania pola `TargetObject`,
* analizowania procesu modyfikującego rejestr,
* sprawdzania procesu nadrzędnego,
* potwierdzania działania persistence po logowaniu,
* odróżniania legalnego użycia `reg.exe` od podejrzanego kontekstu,
* budowania osi czasu zmian rejestru.

## Najważniejszy schemat analizy

```text
kto zmienił rejestr?
        ↓
jaki proces był rodzicem?
        ↓
jaki klucz został zmieniony?
        ↓
jaka wartość została ustawiona?
        ↓
jaki plik ma się uruchamiać?
        ↓
czy uruchomił się po logowaniu?
        ↓
co zrobił później?
```

## Najważniejsza zasada

Samo użycie:

```text
reg.exe
```

nie oznacza ataku.

Samo istnienie wpisu:

```text
CurrentVersion\Run
```

również nie oznacza ataku.

O zagrożeniu decyduje kontekst:

```text
nieznany proces
+ klucz autostartu
+ plik z katalogu użytkownika
+ automatyczny start po logowaniu
+ dalsze podejrzane polecenia
= wysoki poziom ryzyka
```

## Wnioski

Najważniejszym dowodem nie była sama zmiana rejestru.

Kluczowe było potwierdzenie, że wpis rzeczywiście uruchomił payload po zalogowaniu użytkownika.

Pełny łańcuch wyglądał następująco:

```text
PowerShell
    ↓
update.exe
    ↓
reg.exe
    ↓
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
    ↓
WindowsUpdate
    ↓
logowanie użytkownika
    ↓
automatyczne uruchomienie update.exe
    ↓
whoami /groups
```

Analiza rejestru nie polega wyłącznie na znalezieniu podejrzanego klucza.

Najważniejsze jest ustalenie:

```text
kto utworzył wpis,
co wpis uruchamia,
czy mechanizm zadziałał,
co payload zrobił później.
```

