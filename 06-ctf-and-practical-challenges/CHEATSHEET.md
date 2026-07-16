# Bandit 0–19 — ściąga z poznanych mechanizmów 🧰

## SSH i transfer plików

```bash
ssh <user>@<host> -p <port>
ssh -i <private-key> <user>@<host> -p <port>
scp -P <port> <user>@<host>:<remote-file> .
ssh <user>@<host> -p <port> "<remote-command>"
```

## Pliki i katalogi

```bash
pwd
ls -la
cd <directory>
cat <file>
cat -- "<difficult filename>"
file <file>
```

## Wyszukiwanie

```bash
grep "<text>" <file>
find <path> -type f
find <path> -type f -size <bytes>c
find / -type f -user <user> -group <group> 2>/dev/null
```

## Przetwarzanie tekstu

```bash
sort <file> | uniq -u
strings <file> | grep "<marker>"
base64 -d <file>
tr 'A-Za-z' 'N-ZA-Mn-za-m'
diff <old-file> <new-file>
```

## Archiwa i formaty

```bash
xxd -r <hexdump> <output>
gzip -d <file.gz>
bzip2 -d <file.bz2>
tar -xf <archive.tar>
```

## Sieć i TLS

```bash
nc localhost <port>
openssl s_client -connect localhost:<port> -quiet
nmap -sV -p <start>-<end> localhost
```

## Uprawnienia

```bash
ls -l
id
whoami
./<suid-binary> whoami
```

## Uniwersalny schemat diagnostyczny

```text
1. Sprawdź, gdzie jesteś i jako kto działasz.
2. Wylistuj pliki wraz z uprawnieniami.
3. Ustal typ danych lub usługę.
4. Dobierz narzędzie do rozpoznanego typu.
5. Po każdym kroku zweryfikuj wynik.
```
