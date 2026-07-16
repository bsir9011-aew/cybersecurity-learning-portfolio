# Bandit 10 → 11 — dekodowanie Base64

## 🎯 Cel

Rozpoznać i zdekodować dane zapisane w Base64.

## 🧠 Nowe pojęcie

Base64 to **kodowanie**, nie szyfrowanie. Zamienia bajty na zestaw znaków wygodny do przesyłania w formie tekstowej, ale nie zapewnia poufności.

## 🧰 Poznane polecenie

```bash
base64 -d <plik>
```

## 🔎 Mój sposób myślenia

Najpierw identyfikuję sposób reprezentacji danych, a potem używam zgodnego dekodera.

## ⚠️ Pułapka

Dane wyglądające losowo nie muszą być zaszyfrowane. Często są tylko zakodowane lub skompresowane.

## 🛡️ Znaczenie w cyberbezpieczeństwie

Base64 często pojawia się w poczcie, HTTP, PowerShellu i payloadach. Analityk powinien szybko rozpoznać, że nie jest to mechanizm ochrony.

---

[← Powrót do README](../README.md)
