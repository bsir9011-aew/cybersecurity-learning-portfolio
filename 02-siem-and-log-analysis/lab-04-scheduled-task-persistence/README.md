# Lab 04 - Scheduled Task Persistence Detection

## Cel

Celem laboratorium było wykrycie mechanizmu persistence utworzonego za pomocą zadania zaplanowanego Windows.

Zadaniem analityka było ustalenie:

* na jakim hoście utworzono zadanie,
* jaki proces je utworzył,
* jaka była nazwa zadania,
* kiedy zadanie miało się uruchamiać,
* jaki plik wykonywało,
* czy mechanizm persistence zadziałał.

## Scenariusz

Po wcześniejszym uruchomieniu podejrzanego PowerShella atakujący próbował utrzymać dostęp do systemu.

W tym celu utworzył zadanie zaplanowane, które uruchamiało plik `update.exe` po każdym logowaniu użytkownika.

Analizowany łańcuch:

```text
powershell.exe
    ↓
schtasks.exe /create
    ↓
zadanie UpdateCheck
    ↓
trigger ONLOGON
    ↓
update.exe
    ↓
połączenie z zewnętrznym adresem IP
```

## Środowisko

* narzędzie: interaktywny Mini SIEM Lab,
* źródło danych: syntetyczne logi Windows,
* analizowane źródła:

  * Sysmon,
  * Windows Security,
  * Task Scheduler Operational Logs.

## Analizowane zdarzenia

| EventCode | Źródło           | Znaczenie                        |
| --------- | ---------------- | -------------------------------- |
| `1`       | Sysmon           | utworzenie procesu               |
| `3`       | Sysmon           | połączenie sieciowe procesu      |
| `4624`    | Windows Security | udane logowanie                  |
| `4698`    | Windows Security | utworzenie zadania zaplanowanego |
| `106`     | Task Scheduler   | zarejestrowanie zadania          |
| `200`     | Task Scheduler   | rozpoczęcie akcji zadania        |

## Zastosowane zapytania SPL

### Wyszukanie procesu schtasks.exe

```spl
index=endpoint process_name=schtasks.exe
```

### Wyszukanie utworzonych zadań

```spl
index=endpoint EventCode=4698
```

### Wyszukanie poleceń tworzących zadanie

```spl
index=endpoint
| where like(command_line, "%/create%")
```

### Wyszukanie triggera ONLOGON lub podejrzanego payloadu

```spl
index=endpoint
| where match(command_line, "(?i)(/sc onlogon|update.exe)")
```

### Analiza osi czasu hosta

```spl
index=endpoint host=WS-FIN-07
| sort _time
| table _time source EventCode user process_name parent_process task_name action command_line
```

### Normalizacja nazwy procesu

```spl
index=endpoint
| eval process_lower=lower(process_name)
```

Funkcja `lower()` zamienia tekst na małe litery, dzięki czemu łatwiej porównywać wartości zapisane w różny sposób.

### Uzupełnienie brakującej wartości

```spl
index=endpoint
| eval task_label=coalesce(task_name, process_name)
```

Funkcja `coalesce()` zwraca pierwszą wartość, która nie jest pusta.

### Grupowanie akcji według zadania

```spl
index=endpoint
| stats values(action) AS actions by task_name
```

Zapytanie pokazuje wszystkie akcje powiązane z danym zadaniem.

## Wyniki analizy

Podejrzana aktywność wystąpiła na hoście:

```text
WS-FIN-07
```

Użytkownik:

```text
marta.wisniewska
```

Proces tworzący zadanie:

```text
schtasks.exe
```

Proces nadrzędny:

```text
powershell.exe
```

Nazwa utworzonego zadania:

```text
UpdateCheck
```

## Polecenie tworzące zadanie

```cmd
schtasks.exe /create /tn UpdateCheck /tr C:\Users\Public\update.exe /sc ONLOGON /f
```

## Znaczenie przełączników

| Element   | Znaczenie                                     |
| --------- | --------------------------------------------- |
| `/create` | utworzenie nowego zadania                     |
| `/tn`     | nazwa zadania                                 |
| `/tr`     | program lub komenda uruchamiana przez zadanie |
| `/sc`     | harmonogram lub trigger                       |
| `ONLOGON` | uruchomienie po zalogowaniu użytkownika       |
| `/f`      | wymuszenie utworzenia lub nadpisania zadania  |

## Payload

Zadanie uruchamiało plik:

```text
C:\Users\Public\update.exe
```

Lokalizacja `C:\Users\Public\` nie jest automatycznie złośliwa, ale może być atrakcyjna dla atakującego, ponieważ jest dostępna dla wielu użytkowników i procesów.

## Oś czasu incydentu

```text
10:04:18
update.exe zostaje uruchomiony przez powershell.exe
        ↓
