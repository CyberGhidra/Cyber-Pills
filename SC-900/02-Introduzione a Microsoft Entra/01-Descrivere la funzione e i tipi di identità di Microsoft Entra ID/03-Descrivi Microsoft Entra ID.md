# Microsoft Entra ID

## Introduzione

Microsoft Entra ID è il servizio cloud di gestione delle identità e degli accessi di Microsoft.

Serve per controllare in modo sicuro l’accesso a risorse interne ed esterne da parte di:

- dipendenti
- utenti guest
- dispositivi e carichi di lavoro
- agenti di intelligenza artificiale

---

## 1. Cosa permette di fare Microsoft Entra ID

Microsoft Entra ID consente l’accesso sicuro a diverse tipologie di risorse:

### Risorse interne
- applicazioni aziendali on-premises
- intranet aziendale
- applicazioni cloud sviluppate internamente

### Servizi esterni
- Microsoft 365
- portale Azure
- applicazioni SaaS (Software as a Service)

---

## 2. Obiettivo principale

Microsoft Entra ID fornisce un sistema centralizzato per:

- autenticazione
- autorizzazione
- gestione degli accessi

👉 Un’unica identità per accedere a risorse cloud e locali.

---

## 3. Integrazione con altri sistemi

Microsoft Entra ID può essere utilizzato in tre modalità:

- come servizio autonomo (cloud-native)
- sincronizzato con Active Directory locale
- integrato con altri servizi di directory

---

## 4. Supporto per dispositivi e collaborazione

Microsoft Entra ID permette di:

- usare dispositivi personali (BYOD)
- gestire accessi da mobile e tablet
- collaborare con partner esterni
- abilitare accesso sicuro per clienti e guest

---

## 5. Identity Security Score

### Cos’è

È una metrica che misura il livello di sicurezza delle identità.

---

### Caratteristiche

- rappresentato come percentuale
- confronta la configurazione con le best practice Microsoft
- fornisce azioni di miglioramento personalizzate

---

### Utilità

- valutare il livello di sicurezza
- identificare vulnerabilità
- monitorare miglioramenti nel tempo

---

## 6. Concetti fondamentali

### Tenant

Un **tenant** è l’istanza di Microsoft Entra ID dedicata a un’organizzazione.

Contiene:
- utenti
- gruppi
- dispositivi
- applicazioni
- policy di sicurezza

👉 Ogni tenant ha:
- un ID unico
- un dominio (es. contoso.onmicrosoft.com)

---

### Directory

La directory è il contenuto logico del tenant.

Include:
- identità
- gruppi
- applicazioni
- dispositivi

👉 In pratica:
- tenant = contenitore organizzativo
- directory = database delle identità

---

### Multi-tenant

Un’organizzazione può avere più tenant per motivi come:

- aziende con filiali indipendenti
- fusioni o acquisizioni
- requisiti normativi geografici
- separazione di ambienti

---

## 7. Chi utilizza Microsoft Entra ID

---

### Amministratori IT
- gestione accessi e sicurezza
- configurazione MFA e policy
- controllo centralizzato delle identità

---

### Sviluppatori
- integrazione SSO nelle applicazioni
- uso di API e standard di autenticazione
- accesso ai dati aziendali

---

### Utenti aziendali
- accesso semplice e sicuro alle risorse
- uso di un’unica identità per più servizi

---

## 8. Microsoft Entra ID e sicurezza

Gli amministratori possono:

- applicare MFA (Multi-Factor Authentication)
- definire criteri di accesso condizionale
- proteggere identità e credenziali
- automatizzare la governance degli accessi

---

## 9. Microsoft Entra ID e Intelligenza Artificiale

Con la crescita degli agenti AI, Microsoft Entra ID si estende con:

### Microsoft Entra Agent ID

Consente di:

- assegnare identità agli agenti AI
- autenticare e autorizzare agenti autonomi
- applicare policy di sicurezza Zero Trust
- monitorare e tracciare le attività degli agenti

---

### Importanza

Gli agenti AI diventano entità a tutti gli effetti e devono essere:

- controllati
- autenticati
- governati
- monitorati

---

## 10. Sintesi finale

- Microsoft Entra ID è il servizio cloud di identità di Microsoft
- Centralizza autenticazione e autorizzazione
- Supporta risorse cloud, on-premises e SaaS
- Funziona con tenant e directory per organizzare le identità
- Include strumenti di sicurezza come Identity Security Score
- È usato da IT, sviluppatori e utenti finali
- Supporta anche identità per agenti AI tramite Entra Agent ID

---

## Conclusione

Microsoft Entra ID è la base della gestione moderna delle identità: un sistema cloud centralizzato che permette accesso sicuro, scalabile e controllato a tutte le risorse aziendali, includendo anche dispositivi, workload e agenti intelligenti in un modello Zero Trust.
