# Lab 01 - Brute Force Detection in Windows Logs

## Cel

Celem laboratorium było przeanalizowanie zdarzeń logowania systemu Windows i wykrycie próby ataku brute force na konto administracyjne.

## Scenariusz

System SIEM wygenerował alert dotyczący wielu nieudanych prób logowania.

Zadaniem analityka było ustalenie:

* źródłowego adresu IP,
* liczby nieudanych prób,
* atakowanego konta,
* czy po serii błędów nastąpiło udane logowanie.

## Środowisko

* narzędzie: interaktywny Mini SIEM Lab,
* źródło danych: syntetyczne logi Windows,
* analizowane zdarzenia:

  * `4624` - udane logowanie,
  * `4625` - nieudane logowanie,
  * `4672` - przydzielenie specjalnych uprawnień,
  * `4740` - zablokowanie konta.

## Zastosowane zapytania

Filtrowanie nieudanych logowań:

```text
index=security EventCode=4625
```

Zliczenie nieudanych prób według adresu IP:

```text
index=security EventCode=4625 | stats count by src_ip
```

Filtrowanie zdarzeń konkretnego użytkownika:

```text
index=security user=administrator
```

Analiza aktywności podejrzanego adresu:

```text
index=security src_ip=185.220.101.14
```

## Wyniki analizy

Najwięcej nieudanych prób logowania pochodziło z adresu:

```text
185.220.101.14
```

Zidentyfikowano:

* `7` nieudanych prób logowania,
* atakowane konto: `administrator`,
* host docelowy: `SRV-FILE01`,
* typ logowania: `3` — logowanie sieciowe.

Po serii nieudanych prób nastąpiło:

1. udane logowanie - zdarzenie `4624`,
2. przydzielenie specjalnych uprawnień — zdarzenie `4672`.

## Oś czasu

```text
7 × EventCode 4625
        ↓
udane logowanie — EventCode 4624
        ↓
specjalne uprawnienia — EventCode 4672
```

## Ocena incydentu

Zdarzenie nie powinno zostać zaklasyfikowane wyłącznie jako nieudana próba brute force.

Udane logowanie z tego samego adresu IP po serii błędnych prób może oznaczać, że atakujący odgadł lub uzyskał prawidłowe hasło.

Późniejsze przydzielenie specjalnych uprawnień zwiększa ryzyko, ponieważ konto `administrator` posiada dostęp uprzywilejowany.

### Klasyfikacja

```text
Brute force zakończony możliwym przejęciem konta
```

### Priorytet

```text
Wysoki / Critical
```

## Rekomendowane działania

* natychmiastowe zablokowanie lub ograniczenie konta,
* reset hasła administratora,
* zablokowanie źródłowego adresu IP,
* sprawdzenie aktywności wykonanej po udanym logowaniu,
* analiza procesów uruchomionych na serwerze,
* sprawdzenie zmian w plikach, usługach i zadaniach zaplanowanych,
* analiza pozostałych logów z hosta,
* sprawdzenie, czy ten sam adres IP atakował inne konta,
* weryfikacja użycia mechanizmu MFA.

## Dodatkowe dane do sprawdzenia

W rzeczywistym środowisku należałoby sprawdzić:

* dokładny czas trwania sesji,
* procesy uruchomione przez konto administratora,
* polecenia PowerShell,
* nowe usługi i zadania zaplanowane,
* połączenia sieciowe po zalogowaniu,
* próby dostępu do innych hostów,
* zmiany w kontach i grupach,
* pobierane lub tworzone pliki.

## Czego się nauczyłem

* filtrowania logów według EventCode,
* agregowania zdarzeń według adresu IP,
* rozpoznawania wzorca brute force,
* łączenia wielu zdarzeń w jedną historię incydentu,
* odróżniania nieudanej próby ataku od możliwego przejęcia konta,
* nadawania incydentowi odpowiedniego priorytetu.

## Wnioski

Pojedyncze zdarzenie `4625` nie musi oznaczać ataku.

Seria zdarzeń `4625` z jednego adresu IP może wskazywać na brute force, ale dopiero korelacja z późniejszym zdarzeniem `4624` pokazuje, że atak mógł zakończyć się sukcesem.

Najważniejszą umiejętnością analityka SOC nie jest samo znalezienie błędnych logowań, ale połączenie zdarzeń w logiczną historię:

```text
próby logowania → sukces → uzyskanie uprawnień → możliwe działania atakującego
```

