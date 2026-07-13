#  Cybersecurity Learning Portfolio

Repozytorium będzie dokumentować moją praktyczną naukę cyberbezpieczeństwa oraz rozwój kompetencji potrzebnych w pracy analityka SOC.

Nie będzie to zbiór samych notatek. Każdy dział będzie zawierał praktyczne laboratoria, własne analizy, komendy, filtry, zrzuty ekranu, pliki testowe oraz wnioski z wykonanych ćwiczeń.

##  Planowana struktura repozytorium

```text
cybersecurity-learning-portfolio/
│
├── 01-network-analysis/
├── 02-siem-and-log-analysis/
├── 03-linux-security/
├── 04-windows-security/
├── 05-incident-response/
├── 06-threat-detection/
├── 07-ctf-and-challenges/
├── 08-security-projects/
├── 09-cheatsheets/
├── templates/
└── assets/
```

---

##  01-network-analysis

Folder poświęcony analizie ruchu sieciowego i zrozumieniu działania komunikacji pomiędzy urządzeniami.

Będzie zawierał laboratoria dotyczące:

* DNS,
* TCP i UDP,
* TCP three-way handshake,
* HTTP i HTTPS,
* TLS,
* ICMP,
* adresów IP i portów,
* budowy pakietów,
* analizy ruchu w Wiresharku,
* filtrowania pakietów,
* wykrywania nietypowego ruchu,
* skanowania portów,
* komunikacji z podejrzanymi hostami.

Planowane narzędzia:

* Wireshark,
* tcpdump,
* Zeek,
* Suricata.

---

##  02-siem-and-log-analysis

Folder poświęcony analizie logów oraz pracy z systemami SIEM.

Będzie zawierał:

* podstawy wyszukiwania zdarzeń,
* analizę logów Windows i Linux,
* filtrowanie dużych zbiorów zdarzeń,
* wykrywanie nieudanych logowań,
* analizę prób brute force,
* korelację zdarzeń,
* budowanie zapytań,
* analizę alertów SOC,
* tworzenie prostych reguł detekcyjnych,
* dokumentowanie wyników analizy.

Planowane narzędzia:

* Splunk,
* Elastic Stack,
* Microsoft Sentinel,
* Wazuh.

---

##  03-linux-security

Folder poświęcony administracji i bezpieczeństwu systemów Linux.

Będzie zawierał laboratoria dotyczące:

* użytkowników i grup,
* uprawnień do plików,
* procesów i usług,
* logów systemowych,
* SSH,
* konfiguracji sieci,
* analizy aktywnych połączeń,
* harmonogramów zadań,
* bezpieczeństwa kont,
* automatyzacji w Bashu,
* wykrywania podejrzanych procesów i plików.

---

##  04-windows-security

Folder poświęcony analizie bezpieczeństwa systemów Windows.

Będzie zawierał:

* Windows Event Logs,
* Event Viewer,
* Sysmon,
* procesy i usługi,
* konta użytkowników,
* zdarzenia logowania,
* PowerShell,
* rejestr systemowy,
* zadania zaplanowane,
* mechanizmy persistence,
* analizę podejrzanych poleceń,
* podstawy Active Directory.

---

##  05-incident-response

Folder poświęcony analizie i obsłudze incydentów bezpieczeństwa.

Będzie zawierał przykładowe scenariusze:

* brute force,
* phishing,
* pobranie podejrzanego pliku,
* nietypowe logowanie,
* uruchomienie podejrzanego procesu,
* komunikacja z podejrzaną domeną,
* podejrzany ruch PowerShell,
* infekcja malware,
* wyciek danych.

Każdy scenariusz będzie dokumentowany według schematu:

```text
Alert
  ↓
Weryfikacja
  ↓
Zebranie dowodów
  ↓
Analiza
  ↓
Ocena ryzyka
  ↓
Decyzja
  ↓
Rekomendowane działania
```

---

##  06-threat-detection

Folder poświęcony wykrywaniu zagrożeń.

