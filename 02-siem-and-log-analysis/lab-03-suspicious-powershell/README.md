# Lab 03 - Suspicious PowerShell Detection

## Cel

Celem laboratorium było wykrycie podejrzanego użycia PowerShella oraz odtworzenie łańcucha zdarzeń prowadzącego od otwarcia dokumentu Word do pobrania i uruchomienia pliku wykonywalnego.

## Scenariusz

Użytkownik otworzył dokument Word otrzymany pocztą.

Chwilę później program Word uruchomił ukryty proces PowerShell z zakodowanym poleceniem. PowerShell pobrał plik wykonywalny z internetu, zapisał go w katalogu publicznym i uruchomił.

Analizowany łańcuch:

```text
OUTLOOK.EXE
    ↓
WINWORD.EXE
    ↓
powershell.exe
    ↓
pobranie pliku
    ↓
uruchomienie update.exe
    ↓
połączenie z zewnętrznym adresem IP
```

## Środowisko

* narzędzie: interaktywny Mini SIEM Lab,
* źródło danych: syntetyczne logi Windows,
* analizowane źródła:

  * Sysmon,
  * Windows Security,
  * PowerShell Script Block Logging.

## Analizowane zdarzenia

| EventCode | Źródło           | Znaczenie                            |
| --------- | ---------------- | ------------------------------------ |
| `1`       | Sysmon           | utworzenie procesu                   |
| `3`       | Sysmon           | połączenie sieciowe                  |
| `11`      | Sysmon           | utworzenie pliku                     |
| `4688`    | Windows Security | utworzenie nowego procesu            |
| `4104`    | PowerShell       | wykonany fragment skryptu PowerShell |

## Zastosowane zapytania SPL

### Wyszukanie PowerShella

```spl
index=endpoint process_name=powershell.exe
```

### Dodatkowe filtrowanie wyników

```spl
index=endpoint
| search process_name=powershell.exe
```

### Wyszukanie zakodowanych poleceń

```spl
index=endpoint
| where like(command_line, "%-enc%")
```

### Wyszukanie podejrzanych funkcji PowerShell

```spl
index=endpoint
| where match(
    command_line,
    "(?i)(invoke-webrequest|downloadstring|frombase64string)"
)
```

### Analiza osi czasu hosta

```spl
index=endpoint host=WS-FIN-07
| sort _time
| table _time source EventCode user process_name parent_process command_line destination
```

### Utworzenie pola oceny ryzyka

```spl
index=endpoint
| eval risk=case(
    match(command_line, "(?i)-enc"), "high",
    like(command_line, "%Invoke-WebRequest%"), "high",
    parent_process="WINWORD.EXE", "medium",
    true(), "low"
)
| table _time host process_name parent_process command_line risk
```

Pole `risk` nie musi istnieć w surowych logach Splunka. W tym przykładzie zostało utworzone za pomocą `eval` i `case()` jako pomocnicza klasyfikacja zdarzeń.

## Wyniki analizy

Podejrzane zdarzenie wystąpiło na hoście:

```text
WS-FIN-07
```

Użytkownik:

```text
marta.wisniewska
```

Proces nadrzędny PowerShella:

```text
WINWORD.EXE
```

Podejrzana komenda:

```powershell
powershell.exe -NoProfile -WindowStyle Hidden -enc SQBuAHYAbw...
```

Zidentyfikowane elementy:

| Element               | Znaczenie                                      |
| --------------------- | ---------------------------------------------- |
| `-NoProfile`          | uruchomienie bez profilu użytkownika           |
| `-WindowStyle Hidden` | ukrycie okna PowerShell                        |
| `-enc`                | wykonanie polecenia zapisanego w Base64        |
| `Invoke-WebRequest`   | pobranie danych lub pliku z internetu          |
| `-Uri`                | adres źródłowy pobieranego pliku               |
| `-OutFile`            | lokalizacja zapisu pobranego pliku             |
| `/silent`             | uruchomienie programu bez widocznej interakcji |

## Pobrany plik

PowerShell pobrał plik:

```text
update.exe
```

Źródłowa domena:

```text
cdn-system-update.example
```

Lokalizacja zapisu:

```text
C:\Users\Public\update.exe
```

Następnie plik został uruchomiony:

```text
C:\Users\Public\update.exe /silent
```

