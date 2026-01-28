# Ripristino Clean Install Windows 11 (ARM64)

Guida tecnica per reinstallazione *bare-metal* di **Windows 11 ARM64**  
su dispositivi **Samsung Galaxy Book (Go / Edge / S)** con architettura ARM.

---

## Descrizione

Questo progetto contiene una guida dettagliata per il **ripristino completo**  
di Windows 11 ARM64 in scenari in cui:

- il disco **UFS è corrotto, sostituito o formattato**,  
- l’installer standard di Windows non riesce a completare l’installazione,  
- è necessario un deploy *bare-metal* tramite **WinPE e DISM**.

La procedura è pensata per tecnici e utenti esperti, e documenta tutti i passaggi:  
preparazione dell’ambiente ADK, iniezione driver, creazione WinPE, e setup finale.

---

## Requisiti principali

- PC Host Windows 10/11 x64  
- ISO ufficiale di **Windows 11 ARM64**  
- **Windows ADK** + **WinPE add-on**  
- Driver originali dal sito [Samsung Support](https://www.samsung.com/support)

---

## Avvio rapido

1. Clona o scarica questo repository:
   ```bash
   git clone https://github.com/<tuo-utente>/galaxybook-arm64-recovery.git
   ```
2. Apri la guida:
   ```
   Windows11_ARM64_BareMetal_Restore_Samsung.md
   ```
3. Segui le istruzioni passo-passo.

---

## Avvertenza

> Questa guida è destinata a personale tecnico esperto.  
> Un uso improprio può causare perdita di dati o danni permanenti al dispositivo.

---

## Licenza

Distribuito sotto licenza **MIT License** — vedi file [`LICENSE`](LICENSE).

---

© 2026 Flavio