Będzie zawierał:

* reguły Sigma,
* reguły YARA,
* podstawy IDS i IPS,
* wskaźniki kompromitacji,
* analizę IOC,
* techniki i taktyki MITRE ATT&CK,
* tworzenie hipotez detekcyjnych,
* mapowanie alertów do MITRE ATT&CK,
* analizę zachowania atakującego,
* podstawy threat huntingu.

---

##  07-ctf-and-challenges

Folder z rozwiązaniami zadań praktycznych i CTF.

Będzie zawierał:

* OverTheWire Bandit,
* TryHackMe,
* Hack The Box,
* zadania z analizy logów,
* zadania sieciowe,
* ćwiczenia z Linuxa,
* podstawowe zadania forensic,
* własne miniwyzwania.

Każde rozwiązanie będzie opisywać:

* cel zadania,
* sposób rozumowania,
* wykorzystane komendy,
* napotkane problemy,
* końcowe rozwiązanie,
* najważniejszą lekcję.

Hasła, flagi i dane, których nie wolno publikować, nie będą umieszczane w repozytorium.

---

##  08-security-projects

Folder z większymi projektami praktycznymi.

Planowane projekty:

* analizator alertów SOC,
* dashboard zdarzeń bezpieczeństwa,
* parser logów,
* generator raportów z incydentów,
* skrypty wspierające analizę,
* proste środowisko laboratoryjne SOC,
* narzędzia do analizy plików i IOC,
* wizualizacje komunikacji sieciowej.

Każdy projekt będzie posiadał osobny README z opisem:

* problemu,
* zastosowanego rozwiązania,
* architektury,
* sposobu uruchomienia,
* przykładowego użycia,
* ograniczeń,
* dalszego planu rozwoju.

---

##  09-cheatsheets

Folder z krótkimi materiałami pomocniczymi.

Będzie zawierał między innymi:

* filtry Wiresharka,
* komendy Linux,
* komendy sieciowe,
* zapytania Splunk SPL,
* najważniejsze zdarzenia Windows,
* podstawowe porty i protokoły,
* komendy PowerShell,
* skróty MITRE ATT&CK,
* checklisty analityka SOC,
* schemat analizy alertu.

Cheatsheety będą służyć jako szybka pomoc podczas wykonywania laboratoriów.

---

##  templates

Folder z szablonami zapewniającymi spójny sposób dokumentowania pracy.

Planowane szablony:

* szablon laboratorium,
* szablon analizy pakietów,
* szablon analizy alertu,
* szablon raportu z incydentu,
* szablon reguły detekcyjnej,
* szablon projektu,
* szablon rozwiązania CTF.

---

##  assets

Folder na materiały używane w dokumentacji:

* diagramy,
* schematy,
* grafiki,
* zrzuty ekranu,
* obrazy do głównego README.

Pliki związane z konkretnym laboratorium będą przechowywane przede wszystkim w folderze danego laba.

---

##  Standard dokumentowania laboratoriów

Każdy lab będzie docelowo zawierał:

```text
lab-name/
├── README.md
├── screenshots/
├── captures/
├── logs/
├── scripts/
└── samples/
```

Nie każdy lab będzie wymagał wszystkich folderów.

README laboratorium będzie zawierał:

1. cel,
2. scenariusz,
3. środowisko,
4. zastosowane narzędzia,
5. wykonane kroki,
6. analizę wyników,
7. dowody praktyczne,
8. znaczenie dla cyberbezpieczeństwa,
9. własne wnioski.

---

##  Główna zasada repozytorium

Każdy materiał powinien odpowiadać na trzy pytania:

```text
Co analizowałem?
        ↓
Co wykonałem samodzielnie?
        ↓
Jakie wyciągnąłem wnioski?
```

Celem repozytorium jest pokazanie nie tylko znajomości pojęć, ale również umiejętności praktycznej analizy, rozwiązywania problemów i dokumentowania wykonanej pracy.

---

