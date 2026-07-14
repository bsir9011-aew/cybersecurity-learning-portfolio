# Lab 02 - Windows Logon Investigation

## Cel laboratorium

Celem laboratorium było przeanalizowanie logów uwierzytelniania Windows i wykrycie podejrzanego zdalnego logowania.

Analiza obejmowała:

* udane i nieudane logowania,
* typy logowania Windows,
* źródłowy adres IP,
* przydzielenie specjalnych uprawnień,
* użycie jawnie podanych poświadczeń,
* odtworzenie osi czasu zdarzeń.

## Środowisko

Laboratorium zostało wykonane w interaktywnym środowisku HTML imitującym dziennik Windows Security.

Wszystkie dane były syntetyczne i zostały przygotowane wyłącznie do nauki analizy defensywnej.

## Analizowane Event ID

| Event ID | Znaczenie                                       |
| -------- | ----------------------------------------------- |
| `4624`   | udane logowanie                                 |
| `4625`   | nieudane logowanie                              |
| `4648`   | logowanie z użyciem jawnie podanych poświadczeń |
| `4672`   | przydzielenie specjalnych uprawnień             |
| `4776`   | walidacja poświadczeń NTLM                      |

## Typy logowania

Pole `Logon Type` opisuje sposób, w jaki użytkownik zalogował się do systemu.

| Logon Type | Znaczenie                                          |
| ---------- | -------------------------------------------------- |
| `2`        | lokalne logowanie interaktywne                     |
| `3`        | logowanie sieciowe                                 |
| `5`        | uruchomienie konta usługi                          |
| `7`        | odblokowanie stacji roboczej                       |
| `10`       | zdalne logowanie przez RDP                         |
| `11`       | logowanie z użyciem poświadczeń zapisanych w cache |

Najważniejsza zasada:

```text
Event ID pokazuje, co się wydarzyło.

Logon Type pokazuje, w jaki sposób nastąpiło logowanie.
```

## Scenariusz

Konto użytkownika zostało użyte do zdalnego logowania na serwer plików.

Następnie konto otrzymało specjalne uprawnienia, a proces PowerShell użył jawnie podanych poświadczeń administratora.

Zaobserwowany łańcuch:

```text
walidacja poświadczeń
        ↓
logowanie RDP
        ↓
specjalne uprawnienia
        ↓
użycie poświadczeń administratora
        ↓
logowanie sieciowe jako administrator
```

## Wyniki analizy

Podejrzane konto:

```text
jan.kowalski
```

Host docelowy:

```text
SRV-FILES-01
```

Źródłowy adres IP:

```text
203.0.113.25
```

Typ logowania:

```text
Logon Type 10 — RDP
```

Priorytet incydentu:

```text
wysoki
```

## Oś czasu zdarzeń

```text
08:11:52
Event ID 4776
Walidacja poświadczeń NTLM użytkownika jan.kowalski
Źródło: 203.0.113.25
        ↓
08:11:54
Event ID 4624
Udane logowanie na SRV-FILES-01
Logon Type 10 — RDP
        ↓
08:11:55
Event ID 4672
Przydzielenie specjalnych uprawnień
SeDebugPrivilege
SeBackupPrivilege
        ↓
08:12:09
Event ID 4648
PowerShell używa jawnie podanych poświadczeń
Konto docelowe: administrator
        ↓
08:12:15
Event ID 4624
Logowanie sieciowe jako administrator
Logon Type 3
```

## Analiza Event ID 4776

Zdarzenie `4776` informuje o walidacji poświadczeń przez system Windows.

W laboratorium dotyczyło konta:

```text
jan.kowalski
```

oraz adresu:

```text
203.0.113.25
```

Samo zdarzenie `4776` nie oznacza ataku.

Stało się podejrzane dopiero po połączeniu go z późniejszym logowaniem RDP i eskalacją uprawnień.

## Analiza Event ID 4624

Zdarzenie `4624` oznacza udane logowanie.

W analizowanym przypadku wystąpiło:

```text
Event ID: 4624
Logon Type: 10
Host: SRV-FILES-01
User: jan.kowalski
Source IP: 203.0.113.25
```

Logon Type `10` wskazywał na logowanie przez Remote Desktop Protocol.

Samo udane logowanie RDP może być legalne.

Należy sprawdzić:

* czy użytkownik powinien logować się na ten serwer,
* czy adres IP jest znany,
* czy logowanie wystąpiło w normalnych godzinach pracy,
* co użytkownik zrobił po zalogowaniu,
* czy konto otrzymało uprawnienia administracyjne.

## Analiza Event ID 4672

Zdarzenie `4672` oznacza przydzielenie specjalnych uprawnień nowej sesji logowania.

W laboratorium przydzielono między innymi:

```text
SeDebugPrivilege
SeBackupPrivilege
```

Takie uprawnienia mogą umożliwiać:

* dostęp do procesów innych użytkowników,
* odczyt chronionych plików,
* wykonywanie kopii danych,
* działania administracyjne.

