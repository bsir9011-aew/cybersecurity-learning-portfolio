# Bandit 14 → 15 — Netcat, localhost i port

## 🎯 Cel

Połączyć się z usługą działającą lokalnie na określonym porcie i wysłać jej tekst.

## 🧠 Nowe pojęcie

- `localhost` oznacza bieżącą maszynę,
- port wskazuje konkretną usługę na tej maszynie,
- `nc` pozwala otworzyć proste połączenie TCP i przesyłać dane.

## 🧰 Poznany wzorzec

```bash
nc localhost <port>
```

Po połączeniu usługa oczekuje danych wpisanych na standardowe wejście.

## 🔎 Mój sposób myślenia

```text
host = gdzie działa usługa
port = która usługa
treść = co do niej wysyłam
```

## ⚠️ Pułapka

Klucz SSH służył wcześniej do uwierzytelnienia. Do usługi sieciowej wysyła się dane wymagane przez zadanie, a nie treść klucza.

## 🛡️ Znaczenie w cyberbezpieczeństwie

Netcat jest użyteczny do testowania dostępności portów, prostych protokołów i ręcznej komunikacji z usługą.

---

[← Powrót do README](../README.md)
