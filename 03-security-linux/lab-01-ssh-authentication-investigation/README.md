# Lab 01 - SSH Authentication Investigation

## Cel laboratorium

Celem laboratorium było przeanalizowanie logów uwierzytelniania systemu Linux oraz wykrycie przejęcia konta przez SSH.

Analiza obejmowała:

* nieudane próby logowania,
* udane logowanie po serii błędów,
* źródłowy adres IP,
* otwarcie sesji SSH,
* użycie `sudo`,
* utworzenie nowego użytkownika,
* nadanie nowemu kontu uprawnień administracyjnych.

## Środowisko

Laboratorium zostało wykonane w interaktywnym środowisku HTML imitującym terminal Linux i analizę logów bezpieczeństwa.

Wszystkie dane były syntetyczne i zostały przygotowane wyłącznie do nauki analizy defensywnej.

## Logi uwierzytelniania Linux

W systemach Ubuntu i Debian zdarzenia uwierzytelniania często znajdują się w:

```text
/var/log/auth.log
```

W systemach takich jak RHEL lub CentOS podobne informacje mogą być zapisywane w:

```text
/var/log/secure
```

W systemach używających `systemd` logi można również analizować za pomocą:

```bash
journalctl
```

## Windows a Linux

W systemie Windows zdarzenia są często identyfikowane przez Event ID:

```text
4624 → udane logowanie
4625 → nieudane logowanie
```

W Linuxie podobne informacje mogą występować jako tekst:

```text
Failed password
Accepted password
session opened
sudo
useradd
```

Najważniejsza różnica:

```text
Windows → Event ID
Linux   → komunikaty tekstowe
```

Sposób analizy pozostaje jednak podobny:

```text
konto
→ host
→ adres IP
→ czas
→ wynik logowania
→ dalsza aktywność
```

## Analizowane polecenia

### Wyświetlenie całego pliku

```bash
cat /var/log/auth.log
```

Polecenie `cat` wyświetla zawartość pliku.

### Wyszukanie nieudanych logowań

```bash
grep "Failed password" /var/log/auth.log
```

Polecenie pokazuje tylko linie zawierające tekst `Failed password`.

### Wyszukanie udanych logowań

```bash
grep "Accepted password" /var/log/auth.log
```

### Wyszukanie użycia sudo

```bash
grep "sudo:" /var/log/auth.log
```

### Logi usługi SSH

```bash
journalctl -u ssh
```

W zależności od dystrybucji usługa może nazywać się:

```bash
journalctl -u sshd
```

### Historia udanych logowań

```bash
last
```

### Historia nieudanych logowań

```bash
lastb
```

## Najważniejsze pojęcia

| Element             | Znaczenie                                    |
| ------------------- | -------------------------------------------- |
| `ssh`               | protokół zdalnego dostępu                    |
| `sshd`              | usługa przyjmująca połączenia SSH            |
| `Failed password`   | nieudana próba logowania                     |
| `Accepted password` | udane logowanie                              |
| `session opened`    | otwarcie sesji użytkownika                   |
| `sudo`              | wykonanie polecenia z wyższymi uprawnieniami |
| `useradd`           | utworzenie konta użytkownika                 |
| `usermod`           | modyfikacja istniejącego konta               |

## Scenariusz

Na serwerze:

```text
web-01
```

wystąpiła seria prób logowania z adresu:

```text
185.220.101.14
```

Atakujący próbował użyć kont:

```text
root
admin
ubuntu
deploy
```

Po kilku nieudanych próbach nastąpiło udane logowanie na konto:

```text
deploy
```

Następnie konto użyło `sudo`, aby utworzyć nowego użytkownika:

```text
backupsvc
```

Nowe konto zostało później dodane do grupy `sudo`.

## Wyniki analizy

| Element                          | Wynik               |
| -------------------------------- | ------------------- |
| Podejrzany host                  | `web-01`            |
| Źródłowy adres IP                | `185.220.101.14`    |
| Przejęte konto                   | `deploy`            |
| Nowe konto                       | `backupsvc`         |
| Mechanizm podniesienia uprawnień | `sudo`              |
| Plik logów                       | `/var/log/auth.log` |
| Priorytet                        | wysoki              |

