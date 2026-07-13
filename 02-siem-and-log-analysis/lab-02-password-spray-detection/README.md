# Lab 02 - Password Spraying Detection

## Cel

Celem laboratorium było wykrycie ataku typu password spraying na podstawie syntetycznych logów bezpieczeństwa Windows.

Zadaniem było ustalenie:

* źródłowego adresu IP,
* liczby nieudanych prób logowania,
* liczby atakowanych kont,
* listy użytkowników będących celem,
* czy po serii błędów nastąpiło udane logowanie.

## Scenariusz

System SIEM zarejestrował serię nieudanych prób logowania z jednego adresu IP.

Próby były kierowane do wielu różnych kont, a liczba prób przypadających na pojedynczego użytkownika była niewielka.

Taki wzorzec różni się od klasycznego brute force:

```text
Brute force:
wiele prób → jedno konto

Password spraying:
mało prób na konto → wiele kont
```

## Środowisko

* narzędzie: interaktywny Mini SIEM Lab,
* źródło danych: syntetyczne logi Windows,
* analizowane zdarzenia:

  * `4624` — udane logowanie,
  * `4625` — nieudane logowanie,
  * `4672` — przypisanie specjalnych uprawnień do nowego logowania,
  * `4768` — żądanie biletu uwierzytelniającego Kerberos TGT,
  * `4740` — zablokowanie konta.

## Zastosowane zapytania

### Nieudane logowania

```spl
index=security EventCode=4625
```

### Liczba prób według adresu IP

```spl
index=security EventCode=4625
| stats count by src_ip
| sort - count
```

### Liczba unikalnych kont

```spl
index=security EventCode=4625
| stats count, dc(user) AS unique_users by src_ip
| sort - unique_users
```

`dc(user)` oznacza distinct count, czyli liczbę unikalnych wartości pola `user`.

### Lista atakowanych użytkowników

```spl
index=security EventCode=4625
| stats count,
        dc(user) AS unique_users,
        values(user) AS targeted_users
  by src_ip
```

`values(user)` zwraca listę użytkowników powiązanych z danym adresem IP.

### Filtrowanie podejrzanych adresów

```spl
index=security EventCode=4625
| stats count,
        dc(user) AS unique_users,
        values(user) AS targeted_users
  by src_ip
| where unique_users > 5
| sort - unique_users
```

### Analiza osi czasu podejrzanego IP

```spl
index=security src_ip=203.0.113.77
| sort _time
| table _time EventCode user src_ip host logon_type
```

## Wyniki analizy

Podejrzana aktywność pochodziła z adresu:

```text
203.0.113.77
```

Zidentyfikowano:

* `12` nieudanych prób logowania,
* `10` unikalnych atakowanych kont,
* host docelowy: `VPN-GW01`,
* typ logowania: `3` — logowanie sieciowe.

Atakowane konta obejmowały między innymi:

```text
jan.kowalski
anna.nowak
piotr.zielinski
marta.wisniewska
tomasz.wojcik
karolina.mazur
adam.kaczmarek
monika.dabrowska
rafal.lewandowski
ewa.krol
```

## Oś czasu incydentu

Po serii nieudanych prób wystąpiło udane logowanie na konto:

```text
marta.wisniewska
```

Następnie zarejestrowano:

* `4624` - udane logowanie,
* `4672` - przypisanie specjalnych uprawnień,
* `4768` - żądanie biletu Kerberos TGT.

Uproszczona oś czasu:

```text
12 × EventCode 4625
        ↓
10 różnych kont
        ↓
EventCode 4624
udane logowanie: marta.wisniewska
        ↓
EventCode 4672
specjalne uprawnienia
        ↓
EventCode 4768
żądanie biletu Kerberos
```

## Klasyfikacja

Zaobserwowany wzorzec odpowiada atakowi typu:

```text
Password spraying
```

Powody:

* jeden źródłowy adres IP,
* wiele różnych kont,
* niewielka liczba prób przypadająca na konto,
* udane logowanie po serii błędnych prób.

## Ocena ryzyka

Incydent należy traktować jako możliwe przejęcie konta.

Samo zdarzenie `4624` nie potwierdza jeszcze, że logowanie było wykonane przez atakującego. Jednak jego wystąpienie z tego samego adresu IP bezpośrednio po serii nieudanych prób znacząco zwiększa podejrzenie kompromitacji.

Zdarzenie `4672` dodatkowo podnosi ryzyko, ponieważ może wskazywać, że sesja otrzymała uprawnienia specjalne.

### Priorytet

```text
Wysoki
```

Priorytet nie został określony jako krytyczny, ponieważ nie potwierdzono jeszcze wykonania złośliwych działań po zalogowaniu.

## Rekomendowane działania

* tymczasowe zablokowanie konta `marta.wisniewska`,
* wymuszenie zmiany hasła,
* sprawdzenie, czy użytkownik rozpoznaje logowanie,
* zablokowanie lub ograniczenie adresu `203.0.113.77`,
* analiza aktywności po udanym logowaniu,
* sprawdzenie procesów, poleceń i połączeń sieciowych,
* weryfikacja zmian w grupach i uprawnieniach,
* sprawdzenie pozostałych atakowanych kont,
* weryfikacja działania MFA,
* sprawdzenie reputacji adresu IP w źródłach threat intelligence.

## Dodatkowe dane do sprawdzenia

W rzeczywistym środowisku należałoby zebrać:

* geolokalizację adresu IP,
* reputację adresu IP,
* informacje o urządzeniu użytkownika,
* logi VPN,
* logi uwierzytelnienia Kerberos,
* zdarzenia PowerShell,
* procesy uruchomione po logowaniu,
* zmiany w Active Directory,
* aktywność sieciową konta,
* czas trwania sesji,
* dane z systemu MFA.

## Czego się nauczyłem

* rozpoznawania wzorca password spraying,
* odróżniania password spraying od brute force,
* liczenia unikalnych wartości za pomocą `dc()`,
* wyświetlania listy wartości za pomocą `values()`,
* nadawania aliasów za pomocą `AS`,
* filtrowania wyników agregacji przy użyciu `where`,
* analizowania osi czasu podejrzanego adresu IP,
* korelowania nieudanych i udanych logowań.

## Najważniejsza różnica

```text
Brute force
1 konto + wiele prób

Password spraying
wiele kont + mało prób na konto
```

Atak password spraying może być trudniejszy do wykrycia, ponieważ atakujący ogranicza liczbę prób dla pojedynczego użytkownika, aby uniknąć blokady konta.

## Wnioski

Samo liczenie wszystkich nieudanych logowań może nie wystarczyć do wykrycia password spraying.

Najważniejszym sygnałem była liczba unikalnych kont atakowanych z jednego adresu IP:

```spl
dc(user)
```

Dopiero połączenie agregacji z analizą osi czasu pokazało pełny przebieg incydentu:

```text
jeden adres IP
        ↓
wiele różnych kont
        ↓
udane logowanie
        ↓
możliwe przejęcie konta
```

