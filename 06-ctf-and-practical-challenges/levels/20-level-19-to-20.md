# Bandit 19 → 20 — SUID i efektywne uprawnienia

## 🎯 Cel

Użyć programu z ustawionym bitem SUID do wykonania polecenia z uprawnieniami właściciela programu.

## 🧠 Nowe pojęcie

Normalnie proces działa z uprawnieniami użytkownika, który go uruchomił. Program SUID może działać z **efektywnym UID właściciela pliku wykonywalnego**.

To nie jest klucz do jednego konkretnego pliku. Jest to upoważniony pośrednik, który wykonuje przekazane polecenie jako inny użytkownik.

## 🧰 Poznane wzorce

```bash
ls -l
./<program-suid> whoami
./<program-suid> <polecenie>
```

W uprawnieniach bit SUID może wyglądać tak:

```text
-rwsr-x---
   ^
   s
```

## 🔎 Mój sposób myślenia

```text
użytkownik A
    ↓ uruchamia
program należący do użytkownika B z SUID
    ↓
polecenie działa z efektywnymi uprawnieniami B
```

Polecenie `whoami` pozwoliło potwierdzić, jako kto działa proces.

## ⚠️ Pułapka

SUID dotyczy wykonywalnego programu, a nie automatycznie każdego pliku w katalogu. Znaczenie ma właściciel programu i sposób, w jaki program obsługuje przekazane argumenty.

## 🛡️ Znaczenie w cyberbezpieczeństwie

Źle zaprojektowane programy SUID mogą umożliwiać eskalację uprawnień. Podczas audytu należy sprawdzać m.in. listę plików SUID i możliwość nadużycia ich funkcji.

---

[← Powrót do README](../README.md)