## Oś czasu incydentu

```text
08:40:03
Nieudane logowanie na konto root
        ↓
08:40:08
Próba logowania na konto admin
        ↓
08:40:14
Próba logowania na konto ubuntu
        ↓
08:40:21–08:40:37
Kolejne próby logowania na konto deploy
        ↓
08:40:48
Accepted password for deploy
        ↓
08:40:49
Otwarcie sesji SSH
        ↓
08:41:12
sudo useradd -m backupsvc
        ↓
08:41:13
Utworzenie konta backupsvc
        ↓
08:41:30
sudo usermod -aG sudo backupsvc
```

## Analiza nieudanych logowań

Przykładowy log:

```text
Failed password for deploy from 185.220.101.14 port 48132 ssh2
```

Znaczenie:

| Element      | Wartość            |
| ------------ | ------------------ |
| Wynik        | nieudane logowanie |
| Konto        | `deploy`           |
| Źródłowe IP  | `185.220.101.14`   |
| Port klienta | `48132`            |
| Protokół     | SSH                |

Pojedyncze błędne hasło nie musi oznaczać ataku.

Podejrzany był wzorzec:

```text
jedno IP
→ wiele kont
→ wiele nieudanych prób
→ późniejsze udane logowanie
```

## Udane logowanie

Kluczowy komunikat:

```text
Accepted password for deploy from 185.220.101.14 port 48158 ssh2
```

Udane logowanie nastąpiło z tego samego adresu IP, z którego wcześniej pochodziły nieudane próby.

To wskazuje na możliwość:

* odgadnięcia hasła,
* użycia wykradzionych danych logowania,
* ataku brute force,
* password sprayingu.

## Otwarcie sesji SSH

Po udanym logowaniu wystąpił komunikat:

```text
pam_unix(sshd:session): session opened for user deploy
```

Oznacza to, że sesja użytkownika została faktycznie otwarta.

Samo `Accepted password` pokazuje sukces uwierzytelnienia.

`session opened` potwierdza rozpoczęcie sesji.

## Użycie sudo

Po zalogowaniu konto `deploy` wykonało:

```bash
sudo useradd -m backupsvc
```

Znaczenie:

| Element     | Znaczenie                                          |
| ----------- | -------------------------------------------------- |
| `sudo`      | wykonanie polecenia z uprawnieniami administratora |
| `useradd`   | utworzenie użytkownika                             |
| `-m`        | utworzenie katalogu domowego                       |
| `backupsvc` | nazwa nowego konta                                 |

Następnie wykonano:

```bash
sudo usermod -aG sudo backupsvc
```

Znaczenie:

| Element     | Znaczenie                               |
| ----------- | --------------------------------------- |
| `usermod`   | modyfikacja użytkownika                 |
| `-aG`       | dodanie do dodatkowej grupy             |
| `sudo`      | grupa z uprawnieniami administracyjnymi |
| `backupsvc` | modyfikowane konto                      |

Nowe konto uzyskało możliwość wykonywania poleceń administracyjnych.

## Możliwy mechanizm persistence

Utworzenie nowego konta może służyć do utrzymania dostępu.

Atakujący nie musi później korzystać z przejętego konta `deploy`.

Może próbować logować się jako:

```text
backupsvc
```

Nazwa może udawać legalne konto techniczne związane z kopiami zapasowymi.

Podejrzany jest pełny kontekst:

```text
przejęte konto
→ sudo
→ nowe konto
→ grupa administracyjna
```

## Dlaczego incydent miał wysoki priorytet

O wysokim ryzyku zdecydował pełny łańcuch:

```text
seria nieudanych logowań
→ udane logowanie z tego samego IP
→ otwarcie sesji
→ użycie sudo
→ utworzenie nowego konta
→ nadanie uprawnień administracyjnych
```

Nie był to już tylko alert dotyczący błędnego hasła.

