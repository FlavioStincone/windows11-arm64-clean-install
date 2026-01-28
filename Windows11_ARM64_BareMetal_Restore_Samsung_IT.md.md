# GUIDA TECNICA

## Ripristino Clean Install Windows 11 (ARM64)  
### Samsung Galaxy Book (Go / Edge / S)

**Oggetto:**  
Procedura di ripristino *bare-metal* per dispositivi Samsung con architettura **ARM64**,  
in scenari di **storage UFS corrotto, sostituito o completamente formattato**,  
nei quali l’installer standard di Windows non riesce a completare l’installazione.

> **ATTENZIONE – GUIDA AVANZATA**  
> Questa procedura è destinata esclusivamente a tecnici o utenti esperti.  
> Richiede familiarità con **DISM, WinPE, partizionamento manuale e UEFI**.  
> Un uso improprio può causare **perdita totale dei dati**.

**Versione documento:** 2.3  
**Data:** 28/01/2026

---

## 1. PRE-REQUISITI E STRUMENTAZIONE

È necessaria una workstation di supporto (**PC Host**) con Windows funzionante.

### 1.1 Hardware richiesto

- **PC Host:** Windows 10/11 x64  
- **Target PC:** Samsung Galaxy Book ARM (Go, Edge, S, ecc.)  
- **USB 1 – Boot (WinPE):** ≥ 8 GB *(verrà formattata)*  
- **USB 2 – Dati:** ≥ 16 GB, **NTFS**  
- **Accessori consigliati:**
  - Mouse USB cablato  
  - Smartphone Android (tethering USB) o adattatore LAN USB

---

### 1.2 Software da scaricare sul PC Host

1. **Windows ADK – Windows 11**  
   - Installare il componente **Deployment Tools**
2. **Windows PE add-on per ADK**  
   - Installare **dopo** l’ADK
3. **ISO ufficiale Windows 11 ARM64**  
   - Download dal sito **Microsoft** (sezione *Download Windows 11 on ARM64 devices*)

---

### 1.3 Reperimento driver (**PASSAGGIO CRITICO**)

I driver non sono inclusi nella ISO ARM64 generica.  
È necessario scaricarli dal **sito ufficiale di supporto Samsung**:

1. Visitare [https://www.samsung.com/support](https://www.samsung.com/support)  
2. Cercare il modello esatto del dispositivo (es. `NP740XME-KA1IT`)  
3. Scaricare il **Driver Pack** o **System Software Installer**
4. Estrarre e organizzare i driver come segue:

```text
C:\Drivers_WinPE   -> Storage (UFS) / Chipset / USB  (ESSENZIALI)
C:\Drivers         -> Pacchetto driver completo
```

> Senza i driver **UFS / USB**, il disco **non sarà visibile** in WinPE.

---

## 2. INIZIALIZZAZIONE AMBIENTE ADK (PC HOST)

Prima di usare DISM, aprire un **Prompt dei comandi come Amministratore** ed eseguire:

```cmd
cd "C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Deployment Tools"
DandISetEnv.bat
```

Verificare che compaia:
```
Environment variables have been set
```

---

## 3. PREPARAZIONE IMMAGINE WINDOWS (USB DATI)

### 3.1 `install.wim`

1. Montare la ISO ARM64  
2. Copiare da `sources` → `install.wim` o `install.esd`  
3. Se il file è `.esd`, convertirlo in `.wim`:

```cmd
dism /export-image ^
 /sourceimagefile:C:\wim\install.esd ^
 /sourceindex:1 ^
 /destinationimagefile:C:\wim\install.wim ^
 /compress:max /checkintegrity
```

---

### 3.2 Verifica edizione e indice

```cmd
dism /get-wiminfo /wimfile:C:\wim\install.wim
```

> Scegliere l’indice corrispondente all’edizione desiderata (es. Pro, Home, Education).

---

### 3.3 Iniezione driver

```cmd
md C:\wimmount
dism /mount-wim /wimfile:C:\wim\install.wim /index:1 /mountdir:C:\wimmount
dism /image:C:\wimmount /add-driver /driver:C:\Drivers /recurse /forceunsigned
dism /unmount-wim /mountdir:C:\wimmount /commit
```

Copiare il file `install.wim` modificato nella **radice della USB 2**.

---

### 3.4 Script di automazione

#### `CreatePartitions.txt`
```text
select disk 0
clean
convert gpt
create partition efi size=260
format quick fs=fat32 label="System"
assign letter=S
create partition msr size=128
create partition primary
format quick fs=ntfs label="Windows"
assign letter=W
```

#### `ApplyImage.bat`
```bat
@echo off
set INDEX=%2
if "%INDEX%"=="" set INDEX=1
echo [INFO] Applicazione immagine Windows...
dism /apply-image /imagefile:%1 /index:%INDEX% /applydir:W:\
echo [INFO] Creazione bootloader UEFI...
W:\Windows\System32\bcdboot W:\Windows /s S:
echo [SUCCESS] Installazione completata.
pause
```

---

## 4. CREAZIONE USB WINPE (BOOT)

```cmd
Dism /Cleanup-Wim
rd /s /q C:\WinPE_arm64

copype arm64 C:\WinPE_arm64

dism /mount-image ^
 /imagefile:"C:\WinPE_arm64\media\sources\boot.wim" ^
 /index:1 ^
 /mountdir:"C:\WinPE_arm64\mount"

dism /image:"C:\WinPE_arm64\mount" ^
 /add-driver /driver:"C:\Drivers_WinPE" /recurse /forceunsigned

dism /unmount-image /mountdir:"C:\WinPE_arm64\mount" /commit

MakeWinPEMedia /UFD C:\WinPE_arm64 E:
```

> Sostituire `E:` con la lettera della chiavetta **USB 1 (WinPE)**.

---

## 5. INSTALLAZIONE SUL DISPOSITIVO TARGET

### 5.1 Impostazioni UEFI / BIOS

- Disattivare **Secure Boot** e **Fast Boot**  
- Impostare priorità di boot → **USB (WinPE)**  
- Salvare con **F10**

---

### 5.2 Installazione da WinPE

```cmd
diskpart /s D:\CreatePartitions.txt
D:\ApplyImage.bat D:\install.wim 1
```

Al termine:
```cmd
exit
```
Rimuovere le USB e riavviare.

---

## 6. CONFIGURAZIONE POST-INSTALLAZIONE

### 6.1 Bypass rete (solo Windows 11)

```cmd
Shift + F10
OOBE\BYPASSNRO
```

### 6.2 Ripristino completo

- Connettere Internet (tethering USB consigliato)  
- Aggiornare tramite **Windows Update**  
- Verificare funzionamento di:
  - UFS / eMMC Storage  
  - Touchpad e tastiera  
  - Sleep / standby

---

## FINE PROCEDURA

---

## DISCLAIMER E LICENZA

Questa guida è fornita "così com’è", senza alcuna garanzia.  
L’autore non è responsabile per eventuali danni, perdita di dati o malfunzionamenti.

Distribuita sotto licenza **MIT License** (vedi file `LICENSE` nel repository).

---