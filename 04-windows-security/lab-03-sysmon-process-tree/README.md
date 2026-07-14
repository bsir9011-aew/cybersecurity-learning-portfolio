# Lab 03 - Sysmon Process Tree Investigation

## Cel laboratorium

Celem laboratorium było przeanalizowanie drzewa procesów na podstawie logów Sysmon Event ID `1`.

Analiza obejmowała:

* identyfikację procesu,
* sprawdzenie procesu nadrzędnego,
* analizę pełnej linii polecenia,
* powiązanie procesów za pomocą GUID,
* sprawdzenie podpisu cyfrowego,
* analizę hasha pliku,
* wykrycie podejrzanego łańcucha procesów.

## Środowisko

Laboratorium zostało wykonane w interaktywnym środowisku HTML imitującym analizę logów Sysmon.

Wszystkie dane były syntetyczne i zostały przygotowane wyłącznie do nauki analizy defensywnej.

## Analizowane zdarzenie

| Event ID | Źródło | Znaczenie          |
| -------- | ------ | ------------------ |
| `1`      | Sysmon | utworzenie procesu |

Sysmon Event ID `1` może zawierać między innymi:

* nazwę i ścieżkę procesu,
* linię polecenia,
* użytkownika,
* proces nadrzędny,
* identyfikator procesu,
* `ProcessGuid`,
* `ParentProcessGuid`,
* hash pliku,
* informacje o podpisie cyfrowym.

## Najważniejsze pola

| Pole                | Znaczenie                                  |
| ------------------- | ------------------------------------------ |
| `Image`             | pełna ścieżka uruchomionego procesu        |
| `ParentImage`       | proces, który uruchomił analizowany proces |
| `CommandLine`       | pełna komenda wraz z argumentami           |
| `ProcessGuid`       | unikalny identyfikator procesu             |
| `ParentProcessGuid` | identyfikator procesu nadrzędnego          |
| `Hashes`            | skróty kryptograficzne pliku               |
| `Signed`            | informacja, czy plik jest podpisany        |
| `Signature`         | podmiot, który podpisał plik               |

Najprostszy schemat analizy:

```text
Image
→ co się uruchomiło

ParentImage
→ kto to uruchomił

CommandLine
→ z jakimi parametrami
```

## ProcessGuid i ParentProcessGuid

`ProcessGuid` jest unikalnym identyfikatorem konkretnego procesu.

`ParentProcessGuid` wskazuje proces, który go uruchomił.

Relację można potwierdzić w następujący sposób:

```text
ProcessGuid procesu rodzica
=
ParentProcessGuid procesu dziecka
```

Przykład:

```text
WINWORD.EXE
ProcessGuid: ...0002

powershell.exe
ParentProcessGuid: ...0002
ProcessGuid: ...0003
```

Oznacza to:

```text
WINWORD.EXE → powershell.exe
```

GUID-ów nie trzeba zapamiętywać ani ręcznie odczytywać w całości.

Służą one głównie do jednoznacznego łączenia procesów.

## Scenariusz

Na hoście:

```text
WS-MKT-12
```

użytkownik:

```text
karolina.mazur
```

otworzył dokument otrzymany pocztą.

Następnie powstał podejrzany łańcuch procesów:

```text
OUTLOOK.EXE
    ↓
WINWORD.EXE
    ↓
powershell.exe
    ↓
rundll32.exe
    ↓
updater.tmp
    ↓
cmd.exe
```

## Wyniki analizy

| Element                        | Wynik            |
| ------------------------------ | ---------------- |
| Podejrzany host                | `WS-MKT-12`      |
| Użytkownik                     | `karolina.mazur` |
| Rodzic PowerShella             | `WINWORD.EXE`    |
| Podejrzane narzędzie systemowe | `rundll32.exe`   |
| Niepodpisany plik              | `updater.tmp`    |
| Algorytm hasha                 | `SHA256`         |
| Priorytet                      | wysoki           |

## Oś czasu procesów

```text
09:10:11
OUTLOOK.EXE zostaje uruchomiony
        ↓
09:10:35
OUTLOOK.EXE uruchamia WINWORD.EXE
Dokument: Oferta_wspolpracy.docm
        ↓
09:10:41
WINWORD.EXE uruchamia powershell.exe
        ↓
PowerShell działa w ukrytym oknie
i używa zakodowanego polecenia
        ↓
09:10:44
PowerShell uruchamia rundll32.exe
        ↓
09:10:45
rundll32.exe uruchamia updater.tmp
        ↓
09:11:03
updater.tmp uruchamia cmd.exe
        ↓
cmd.exe wykonuje: whoami /all
```

