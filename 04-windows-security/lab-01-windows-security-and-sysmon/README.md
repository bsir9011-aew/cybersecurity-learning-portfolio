# Lab 01 - Windows Security and Sysmon Basics

## Cel laboratorium

Celem laboratorium było poznanie podstaw analizy logów Windows oraz zrozumienie różnicy między:

* Windows Security,
* Sysmon,
* zdarzeniami logowania,
* zdarzeniami utworzenia procesu,
* zdarzeniami sieciowymi i plikowymi.

## Środowisko

Laboratorium zostało wykonane w interaktywnym środowisku HTML imitującym Windows Event Viewer.

Wykorzystane źródła logów:

* Windows Security,
* Sysmon.

Wszystkie dane użyte w laboratorium były syntetyczne.

## Windows Event Viewer

Podgląd zdarzeń Windows można uruchomić poleceniem:

```text
eventvwr.msc
```

Logi bezpieczeństwa znajdują się w:

```text
Dzienniki systemu Windows
→ Zabezpieczenia
```

Logi Sysmon znajdują się w:

```text
Dzienniki aplikacji i usług
→ Microsoft
→ Windows
→ Sysmon
→ Operational
```

Sysmon musi zostać wcześniej zainstalowany i skonfigurowany.

## Analizowane Event ID

### Windows Security

| Event ID | Znaczenie                 |
| -------- | ------------------------- |
| `4624`   | udane logowanie           |
| `4625`   | nieudane logowanie        |
| `4688`   | utworzenie nowego procesu |

### Sysmon

| Event ID | Znaczenie           |
| -------- | ------------------- |
| `1`      | utworzenie procesu  |
| `3`      | połączenie sieciowe |
| `11`     | utworzenie pliku    |

## Security 4688 a Sysmon 1

Oba zdarzenia mogą informować o utworzeniu procesu.

### Windows Security 4688

Może zawierać:

* nazwę procesu,
* użytkownika,
* identyfikator procesu,
* proces nadrzędny,
* linię polecenia, jeśli audyt jest odpowiednio skonfigurowany.

### Sysmon Event ID 1

Zwykle zawiera więcej szczegółów:

* pełną ścieżkę procesu,
* pełną linię polecenia,
* proces nadrzędny,
* ścieżkę procesu nadrzędnego,
* użytkownika,
* hash pliku,
* identyfikatory procesów.

Najważniejsza różnica:

```text
Windows Security pokazuje, że proces powstał.

Sysmon pomaga dokładniej ustalić:
→ skąd proces pochodził,
→ kto go uruchomił,
→ z jakimi parametrami,
→ jaki proces był jego rodzicem.
```

## Scenariusz analizy

Podejrzana aktywność wystąpiła na hoście:

```text
WS-FIN-07
```

Użytkownik:

```text
marta.wisniewska
```

Użytkownik otworzył dokument:

```text
Faktura_Q3.docm
```

Następnie został uruchomiony PowerShell.

Zaobserwowany łańcuch:

```text
OUTLOOK.EXE
    ↓
WINWORD.EXE
    ↓
powershell.exe
    ↓
utworzenie pliku update.exe
    ↓
uruchomienie update.exe
    ↓
połączenie sieciowe
```

## Podejrzany proces PowerShell

Proces:

```text
powershell.exe
```

Proces nadrzędny:

```text
WINWORD.EXE
```

Linia polecenia:

```powershell
powershell.exe -NoProfile -WindowStyle Hidden -enc SQBuAHYAbw...
```

Podejrzane elementy:

| Element                   | Znaczenie                               |
| ------------------------- | --------------------------------------- |
| `WINWORD.EXE` jako rodzic | dokument Word uruchomił PowerShell      |
| `-NoProfile`              | uruchomienie bez profilu użytkownika    |
| `-WindowStyle Hidden`     | ukrycie okna PowerShell                 |
| `-enc`                    | wykonanie zakodowanego polecenia Base64 |

Pojedynczy przełącznik nie musi oznaczać ataku.

Podejrzany jest cały kontekst:

```text
Word
→ ukryty PowerShell
→ zakodowane polecenie
→ utworzenie pliku
→ połączenie sieciowe
```

## Zdarzenie sieciowe

Sysmon Event ID `3` pokazał połączenie procesu:

```text
powershell.exe
```

z adresem:

```text
198.51.100.44:443
```

W analizie zdarzenia sieciowego ważne są:

* proces wykonujący połączenie,
* adres docelowy,
* port docelowy,
* protokół,
* użytkownik,
* czas zdarzenia.

## Utworzenie pliku

Sysmon Event ID `11` pokazał utworzenie pliku:

```text
C:\Users\Public\update.exe
```

Proces tworzący plik:

```text
powershell.exe
```

Następnie plik został uruchomiony:

```text
C:\Users\Public\update.exe /silent
```

## Relacja proces rodzic–dziecko

Jednym z najważniejszych elementów analizy było sprawdzenie relacji:

```text
parent_process → process
```

W tym przypadku:

```text
WINWORD.EXE → powershell.exe
```

oraz:

```text
powershell.exe → update.exe
```

Relacje między procesami pomagają odtworzyć przebieg aktywności na hoście.

Normalny proces może stać się podejrzany, jeśli został uruchomiony przez nietypowego rodzica.

## Wyniki laboratorium

| Element                          | Wynik              |
| -------------------------------- | ------------------ |
| Podejrzany host                  | `WS-FIN-07`        |
| Użytkownik                       | `marta.wisniewska` |
| Proces nadrzędny PowerShella     | `WINWORD.EXE`      |
| Security Event ID procesu        | `4688`             |
| Sysmon Event ID procesu          | `1`                |
| Sysmon Event ID połączenia       | `3`                |
| Sysmon Event ID utworzenia pliku | `11`               |
| Priorytet                        | wysoki             |

## Ocena ryzyka

```text
Priorytet: wysoki
```

Powodem wysokiej oceny był pełny łańcuch zdarzeń:

```text
dokument Word
→ PowerShell
→ ukryte i zakodowane polecenie
→ utworzenie pliku EXE
→ uruchomienie pliku
→ połączenie z zewnętrznym adresem
```

Pole `risk` widoczne w laboratorium było polem treningowym.

W prawdziwym środowisku ocena ryzyka może pochodzić z:

* reguły detekcyjnej,
* SIEM,
* systemu EDR,
* klasyfikacji analityka.

## Rekomendowane działania

* odizolowanie hosta od sieci,
* zabezpieczenie pliku `update.exe`,
* obliczenie hasha SHA-256,
* sprawdzenie reputacji adresu IP,
* analiza dokumentu `Faktura_Q3.docm`,
* sprawdzenie logów pocztowych,
* analiza procesów potomnych,
* sprawdzenie innych hostów pod kątem tego samego pliku lub adresu IP,
* analiza mechanizmów persistence.

## Czego się nauczyłem

* poruszania się po Windows Event Viewer,
* rozróżniania Windows Security i Sysmon,
* znaczenia zdarzeń `4624`, `4625` i `4688`,
* znaczenia zdarzeń Sysmon `1`, `3` i `11`,
* analizy relacji parent–child,
* analizy linii polecenia procesu,
* analizy utworzenia pliku,
* analizy połączenia sieciowego,
* łączenia wielu zdarzeń w jedną oś czasu.

## Najważniejszy schemat analizy

```text
kto?
↓
na jakim hoście?
↓
jaki proces się uruchomił?
↓
kto był jego rodzicem?
↓
jaka była linia polecenia?
↓
czy utworzono plik?
↓
czy nastąpiło połączenie sieciowe?
↓
co wydarzyło się później?
```

## Wnioski

Windows Security i Sysmon uzupełniają się:

```text
Windows Security
→ logowania i podstawowy audyt systemu

Sysmon
→ dokładniejsza telemetria procesów, plików i sieci
```

Najważniejszą umiejętnością analityka nie jest zapamiętanie wszystkich Event ID.

Najważniejsze jest połączenie zdarzeń w logiczny łańcuch aktywności.