10:05:02
PowerShell uruchamia schtasks.exe
        ↓
utworzenie zadania UpdateCheck
        ↓
trigger ONLOGON
        ↓
10:05:03
EventCode 4698 — zadanie zostaje utworzone
        ↓
10:05:04
Task Scheduler rejestruje zadanie
        ↓
10:20:00
użytkownik loguje się do systemu
        ↓
10:20:03
zadanie UpdateCheck uruchamia akcję
        ↓
10:20:04
startuje update.exe
        ↓
10:20:07
update.exe łączy się z 198.51.100.44:8443
```

## Klasyfikacja

Zaobserwowany wzorzec wskazuje na mechanizm persistence wykorzystujący zadanie zaplanowane.

Najważniejsze sygnały:

* zadanie zostało utworzone przez `schtasks.exe`,
* procesem nadrzędnym był PowerShell,
* zadanie uruchamiało wcześniej pobrany plik,
* triggerem było logowanie użytkownika,
* payload rzeczywiście uruchomił się po logowaniu,
* proces nawiązał połączenie z zewnętrznym adresem IP.

## Ocena ryzyka

```text
Priorytet: wysoki
```

Powodem wysokiego priorytetu jest potwierdzenie, że mechanizm persistence nie tylko został utworzony, ale również zadziałał.

Po zalogowaniu użytkownika zadanie uruchomiło payload, który następnie nawiązał połączenie sieciowe.

## Rekomendowane działania

* odizolowanie hosta `WS-FIN-07` od sieci,
* wyłączenie i usunięcie zadania `UpdateCheck`,
* zabezpieczenie konfiguracji zadania do analizy,
* zakończenie procesu `update.exe`,
* zabezpieczenie pliku do dalszej analizy,
* obliczenie skrótu SHA-256 pliku,
* zablokowanie adresu `198.51.100.44`,
* sprawdzenie, czy zadanie występuje na innych hostach,
* analiza wszystkich zadań utworzonych w podobnym czasie,
* sprawdzenie innych mechanizmów persistence,
* reset poświadczeń użytkownika,
* analiza wcześniejszego uruchomienia PowerShella.

## Dodatkowe dane do sprawdzenia

W rzeczywistym środowisku należałoby zweryfikować:

* pełną konfigurację XML zadania,
* konto, na którym zadanie się uruchamia,
* poziom uprawnień zadania,
* datę utworzenia i modyfikacji,
* podpis cyfrowy pliku,
* hash pliku,
* procesy potomne payloadu,
* wpisy rejestru,
* nowe usługi,
* foldery autostartu,
* zdarzenia logowania,
* logi DNS, proxy i firewalla,
* aktywność tego samego użytkownika na innych hostach.

## Czego się nauczyłem

* wykrywania procesu `schtasks.exe`,
* rozpoznawania zdarzenia `4698`,
* analizowania parametrów zadania zaplanowanego,
* rozpoznawania triggera `ONLOGON`,
* łączenia utworzenia zadania z jego późniejszym wykonaniem,
* używania funkcji `lower()`,
* używania funkcji `coalesce()`,
* grupowania wartości za pomocą `stats values()`,
* analizowania mechanizmu persistence,
* odtwarzania osi czasu incydentu.

## Najważniejszy schemat analizy

```text
kto utworzył zadanie?
        ↓
jaki proces był rodzicem?
        ↓
jak nazywa się zadanie?
        ↓
kiedy się uruchamia?
        ↓
jaki plik wykonuje?
        ↓
czy zadanie rzeczywiście zadziałało?
        ↓
co zrobił uruchomiony payload?
```

## Uwaga dotycząca pola risk

Kolumna `risk` w środowisku HTML była polem treningowym.

W prawdziwym Splunku pole takie jak:

```text
risk
risk_score
severity
priority
```

może pochodzić z reguły detekcyjnej, aplikacji bezpieczeństwa albo zostać utworzone za pomocą `eval`.

Nie należy traktować go jako informacji występującej automatycznie w każdym surowym logu.

## Wnioski

Samo istnienie zadania zaplanowanego nie oznacza ataku.

W systemach Windows działa wiele legalnych zadań związanych z aktualizacjami, kopiami zapasowymi i administracją.

O podejrzanym charakterze zdarzenia zdecydował pełny kontekst:

```text
PowerShell
    ↓
schtasks.exe /create
    ↓
ONLOGON
    ↓
C:\Users\Public\update.exe
    ↓
uruchomienie po logowaniu
    ↓
połączenie z zewnętrznym IP
```

Najważniejszą umiejętnością analityka było połączenie utworzenia zadania z jego późniejszym wykonaniem i aktywnością sieciową payloadu.

