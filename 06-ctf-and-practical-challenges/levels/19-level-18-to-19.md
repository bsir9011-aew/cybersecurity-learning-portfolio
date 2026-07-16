# Bandit 18 → 19 — `.bashrc` i zdalne wykonanie polecenia

## 🎯 Cel

Odczytać plik mimo automatycznego zamykania interaktywnej sesji SSH.

## 🧠 Nowe pojęcie

Po zwykłym logowaniu SSH uruchamiana jest powłoka. Jej pliki startowe, np. `.bashrc`, mogą wykonywać polecenia automatycznie. W tym przypadku uruchamiały `exit`, dlatego po poprawnym zalogowaniu pojawiał się komunikat pożegnalny i sesja była zamykana.

SSH potrafi jednak nie tylko otwierać terminal, ale również wykonać jedno zdalne polecenie.

## 🧰 Poznany wzorzec

```bash
ssh <użytkownik>@<host> -p <port> "<polecenie>"
```

## 🔎 Mój sposób myślenia

```text
zwykłe SSH → interaktywna powłoka → plik startowy → exit
SSH z poleceniem → wykonanie zadania → zwrot wyniku → koniec
```

Najpierw można zdalnie wykonać polecenie listujące pliki, a dopiero potem wybrać właściwy plik do odczytu.

## ⚠️ Pułapka

Komunikat `Byebye!` nie oznaczał błędnego hasła. Logowanie się udało, lecz uruchomiona powłoka natychmiast zakończyła działanie.

## 🛡️ Znaczenie w cyberbezpieczeństwie

Zdalne wykonywanie pojedynczych komend jest powszechne w administracji, automatyzacji i reagowaniu na incydenty.

---

[← Powrót do README](../README.md)
