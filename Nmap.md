# NMAP

Host Discovery

```
nmap -sn 10.129.2.0/24
```

Sprawdza, które hosty są aktywne (bez skanowania portów).

```
nmap -sn -iL hosts.lst
```

Skanuje listę adresów IP z pliku.

```
nmap -sn 10.129.2.18-20
```

Skanuje zakres adresów IP.

```
nmap -PE --disable-arp-ping 10.129.2.18
```

Wysyła ICMP Echo Request (ping).

---

## Skanowanie portów TCP

```
nmap 10.129.2.28
```

Domyślnie skanuje 1000 najpopularniejszych portów TCP.

```
nmap --top-ports=10 10.129.2.28
```

Skanuje 10 najczęściej używanych portów.

```
nmap -F 10.129.2.28
```

Szybki skan 100 najpopularniejszych portów.

```
nmap -p 22,80,443 10.129.2.28
```

Skanuje wybrane porty.

```
nmap -p 1-1000 10.129.2.28
```

Skanuje zakres portów.

```
nmap -p- 10.129.2.28
```

Skanuje wszystkie 65535 portów.

---

## Typy skanowania

```
sudo nmap -sS 10.129.2.28
```

SYN Scan (Half-Open) – szybki i bardziej ukryty.

```
nmap -sT 10.129.2.28
```

TCP Connect Scan – pełne połączenie TCP.

```
sudo nmap -sU 10.129.2.28
```

Skanowanie portów UDP.

---

## Wykrywanie usług

```
nmap -sV 10.129.2.28
```

Wykrywa wersje usług.

```
nmap -A 10.129.2.28
```

Wykrywa wersje, system operacyjny, skrypty NSE i traceroute.

```
nmap -O 10.129.2.28
```

Próbuje wykryć system operacyjny.

---

## Przydatne opcje

```
-Pn
```

Pomija ping przed skanowaniem.

```
-n
```

Wyłącza DNS.

```
--disable-arp-ping
```

Wyłącza ARP Ping.

```
--packet-trace
```

Pokazuje wszystkie wysłane i odebrane pakiety.

```
--reason
```

Pokazuje powód wyniku (np. syn-ack, reset).

---

## Zapisywanie wyników

```
nmap -oN wynik.nmap
```

Zapis do zwykłego pliku.

```
nmap -oG wynik.gnmap
```

Zapis w formacie grep.

```
nmap -oX wynik.xml
```

Zapis w XML.

```
nmap -oA wynik
```

Zapis we wszystkich formatach (.nmap, .gnmap, .xml).

```
xsltproc wynik.xml -o wynik.html
```

Konwertuje XML do raportu HTML.

# Stany portów

- **open** – port jest otwarty, można się z nim połączyć.
- **closed** – port jest zamknięty, host odpowiada pakietem **RST**.
- **filtered** – firewall filtruje ruch, Nmap nie wie czy port jest otwarty.
- **unfiltered** – port jest dostępny, ale nie wiadomo czy jest otwarty (ACK Scan).
- **open|filtered** – brak odpowiedzi, port może być otwarty lub filtrowany.
- **closed|filtered** – nie można określić stanu (Idle Scan).

---

# Protokoły i flagi

### TCP

Protokół połączeniowy (3-way handshake).

### UDP

Protokół bezpołączeniowy, szybszy, ale trudniejszy do skanowania.

### ICMP

Protokół diagnostyczny (ping, komunikaty o błędach).

### ARP

Służy do poznania adresu MAC urządzenia w sieci lokalnej.

---

# Flagi TCP

### SYN

Rozpoczyna połączenie TCP.

### SYN-ACK

Odpowiedź oznaczająca otwarty port.

### ACK

Potwierdzenie odebrania pakietu.

### RST

Resetuje połączenie → port zamknięty.

### RA (RST + ACK)

Port zamknięty.

### SA (SYN + ACK)

Port otwarty.

