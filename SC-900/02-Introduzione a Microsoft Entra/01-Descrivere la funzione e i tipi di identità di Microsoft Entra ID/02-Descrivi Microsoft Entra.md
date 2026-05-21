# Microsoft Entra – Famiglia di prodotti per identità e accesso

## Introduzione

Microsoft Entra è una famiglia di soluzioni Microsoft dedicate alla gestione delle identità e degli accessi.

È progettata per supportare un modello di sicurezza **Zero Trust**, in cui:
- ogni accesso viene verificato
- nessuna identità è automaticamente affidabile
- sicurezza e controllo sono basati su identità, contesto e rischio

---

## 1. Obiettivo della famiglia Microsoft Entra

Microsoft Entra protegge l’accesso a:

- dipendenti
- clienti
- partner
- applicazioni e carichi di lavoro
- agenti di intelligenza artificiale

👉 In qualsiasi ambiente:
- cloud
- on-premises
- ibrido

---

## 2. Struttura della famiglia Microsoft Entra

I prodotti Entra sono organizzati per scenari di sicurezza:

### 1. Controlli Zero Trust di base
- identità
- autenticazione
- servizi directory

---

### 2. Accesso sicuro per i dipendenti
- governance delle identità
- protezione accessi
- rete sicura
- gestione credenziali verificate

---

### 3. Accesso per clienti e partner
- identità esterne (CIAM)
- collaborazione B2B
- gestione utenti guest

---

### 4. Accesso in qualsiasi cloud
- protezione di applicazioni e workload
- identità per servizi e API

---

### 5. Accesso per agenti AI
- identità per sistemi di intelligenza artificiale
- governance e sicurezza degli agenti non umani

---

## 3. Principali prodotti Microsoft Entra

---

## Microsoft Entra ID (fondamentale)

È il servizio principale della famiglia Entra.

### Funzioni principali:
- autenticazione utenti
- Single Sign-On (SSO)
- gestione accessi
- applicazione policy di sicurezza
- protezione di utenti, dispositivi e app

👉 È la base di Microsoft 365, Azure e Dynamics.

---

## Microsoft Entra Domain Services

- fornisce servizi Active Directory gestiti nel cloud
- supporta applicazioni legacy che richiedono AD tradizionale
- elimina la necessità di gestire controller di dominio

---

## 4. Protezione accesso per i dipendenti

---

### Microsoft Entra Private Access
- accesso sicuro a risorse private
- sostituisce la VPN
- connessione sicura da qualsiasi rete o dispositivo

---

### Microsoft Entra Internet Access
- protegge accesso a Internet e SaaS
- include filtri web e controllo contenuti

---

### Microsoft Entra ID Governance
- automatizza gestione utenti e accessi
- provisioning e deprovisioning automatico
- gestione ciclo di vita identità

---

### Microsoft Entra ID Protection
- rileva rischi basati sull’identità
- accessi sospetti e utenti a rischio
- applica MFA e accesso condizionale basato sul rischio

---

### Microsoft Entra Verified ID
- credenziali verificabili (identità decentralizzata)
- documenti digitali firmati e verificabili
- esempio: diploma digitale o certificazioni

---

## 5. Accesso clienti e partner

---

### Microsoft Entra External ID

Consente accesso sicuro per utenti esterni:

- partner aziendali (B2B)
- clienti (CIAM)
- utenti guest

### Funzionalità:
- registrazione self-service
- login con social account (Google, Facebook, ecc.)
- accesso sicuro a risorse aziendali

---

## 6. Accesso ai workload (app e servizi)

---

### Microsoft Entra Workload ID

Gestisce identità non umane:

- applicazioni
- API
- container
- servizi automatizzati

### Esempio:
- GitHub Actions che accede ad Azure

👉 Permette accesso sicuro tra servizi senza utenti umani.

---

## 7. Accesso per agenti di intelligenza artificiale

---

### Microsoft Entra Agent ID

Gestisce identità degli agenti AI:

- agenti autonomi
- agenti assistivi
- sistemi intelligenti

### Funzioni:
- identità dedicate per ogni agente
- controllo accessi granulari
- audit delle azioni
- applicazione del principio del least privilege

---

## 8. Come lavorano insieme i prodotti Entra

Esempio: onboarding di un nuovo dipendente

- **Entra ID** → autentica e abilita SSO
- **Entra ID Governance** → assegna automaticamente accessi
- **Entra ID Protection** → controlla rischio accessi
- **Entra Internet Access** → protegge traffico verso cloud e Internet
- **Entra Private Access** → accesso sicuro alle risorse interne senza VPN

👉 Risultato: sicurezza integrata end-to-end

---

## 9. Licenze Microsoft Entra

---

### Entra ID Free
- gestione base utenti e gruppi
- report semplici
- self-service password reset

---

### Entra ID P1
- accesso condizionale
- identità ibride
- gestione avanzata gruppi
- incluso in Microsoft 365 E3

---

### Entra ID P2
- risk-based access
- Identity Protection
- Privileged Identity Management (PIM)
- incluso in Microsoft 365 E5

---

### Entra Suite
Include:
- Private Access
- Internet Access
- ID Governance
- ID Protection
- Verified ID (premium)

---

## 10. Interfaccia di gestione

Tutti i prodotti Microsoft Entra sono gestiti tramite:

> **Microsoft Entra Admin Center**

Un portale unico web per configurazione e controllo.

---

## 11. Sintesi finale

- Microsoft Entra è la famiglia di prodotti per identità e accesso
- Supporta modello Zero Trust
- Protegge utenti, dispositivi, applicazioni, workload e agenti AI
- Include soluzioni per dipendenti, clienti, partner e sistemi automatizzati
- Entra ID è il servizio centrale della piattaforma
- I prodotti lavorano insieme per sicurezza end-to-end