## Połączenie sieciowe

Po uruchomieniu plik `update.exe` nawiązał połączenie z:

```text
198.51.100.44:8443
```

Port `8443` może być używany legalnie, jednak w tym przypadku wystąpił bezpośrednio po pobraniu i uruchomieniu podejrzanego pliku.

## Oś czasu incydentu

```text
10:04:02
WINWORD.EXE otwiera Faktura_Q3.docm
        ↓
10:04:09
WINWORD.EXE uruchamia powershell.exe
        ↓
-NoProfile
-WindowStyle Hidden
-enc
        ↓
10:04:10
Invoke-WebRequest pobiera update.exe
        ↓
10:04:11
PowerShell łączy się z podejrzaną domeną
        ↓
10:04:14
Powstaje C:\Users\Public\update.exe
        ↓
10:04:18
update.exe zostaje uruchomiony z /silent
        ↓
10:04:21
update.exe łączy się z 198.51.100.44:8443
```

## Klasyfikacja

Zaobserwowany wzorzec wskazuje na możliwe wykonanie złośliwego kodu po otwarciu dokumentu.

Najważniejsze sygnały:

* Word uruchamia PowerShell,
* PowerShell działa w ukrytym oknie,
* polecenie jest zakodowane,
* PowerShell pobiera plik wykonywalny,
* pobrany plik zostaje uruchomiony,
* proces nawiązuje połączenie z zewnętrznym adresem IP.

## Ocena ryzyka

```text
Priorytet: wysoki
```

Powodem wysokiego priorytetu jest pełny łańcuch wykonania, a nie pojedynczy przełącznik PowerShella.

Każdy element osobno może mieć legalne zastosowanie. Ich połączenie tworzy jednak silny sygnał możliwej infekcji.

## Rekomendowane działania

* odizolowanie hosta `WS-FIN-07` od sieci,
* zakończenie procesów `powershell.exe` i `update.exe`,
* zabezpieczenie pliku `update.exe` do analizy,
* obliczenie jego skrótu SHA-256,
* zablokowanie domeny i adresu IP,
* sprawdzenie, czy domena lub IP wystąpiły na innych hostach,
* analiza dokumentu `Faktura_Q3.docm`,
* sprawdzenie logów poczty i załącznika,
* analiza procesów potomnych,
* sprawdzenie mechanizmów persistence,
* reset poświadczeń użytkownika, jeśli istnieje ryzyko ich przejęcia.

## Dodatkowe dane do sprawdzenia

W rzeczywistym środowisku należałoby sprawdzić:

* hash pobranego pliku,
* podpis cyfrowy pliku,
* reputację domeny i adresu IP,
* zawartość polecenia Base64,
* logi DNS,
* logi proxy i firewalla,
* procesy potomne `update.exe`,
* nowe usługi i zadania zaplanowane,
* zmiany w rejestrze,
* pliki utworzone po uruchomieniu programu,
* aktywność konta użytkownika po zdarzeniu.

## Czego się nauczyłem

* analizowania pola `parent_process`,
* odróżniania legalnego PowerShella od podejrzanego użycia,
* rozpoznawania przełączników `-enc`, `-NoProfile` i `-WindowStyle Hidden`,
* wyszukiwania fragmentów tekstu za pomocą `like()`,
* stosowania wyrażeń regularnych z `match()`,
* tworzenia nowych pól za pomocą `eval`,
* przypisywania klasyfikacji za pomocą `case()`,
* korelowania logów procesów, plików i połączeń sieciowych,
* odtwarzania pełnej osi czasu incydentu.

## Najważniejszy schemat analizy

```text
rodzic procesu
      ↓
przełączniki
      ↓
treść polecenia
      ↓
pobrany plik
      ↓
uruchomiony proces
      ↓
połączenie sieciowe
```

## Wnioski

Sam fakt uruchomienia `powershell.exe` nie oznacza ataku.

Najważniejszy był kontekst:

```text
WINWORD.EXE
    ↓
powershell.exe -WindowStyle Hidden -enc
    ↓
Invoke-WebRequest
    ↓
update.exe
    ↓
zewnętrzne połączenie sieciowe
```

Analiza SOC nie polega na uznaniu jednego przełącznika za złośliwy. Polega na połączeniu wielu zdarzeń w jedną spójną historię możliwej infekcji.

