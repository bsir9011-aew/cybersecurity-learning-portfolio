# Bandit 16 → 17 — skanowanie portów i identyfikacja usług

## 🎯 Cel

Przeskanować zakres portów, rozpoznać działające usługi, odróżnić zwykłe echo od TLS i znaleźć właściwy serwer.

## 🧠 Nowe pojęcie

Otwarty port mówi tylko, że coś nasłuchuje. Opcja wykrywania wersji i usługi pomaga ustalić, **co** działa za portem.

## 🧰 Poznane wzorce

```bash
nmap -sV -p <port-początkowy>-<port-końcowy> localhost
openssl s_client -connect localhost:<port-TLS> -quiet
```

- `-sV` — identyfikacja usługi i wersji,
- `-p` — zakres skanowanych portów.

## 🔎 Mój sposób myślenia

```text
skan → otwarte porty → klasyfikacja usług → odrzucenie echo → test TLS
```

Usługa echo zwraca dokładnie to, co otrzymała. Interesujący był port TLS zachowujący się inaczej i oczekujący poprawnych danych.

## ⚠️ Pułapka

Wynik Nmapa jest hipotezą opartą na odpowiedziach usługi. Ostatecznie trzeba ręcznie zweryfikować zachowanie portu.

Otrzymany klucz prywatny zapisuję dokładnie od nagłówka `BEGIN` do stopki `END`, bez dodatkowych komunikatów i bez BOM.

## 🛡️ Znaczenie w cyberbezpieczeństwie

To miniaturowy rekonesans: wykrywanie powierzchni ataku, klasyfikacja usług i ręczna walidacja wyników skanera.

---

[← Powrót do README](../README.md)