Zdarzenie `4672` często występuje podczas legalnych logowań kont administracyjnych.

Wymaga jednak analizy, gdy pojawia się po nietypowym logowaniu użytkownika.

## Analiza Event ID 4648

Zdarzenie `4648` oznacza użycie jawnie podanych poświadczeń.

Może wystąpić między innymi podczas użycia:

```text
runas
PowerShell
zaplanowanych zadań
narzędzi administracyjnych
połączeń do innych systemów
```

W analizowanym przypadku:

```text
Proces: powershell.exe
Użytkownik: jan.kowalski
Konto docelowe: administrator
```

Jest to silny sygnał ryzyka, ponieważ użytkownik po zdalnym logowaniu użył poświadczeń konta administratora.

## Drugie logowanie 4624

Po zdarzeniu `4648` wystąpiło kolejne udane logowanie:

```text
User: administrator
Logon Type: 3
Source IP: 127.0.0.1
Process: powershell.exe
```

Logon Type `3` oznacza logowanie sieciowe.

W tym kontekście zdarzenie potwierdza, że poświadczenia administratora zostały użyte przez PowerShell.

## Dlaczego incydent był podejrzany

O wysokim ryzyku nie zdecydowało jedno zdarzenie.

Podejrzany był cały łańcuch:

```text
zewnętrzny adres IP
        ↓
udane logowanie RDP
        ↓
specjalne uprawnienia
        ↓
PowerShell
        ↓
poświadczenia administratora
        ↓
kolejne logowanie jako administrator
```

Najważniejsze sygnały:

* logowanie przez RDP z zewnętrznego adresu IP,
* konto użytkownika uzyskało specjalne uprawnienia,
* PowerShell użył jawnych poświadczeń,
* użyto konta administratora,
* zdarzenia wystąpiły w odstępie kilkunastu sekund.

## Ocena ryzyka

```text
Priorytet: wysoki
```

Powody:

* możliwe przejęcie konta,
* zdalny dostęp do serwera,
* użycie uprawnień administracyjnych,
* możliwość ruchu bocznego,
* możliwość dostępu do plików i danych organizacji.

Pole `risk` w środowisku HTML było polem treningowym.

W prawdziwym środowisku priorytet może pochodzić z:

* reguły SIEM,
* systemu EDR,
* mechanizmu risk scoring,
* klasyfikacji analityka SOC.

## Rekomendowane działania

* potwierdzenie z użytkownikiem, czy wykonywał logowanie,
* sprawdzenie właściciela i reputacji adresu IP,
* odizolowanie serwera w przypadku potwierdzenia incydentu,
* zablokowanie aktywnych sesji,
* reset hasła użytkownika,
* reset poświadczeń administratora,
* wymuszenie ponownej rejestracji MFA,
* analiza aktywności PowerShella,
* sprawdzenie uruchomionych procesów,
* analiza dostępu do plików,
* sprawdzenie ruchu bocznego do innych hostów,
* wyszukanie tego samego adresu IP w całej organizacji,
* sprawdzenie, czy konto logowało się na inne serwery.

## Dodatkowe dane do sprawdzenia

W prawdziwym środowisku należałoby przeanalizować:

* godziny typowej aktywności użytkownika,
* historię logowań RDP,
* geolokalizację adresu IP,
* logi VPN,
* logi firewalla,
* logi PowerShell Script Block Logging,
* procesy utworzone po logowaniu,
* dostęp do udziałów sieciowych,
* modyfikacje kont i grup,
* utworzone usługi i zadania zaplanowane,
* użycie konta administratora na innych hostach.

## Czego się nauczyłem

* znaczenia zdarzeń `4624`, `4625`, `4648`, `4672` i `4776`,
* rozróżniania typów logowania Windows,
* rozpoznawania logowania RDP,
* analizowania źródłowego adresu IP,
* rozpoznawania użycia jawnych poświadczeń,
* analizy specjalnych uprawnień,
* łączenia kilku zdarzeń w jedną oś czasu,
* odróżniania pojedynczego logu od pełnego incydentu.

## Najważniejszy schemat analizy logowania

```text
kto się zalogował?
        ↓
na jaki host?
        ↓
z jakiego adresu IP?
        ↓
jaki był Logon Type?
        ↓
czy logowanie było udane?
        ↓
jakie uprawnienia otrzymało konto?
        ↓
co wydarzyło się później?
```

## Wnioski

Samo zdarzenie `4624` nie oznacza zagrożenia.

Dopiero dodatkowy kontekst pozwala ocenić sytuację:

```text
Event ID
+ Logon Type
+ użytkownik
+ host
+ adres IP
+ kolejne zdarzenia
= pełna analiza logowania
```

Najważniejszą umiejętnością analityka było połączenie logowania RDP, specjalnych uprawnień i użycia konta administratora w jeden spójny łańcuch potencjalnego przejęcia systemu.