NSE (**Nmap Scripting Engine**) to silnik skryptów wbudowany w Nmapa. Pozwala automatycznie wykonywać dodatkowe czynności na znalezionych portach, np.:

- pobierać bannery,
- wykrywać wersje usług,
- szukać podatności,
- testować logowanie,
- zbierać dodatkowe informacje.

Skrypty są napisane w języku **Lua**.

---

# Kategorie skryptów NSE

| Kategoria | Co robi |
| --- | --- |
| **auth** | Sprawdza metody uwierzytelniania i dane logowania. |
| **broadcast** | Wyszukuje hosty poprzez broadcast. |
| **brute** | Atakuje usługę metodą brute-force. |
| **default** | Domyślne skrypty uruchamiane przez `-sC`. |
| **discovery** | Zbiera informacje o usługach i hostach. |
| **dos** | Testuje podatność na DoS. |
| **exploit** | Próbuje wykorzystać znane podatności. |
| **external** | Korzysta z zewnętrznych serwisów. |
| **fuzzer** | Wysyła nietypowe pakiety w celu znalezienia błędów. |
| **intrusive** | Agresywne skrypty mogące wpływać na działanie systemu. |
| **malware** | Sprawdza obecność malware. |
| **safe** | Bezpieczne skrypty, nie ingerują w system. |
| **version** | Dokładniej wykrywa wersje usług. |
| **vuln** | Szuka znanych podatności (CVE). |

---

# Uruchamianie skryptów

## 1. Domyślne skrypty

```
nmap <IP> -sC
```

### Co robi?

Uruchamia zestaw bezpiecznych, domyślnych skryptów NSE.

Najczęściej:

- banner grabbing,
- informacje o HTTP,
- SMB,
- FTP,
- SSH,
- DNS.

---

## 2. Cała kategoria skryptów

```
nmap <IP> --script vuln
```

lub

```
nmap <IP> --script discovery
```

### Co robi?

Uruchamia wszystkie skrypty z wybranej kategorii.

---

## 3. Konkretne skrypty

```
nmap <IP> --script banner,smtp-commands
```

### Co robi?

Uruchamia tylko wskazane skrypty.

---

# Przykład

```
nmap 10.129.2.28 -p 25 --script banner,smtp-commands
```

## Parametry

| Parametr | Opis |
| --- | --- |
| `-p 25` | Skanuje tylko port 25 (SMTP). |
| `--script banner,smtp-commands` | Uruchamia dwa skrypty NSE. |

---

## Wynik

```
25/tcp open smtp

banner:
220 inlane ESMTP Postfix (Ubuntu)

smtp-commands:
PIPELINING
SIZE
VRFY
ETRN
STARTTLS
SMTPUTF8
```

### Co znaleziono?

### banner

Pokazuje banner serwera.

Dowiadujemy się:

- serwer SMTP,
- Postfix,
- Ubuntu.

To pomaga określić:

- system operacyjny,
- wersję usługi,
- potencjalne exploity.

---

### smtp-commands

Pokazuje komendy obsługiwane przez serwer SMTP.

Przykłady:

- **VRFY** – sprawdza, czy użytkownik istnieje.
- **STARTTLS** – obsługa szyfrowania TLS.
- **SIZE** – maksymalny rozmiar wiadomości.
- **PIPELINING** – możliwość wysyłania wielu komend naraz.

To przydaje się podczas enumeracji SMTP.

---

# Skan agresywny (-A)

```
nmap 10.129.2.28 -p 80 -A
```

## Co robi `A`?

Opcja **-A** uruchamia jednocześnie:

- `sV` → wykrywanie wersji usług,
- `O` → wykrywanie systemu operacyjnego,
- `sC` → domyślne skrypty NSE,
- `-traceroute` → wyznacza trasę do hosta.

---

## Wynik

```
80/tcp open http Apache 2.4.29

http-generator:
WordPress 5.3.4

http-server-header:
Apache/2.4.29

http-title:
blog.inlanefreight.com
```

