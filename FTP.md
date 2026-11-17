# FTP

# Atakowanie FTP

## Podstawy

FTP (File Transfer Protocol) — transfer plików między komputerami. Domyślnie port **TCP/21**.

---

## Rekonesans — Nmap

```bash
sudo nmap -sC -sV -p 21 192.168.2.142
```

- `sC` — uruchamia skrypt `ftp-anon` (sprawdza anonymous login)
- `sV` — wersja usługi i banner FTP

---

## Anonymous Login

```bash
ftp 192.168.2.142
# User: anonymous
# Password: (puste)
```

### Komendy po zalogowaniu

| Komenda | Opis |
| --- | --- |
| `ls` | Lista plików |
| `cd katalog` | Zmiana katalogu |
| `get plik` | Pobierz jeden plik |
| `mget *` | Pobierz wiele plików |
| `put plik` | Wyślij jeden plik |
| `mput *` | Wyślij wiele plików |
| `help` | Pomoc |

> **Zagrożenie:** Jeśli anonymous ma uprawnienia zapisu — można wgrać złośliwy skrypt i wykonać go np. przez path traversal w aplikacji webowej.
> 

---

## Brute-force — Medusa

```bash
medusa -u fiona -P /usr/share/wordlists/rockyou.txt -h 10.129.203.7 -M ftp
```

| Opcja | Opis |
| --- | --- |
| `-u` | Jeden użytkownik |
| `-U` | Plik z listą użytkowników |
| `-P` | Plik z hasłami |
| `-M` | Protokół (ftp, ssh, smb...) |
| `-h` | Adres celu |

> **Uwaga:** Skuteczniejszy niż brute-force jest **Password Spraying** — wiele aplikacji blokuje brute-force.
> 

---

## FTP Bounce Attack

Używasz publicznego serwera FTP jako proxy do skanowania wewnętrznych maszyn przez komendę `PORT`.

**Scenariusz:**

- `FTP_DMZ` — wystawiony na internet
- `Internal_DMZ` — niedostępny z zewnątrz

Kazujesz `FTP_DMZ` wysyłać pakiety do `Internal_DMZ` — FTP robi skan za Ciebie.

```bash
nmap -b ftp_user:pass@ftp_server internal_target
```