# Bandit 15 → 16 — szyfrowane połączenie TLS

## 🎯 Cel

Wysłać dane do lokalnej usługi przez kanał zabezpieczony SSL/TLS.

## 🧠 Nowe pojęcie

`nc` tworzy zwykłe połączenie TCP. `openssl s_client` potrafi przeprowadzić handshake TLS i komunikować się z usługą szyfrowaną.

## 🧰 Poznany wzorzec

```bash
openssl s_client -connect localhost:<port> -quiet
```

- `s_client` — testowy klient TLS,
- `-connect` — host i port,
- `-quiet` — ograniczenie komunikatów diagnostycznych.

## 🔎 Mój sposób myślenia

Najpierw ustalam, czy usługa wymaga zwykłego TCP, czy TLS. Narzędzie musi mówić tym samym protokołem co serwer.

## ⚠️ Pułapka

Polecenie należy wykonywać na serwerze Bandita, gdzie dostępny jest OpenSSL. Lokalny CMD Windows może nie mieć programu `openssl`.

## 🛡️ Znaczenie w cyberbezpieczeństwie

`openssl s_client` pomaga analizować certyfikaty, handshake, wersję TLS i zachowanie szyfrowanych usług.

---

[← Powrót do README](../README.md)