### Co wykryto?

**Apache 2.4.29**

→ serwer [WWW](http://www/).

**WordPress 5.3.4**

→ aplikacja działająca na stronie.

**http-title**

→ tytuł strony.

**OS Detection**

```
Linux 96%
```

→ Nmap ocenia, że system to Linux z 96% pewnością.

**Traceroute**

Pokazuje liczbę przeskoków (hopów) do celu.

---

# Wyszukiwanie podatności

Możemy użyć kategorii **vuln**.

```
nmap 10.129.2.28 -p 80 -sV --script vuln
```

## Parametry

| Parametr | Opis |
| --- | --- |
| `-p 80` | Port HTTP. |
| `-sV` | Wykrywa wersję usługi. |
| `--script vuln` | Uruchamia skrypty szukające podatności. |

---

## Co wykryto?

### http-enum

```
/wp-login.php
/readme.html
/wp-admin/
```

Znajduje interesujące katalogi i pliki.

---

### WordPress

```
WordPress 5.3.4
```

Wykrywa wersję WordPressa.

---

### http-wordpress-users

```
admin
```

Odnajduje nazwy użytkowników WordPressa.

Może być wykorzystane później przy brute-force.

---

### vulners

```
Apache 2.4.29

CVE-2019-0211
CVE-2018-1312
CVE-2017-15715
```

Na podstawie wykrytej wersji usługi porównuje ją z bazą podatności i pokazuje pasujące **CVE (Common Vulnerabilities and Exposures)**.

---

# Najważniejsze komendy

```
nmap <IP> -sC
```

➡️ Domyślne skrypty NSE.

```
nmap <IP> --script vuln
```

➡️ Szukanie podatności.

```
nmap <IP> --script banner
```

➡️ Pobieranie bannera.

```
nmap <IP> --script smtp-commands
```

➡️ Pokazuje obsługiwane komendy SMTP.

```
nmap <IP> --script discovery
```

➡️ Enumeracja usług.

```
nmap <IP> --script brute
```

➡️ Próby logowania brute-force.

```
nmap <IP> --script auth
```

➡️ Sprawdza mechanizmy uwierzytelniania.

```
nmap <IP> -A
```

➡️ Agresywny skan (`-sV + -O + -sC + traceroute`).

---

# Co warto zapamiętać?

- **NSE** = silnik skryptów Nmapa do automatycznej enumeracji i testów.
- **`sC`** uruchamia bezpieczne, domyślne skrypty.
- **`-script <nazwa>`** uruchamia konkretny skrypt.
- **`-script <kategoria>`** uruchamia wszystkie skrypty z danej kategorii.
- **`banner`** pobiera informacje identyfikacyjne usługi.
- **`smtp-commands`** pokazuje dostępne komendy SMTP.
- **`vuln`** wyszukuje znane podatności (CVE).
- **`A`** to kompleksowy skan: wersje usług, system operacyjny, traceroute i domyślne skrypty NSE.

Wykrywanie wersji usług

```
nmap -p- -sV <IP>
```

➡️ Skanuje wszystkie porty i wykrywa wersje usług.

---

```
nmap -p- -sV --stats-every=5s <IP>
```

➡️ Pokazuje postęp skanowania co 5 sekund.

---

```
nmap -p- -sV -v <IP>
```

➡️ Tryb verbose – wyświetla znalezione porty na bieżąco.

---

## Opcje

- `-p-`

> Skanuje wszystkie 65535 portów.
> 
- `-sV`

> Wykrywa wersję usługi.
> 
- `-v`

> Więcej informacji podczas skanowania.
> 
- `-stats-every=5s`

> Pokazuje postęp skanowania co określony czas.
> 

---

# Banner Grabbing

**Banner** – wiadomość wysyłana przez usługę po nawiązaniu połączenia.

Może zawierać:

- nazwę usługi,
- wersję,
- system operacyjny,
- hostname.

Przykład:

```
220 inlane ESMTP Postfix (Ubuntu)
```

Dowiadujemy się:

- SMTP
- Postfix
- Ubuntu
- hostname: inlane

---

## Pobieranie banera

### Netcat

```
nc -nv <IP> 25
```

➡️ Łączy się z portem i wyświetla banner.

---

### Tcpdump

```
tcpdump -i eth0 host <IP>
```

➡️ Przechwytuje ruch sieciowy.

---

# Packet Trace

```
nmap -Pn -n --disable-arp-ping --packet-trace -sV <IP>
```

➡️ Pokazuje wszystkie wysłane i odebrane pakiety.

---

## Opcje

- `Pn`

> Pomija ping.
> 
- `n`

> Wyłącza DNS.
> 
- `-disable-arp-ping`

> Wyłącza ARP Ping.
> 
- `-packet-trace`

> Wyświetla pakiety.
> 

---

# Three-Way Handshake

```
SYN
```

➡️ Klient rozpoczyna połączenie.

---

```
SYN-ACK
```

➡️ Serwer akceptuje połączenie.

---

```
ACK
```

➡️ Klient potwierdza.

Połączenie zostało nawiązane.

---

# Flagi TCP

### SYN

Rozpoczęcie połączenia.

### SYN-ACK

Odpowiedź serwera – port otwarty.

### ACK

Potwierdzenie odebrania.

### PSH

Serwer wysyła dane do klienta (np. banner).

### PSH-ACK

Serwer wysyła dane i potwierdza wcześniejsze pakiety.

### RST

Reset połączenia (port zamknięty).

---

# Jak działa Banner Grabbing?

1. Klient wykonuje **TCP Three-Way Handshake**.
2. Serwer wysyła **banner** (PSH-ACK).
3. Klient odbiera dane i odsyła ACK.

Schemat:

```
Client                Server

SYN      ---------->
         <---------- SYN-ACK
ACK      ---------->
         <---------- PSH-ACK (Banner)
ACK      ---------->
```

---

# Przykładowy wynik Nmap

```
22/tcp  OpenSSH 7.6p1
25/tcp  Postfix smtpd
80/tcp  Apache 2.4.29
110/tcp Dovecot pop3
143/tcp Dovecot imap
993/tcp Dovecot ssl/imap
995/tcp Dovecot ssl/pop3
```

| Narzędzie | Do czego służy? | Co robi? |
| --- | --- | --- |
| **nc (netcat)** | Łączenie się z usługą | Wysyła i odbiera dane z portu. Możesz zobaczyć banner serwera lub ręcznie wysyłać komendy. |
| **tcpdump** | Podsłuchiwanie ruchu sieciowego | Przechwytuje wszystkie pakiety przechodzące przez kartę sieciową i pokazuje ich zawartość. |

**1. Timeouty**

- `-initial-rtt-timeout 50ms` — początkowy timeout.
- `-max-rtt-timeout 100ms` — maksymalny timeout.
- Za małe wartości → szybciej, ale możesz pominąć hosty/porty.

```
sudo nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms
```

**2. Ponowienia**

- `-max-retries 0` — brak ponowień → szybszy skan, ale mniej dokładny.

```
sudo nmap 10.129.2.0/24 -F --max-retries 0
```

**3. Rate**

- `-min-rate 300` — minimum 300 pakietów/s.

```
sudo nmap 10.129.2.0/24 -F --min-rate 300
```

**4. Timing**

- `T3` — normalny, domyślny
- `T4` — agresywny
- `T5` — bardzo agresywny

```
sudo nmap 10.129.2.0/24 -F -T4
```

- `-min-rate` = więcej pakietów/s
- `-max-retries 0` = mniej ponowień

`RTT timeout` = jak długo Nmap czeka na odpowiedź

`F` w Nmap oznacza **Fast Scan**.