## Podejrzany PowerShell

Linia polecenia:

```powershell
powershell.exe -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -enc SQBFAFgA...
```

Podejrzane elementy:

| Element                   | Znaczenie                               |
| ------------------------- | --------------------------------------- |
| `-NoProfile`              | uruchomienie bez profilu użytkownika    |
| `-WindowStyle Hidden`     | ukrycie okna PowerShell                 |
| `-ExecutionPolicy Bypass` | ominięcie polityki wykonywania skryptów |
| `-enc`                    | uruchomienie zakodowanego polecenia     |

Najważniejsza czerwona flaga:

```text
WINWORD.EXE → powershell.exe
```

Program Word zwykle nie powinien uruchamiać ukrytego PowerShella z zakodowaną komendą.

## Analiza rundll32.exe

Proces:

```text
rundll32.exe
```

jest legalnym narzędziem systemu Windows.

Może jednak zostać wykorzystany do uruchamiania złośliwego kodu.

Takie legalne narzędzia używane przez atakujących określa się często jako:

```text
LOLBin
```

czyli:

```text
Living Off the Land Binary
```

W laboratorium `rundll32.exe` uruchomił:

```text
C:\Users\karolina.mazur\AppData\Local\Temp\updater.tmp
```

Linia polecenia:

```cmd
rundll32.exe C:\Users\karolina.mazur\AppData\Local\Temp\updater.tmp,Start
```

Czerwone flagi:

* proces został uruchomiony przez PowerShell,
* plik znajdował się w katalogu `Temp`,
* plik miał nietypowe rozszerzenie `.tmp`,
* plik był niepodpisany.

## Analiza updater.tmp

Plik:

```text
updater.tmp
```

znajdował się w:

```text
C:\Users\karolina.mazur\AppData\Local\Temp\
```

Informacje z logu:

```text
Signed: false
Signature: Unsigned
Hash: SHA256=F6A1B2C3D4E5...LAB
```

Samo miejsce w katalogu `Temp` nie oznacza ataku.

Podejrzany był pełny kontekst:

```text
PowerShell
→ rundll32.exe
→ plik z Temp
→ plik niepodpisany
→ uruchomienie cmd.exe
```

## Analiza podpisu cyfrowego

Pole:

```text
Signed
```

informuje, czy plik posiada podpis cyfrowy.

Przykłady:

```text
Signed: true
Signature: Microsoft Windows
```

oraz:

```text
Signed: false
Signature: Unsigned
```

Brak podpisu nie oznacza automatycznie malware.

Wiele legalnych programów może być niepodpisanych.

Jest to jednak istotny sygnał, gdy plik:

* pochodzi z katalogu tymczasowego,
* został uruchomiony przez nietypowy proces,
* powstał po otwarciu dokumentu,
* wykonuje kolejne polecenia systemowe.

## Analiza hasha

Pole `Hashes` zawiera skrót pliku, na przykład:

```text
SHA256=F6A1B2C3D4E5...LAB
```

Hash może służyć do:

* jednoznacznej identyfikacji pliku,
* wyszukania pliku na innych hostach,
* sprawdzenia reputacji,
* porównania z bazami threat intelligence,
* utworzenia reguły blokującej.

W laboratorium użyto algorytmu:

```text
SHA256
```

## Proces potomny cmd.exe

Plik `updater.tmp` uruchomił:

```cmd
cmd.exe /c whoami /all
```

Polecenie:

```text
whoami /all
```

wyświetla informacje o:

* aktualnym użytkowniku,
* grupach użytkownika,
* uprawnieniach,
* tokenie bezpieczeństwa.

Takie polecenie może być używane legalnie.

W tym kontekście może jednak wskazywać na rozpoznanie uprawnień po uzyskaniu dostępu do systemu.

## Najważniejsze czerwone flagi

```text
OUTLOOK.EXE uruchamia WINWORD.EXE
```

Może być normalne podczas otwierania załącznika.

```text
WINWORD.EXE uruchamia powershell.exe
```

Nietypowa i podejrzana relacja.

```text
PowerShell uruchamia rundll32.exe
```

