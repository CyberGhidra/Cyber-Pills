# Tipi di identità in Microsoft Entra ID

## Introduzione

Microsoft Entra ID supporta diversi tipi di identità per gestire in modo sicuro accessi e autorizzazioni.

Le principali categorie sono:

- Identità utente (persone)
- Identità del carico di lavoro (software e servizi)
- Identità dei dispositivi (hardware)
- Identità degli agenti (AI)
- Identità dei gruppi (organizzazione logica)

---

## 1. Identità utente

Le identità utente rappresentano persone:

- dipendenti
- clienti
- partner
- consulenti
- fornitori

### Tipi di autenticazione

Un utente può autenticarsi:

- **internamente** → account nel tenant dell’organizzazione
- **esternamente** → tramite altro provider (es. Microsoft, Google, social login)

---

### Tipi di utente (UserType)

#### Membro interno
- tipico dipendente
- account nel tenant aziendale
- accesso completo alle risorse interne

#### Guest esterno
- utenti esterni (B2B)
- accesso limitato
- autenticazione tramite provider esterno

#### Membro esterno
- utenti di altri tenant aziendali
- usano le proprie credenziali ma hanno ruolo da “membro” nel tenant ospitante

#### Guest interno (legacy)
- account creati internamente per utenti esterni
- oggi sostituiti dal modello B2B moderno

---

## 2. Identità del carico di lavoro

Rappresentano identità non umane:

- applicazioni
- servizi
- container
- VM

### Componenti principali

#### Service Principal
- identità di un’applicazione in un tenant
- consente autenticazione e autorizzazione verso risorse Azure/Entra

#### Applicazioni registrate
- definiscono l’applicazione a livello globale
- ogni utilizzo in un tenant crea un service principal

---

### Identità gestite (Managed Identities)

Servono per evitare gestione manuale delle credenziali.

#### Tipi:

- **System-assigned**
  - legata a una risorsa (es. VM)
  - viene eliminata con la risorsa

- **User-assigned**
  - creata indipendentemente
  - assegnabile a più risorse
  - non viene eliminata automaticamente

👉 Vantaggio principale: niente password o secret da gestire.

---

## 3. Identità degli agenti (AI)

Con l’aumento dell’intelligenza artificiale:

- ogni agente può avere una propria identità

### Microsoft Entra Agent ID

Permette di:

- registrare agenti AI
- applicare policy di sicurezza
- gestire ciclo di vita
- tracciare attività e responsabilità

### Tipi di utilizzo:

- agenti assistiti (per conto dell’utente)
- agenti autonomi (esecuzione indipendente)

---

## 4. Identità dei dispositivi

Rappresentano hardware come:

- PC
- smartphone
- tablet
- server
- stampanti

---

### Tipologie di dispositivi

#### Entra registered
- dispositivi personali (BYOD)
- accesso limitato alle risorse aziendali

#### Entra joined
- dispositivi aziendali
- login con account aziendale

#### Hybrid Entra joined
- uniti sia ad Active Directory locale che a Entra ID
- tipici ambienti enterprise ibridi

---

### Vantaggi

- Single Sign-On (SSO)
- controllo accessi basato sul dispositivo
- gestione tramite strumenti come Microsoft Intune

---

## 5. Gruppi

I gruppi semplificano la gestione degli accessi.

### Tipi di gruppi

#### Gruppi di sicurezza
- gestiscono accessi a risorse
- includono utenti, dispositivi, app e service principal
- supportano anche gruppi nidificati

#### Gruppi Microsoft 365
- usati per collaborazione
- includono:
  - email condivise
  - calendari
  - SharePoint
  - Teams

---

### Tipi di assegnazione membri

- **Assegnazione manuale**
- **Dynamic membership**
  - regole automatiche per aggiungere/rimuovere utenti

---

## 6. Sintesi finale

Microsoft Entra ID gestisce identità diverse per un modello Zero Trust:

- persone (utenti)
- software (workload)
- hardware (dispositivi)
- intelligenze artificiali (agenti)
- gruppi (logica organizzativa)

👉 Tutte queste identità devono essere:
- autenticate
- autorizzate
- monitorate
- governate

---

## Conclusione

La gestione moderna delle identità in Microsoft Entra ID non riguarda solo gli utenti, ma ogni entità che accede a una risorsa, rendendo il sistema scalabile, sicuro e adatto a cloud, ibrido e AI.