Zdarzenia potwierdziły działania wykonane po uzyskaniu dostępu.

## Oddzielanie szumu od incydentu

Analizę rozpoczęto od wszystkich logów SSH.

Następnie dane były zawężane:

```text
wszystkie logi
        ↓
Failed password
        ↓
adres 185.220.101.14
        ↓
konto deploy
        ↓
Accepted password
        ↓
session opened
        ↓
sudo
        ↓
useradd i usermod
```

Najważniejszy zestaw filtrów:

```text
IP + konto + host + czas + dalsza aktywność
```

Sam adres IP nie zawsze wystarcza, ponieważ może należeć do:

* VPN,
* serwera proxy,
* wspólnej sieci,
* infrastruktury NAT.

## Ocena ryzyka

```text
Priorytet: wysoki
```

Możliwe skutki:

* przejęcie konta,
* uzyskanie dostępu do serwera,
* utworzenie ukrytego konta,
* podniesienie uprawnień,
* utrzymanie dostępu,
* dalszy ruch boczny,
* modyfikacja danych lub usług.

Pole `risk` w środowisku HTML było polem treningowym.

## Rekomendowane działania

* odizolowanie serwera od sieci,
* zakończenie podejrzanej sesji SSH,
* zablokowanie źródłowego adresu IP,
* wyłączenie konta `deploy`,
* usunięcie lub zablokowanie konta `backupsvc`,
* sprawdzenie pliku `/etc/passwd`,
* sprawdzenie grup w `/etc/group`,
* analiza konfiguracji `sudo`,
* reset poświadczeń,
* sprawdzenie kluczy SSH,
* analiza historii poleceń,
* sprawdzenie uruchomionych procesów,
* analiza połączeń sieciowych,
* sprawdzenie innych serwerów pod kątem tego samego IP i konta.

## Dodatkowe dane do sprawdzenia

W prawdziwym środowisku należałoby przeanalizować:

```bash
last
lastb
who
w
ps aux
ss -tulpn
cat /etc/passwd
cat /etc/group
grep backupsvc /etc/passwd
grep backupsvc /etc/group
```

Warto również sprawdzić:

* `~/.bash_history`,
* `~/.ssh/authorized_keys`,
* `/etc/sudoers`,
* `/etc/sudoers.d/`,
* zadania `cron`,
* usługi `systemd`,
* ostatnio zmodyfikowane pliki,
* logi firewalla,
* logi proxy i SIEM.

## Czego się nauczyłem

* lokalizacji podstawowych logów uwierzytelniania Linux,
* znaczenia `Failed password` i `Accepted password`,
* filtrowania logów za pomocą `grep`,
* używania `journalctl`,
* działania poleceń `last` i `lastb`,
* rozpoznawania sesji SSH,
* analizowania użycia `sudo`,
* rozpoznawania poleceń `useradd` i `usermod`,
* wykrywania utworzenia konta administracyjnego,
* zawężania danych po IP, koncie, hoście i czasie,
* budowania pełnej osi czasu incydentu.

## Najważniejszy schemat analizy

```text
co się wydarzyło?
        ↓
na jakim serwerze?
        ↓
jakie konto?
        ↓
z jakiego IP?
        ↓
ile było nieudanych prób?
        ↓
czy potem nastąpił sukces?
        ↓
czy otwarto sesję?
        ↓
czy użyto sudo?
        ↓
co zmieniono w systemie?
```

## Wnioski

Najważniejszym sygnałem nie była sama liczba nieudanych logowań.

Kluczowe było połączenie zdarzeń:

```text
Failed password
→ Accepted password
→ session opened
→ sudo
→ useradd
→ usermod
```

Analiza logów polegała na stopniowym eliminowaniu szumu:

```text
wykrycie wzorca
→ zawężenie po IP
→ zawężenie po koncie
→ analiza hosta i czasu
→ sprawdzenie dalszej aktywności
```

Dzięki temu seria zwykłych komunikatów tekstowych została połączona w historię możliwego przejęcia serwera. 🐧🛡️