Możliwe użycie legalnego narzędzia do wykonania kodu.

```text
rundll32.exe uruchamia plik z Temp
```

Podejrzana lokalizacja i sposób wykonania.

```text
updater.tmp jest niepodpisany
```

Dodatkowy sygnał ryzyka.

```text
updater.tmp uruchamia cmd.exe /c whoami /all
```

Możliwe rozpoznanie konta i uprawnień.

## Ocena ryzyka

```text
Priorytet: wysoki
```

O wysokim priorytecie zdecydował cały łańcuch procesów:

```text
dokument pocztowy
→ Word
→ ukryty PowerShell
→ rundll32.exe
→ niepodpisany plik z Temp
→ cmd.exe
→ rozpoznanie uprawnień
```

Pole `risk` w laboratorium było polem treningowym.

W prawdziwym środowisku ocena może pochodzić z:

* reguły SIEM,
* systemu EDR,
* reguły Sysmon,
* korelacji zdarzeń,
* decyzji analityka.

## Rekomendowane działania

* odizolowanie hosta `WS-MKT-12` od sieci,
* zabezpieczenie dokumentu `Oferta_wspolpracy.docm`,
* zabezpieczenie pliku `updater.tmp`,
* obliczenie pełnego hasha SHA-256,
* sprawdzenie reputacji hasha,
* analiza pełnej komendy PowerShell,
* dekodowanie polecenia Base64,
* analiza procesów potomnych,
* sprawdzenie połączeń sieciowych,
* wyszukanie hasha na innych hostach,
* wyszukanie podobnego drzewa procesów,
* analiza mechanizmów persistence,
* reset poświadczeń użytkownika w przypadku potwierdzenia incydentu.

## Dodatkowe dane do sprawdzenia

W prawdziwym środowisku należałoby przeanalizować:

* Sysmon Event ID `3` — połączenia sieciowe,
* Sysmon Event ID `11` — utworzone pliki,
* PowerShell Event ID `4104`,
* logi poczty,
* logi proxy i DNS,
* pełny hash pliku,
* podpis cyfrowy,
* czas utworzenia pliku,
* inne procesy uruchomione przez `updater.tmp`,
* zadania zaplanowane,
* usługi,
* wpisy rejestru,
* aktywność konta na innych hostach.

## Czego się nauczyłem

* analizowania Sysmon Event ID `1`,
* rozpoznawania relacji parent–child,
* analizowania pól `Image` i `ParentImage`,
* analizowania pełnej linii polecenia,
* rozumienia `ProcessGuid` i `ParentProcessGuid`,
* sprawdzania podpisu cyfrowego pliku,
* wykorzystywania hasha SHA-256,
* rozpoznawania legalnych narzędzi użytych w podejrzanym kontekście,
* budowania drzewa procesów,
* oceny całego łańcucha zamiast pojedynczego procesu.

## Najważniejszy schemat analizy

```text
co się uruchomiło?
        ↓
kto to uruchomił?
        ↓
z jakimi parametrami?
        ↓
skąd pochodzi plik?
        ↓
czy plik jest podpisany?
        ↓
jaki ma hash?
        ↓
co uruchomił później?
```

## Najważniejsza zasada

Nie każdy proces jest złośliwy sam w sobie.

Przykładowo:

```text
powershell.exe
rundll32.exe
cmd.exe
```

są legalnymi narzędziami Windows.

O zagrożeniu decyduje kontekst:

```text
nietypowy rodzic
+ podejrzana command line
+ plik z katalogu Temp
+ brak podpisu
+ kolejne procesy potomne
= wysoki poziom ryzyka
```

## Wnioski

Najważniejszym elementem analizy było odtworzenie pełnego drzewa procesów:

```text
OUTLOOK.EXE
    ↓
WINWORD.EXE
    ↓
powershell.exe
    ↓
rundll32.exe
    ↓
updater.tmp
    ↓
cmd.exe
```

GUID-y pozwalają technicznie potwierdzić relacje między procesami.

Na początkowym etapie najważniejsze są jednak trzy pola:

```text
Image
ParentImage
CommandLine
```

Pozwalają one odpowiedzieć:

```text
co się uruchomiło?
kto to uruchomił?
z jakimi parametrami?
```

To właśnie te informacje pozwoliły rozpoznać podejrzany łańcuch aktywności.

