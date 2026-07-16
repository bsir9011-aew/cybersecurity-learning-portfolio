# Bandit 3 → 4 — pliki ukryte

## 🎯 Cel

Odnaleźć plik ukryty we wskazanym katalogu.

## 🧠 Nowe pojęcie

W Linuksie plik jest ukryty, gdy jego nazwa zaczyna się od kropki. Zwykłe `ls` go nie pokaże.

## 🧰 Poznane polecenia

```bash
cd <katalog>
ls -la
cat .<ukryty-plik>
```

## 🔎 Mój sposób myślenia

Jeżeli katalog wygląda na pusty, sprawdzam go ponownie z opcją pokazującą wszystkie wpisy.

## ⚠️ Pułapka

`ls -l` pokazuje szczegóły, ale nie pokazuje plików ukrytych. Potrzebna jest opcja `-a`.

## 🛡️ Znaczenie w cyberbezpieczeństwie

Pliki konfiguracyjne, historia powłoki i część mechanizmów persistence znajdują się w ukrytych plikach użytkownika.

---

[← Powrót do README](../README.md)
