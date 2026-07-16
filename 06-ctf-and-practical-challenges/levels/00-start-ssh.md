# Bandit — start: logowanie przez SSH

## 🎯 Cel

Połączyć się ze zdalnym serwerem gry przy użyciu klienta SSH i niestandardowego portu.

## 🧠 Nowe pojęcie

**SSH** tworzy szyfrowane połączenie ze zdalnym systemem. Potrzebne są: użytkownik, host, port i metoda uwierzytelnienia.

## 🧰 Poznana składnia

```bash
ssh <użytkownik>@<host> -p <port>
```

- `ssh` — klient zdalnego połączenia,
- `<użytkownik>@<host>` — konto i serwer,
- `-p` — wskazanie portu innego niż domyślny `22`.

## 🔎 Mój sposób myślenia

Najpierw identyfikuję cztery elementy: **kto → dokąd → którym portem → jak się uwierzytelniam**.

## ⚠️ Pułapka

Hasło wpisywane w terminalu nie pokazuje gwiazdek ani kropek. To normalne zachowanie, a nie zawieszenie terminala.

## 🛡️ Znaczenie w cyberbezpieczeństwie

SSH jest podstawowym narzędziem bezpiecznej administracji serwerami Linux.

---

[← Powrót do README](../README.md)
