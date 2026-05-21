# Microsoft Entra External ID (Identità esterne)

## Introduzione

Microsoft Entra External ID è un insieme di funzionalità che permette alle organizzazioni di gestire in modo sicuro l’accesso di utenti esterni.

Serve per supportare la collaborazione con:

- partner aziendali
- clienti
- consumatori
- utenti guest

👉 Obiettivo: accesso sicuro senza compromettere i dati aziendali.

---

## 1. Scenari principali delle identità esterne

Microsoft Entra External ID copre due scenari principali:

- Collaborazione con utenti guest (B2B)
- Accesso per clienti e consumatori (CIAM)

---

## 2. Tipi di tenant

### Tenant della forza lavoro

Usato per:

- dipendenti
- app aziendali interne
- collaborazione con utenti guest

👉 Consente anche accesso esterno tramite inviti B2B.

---

### Tenant esterno

Usato per:

- applicazioni rivolte a clienti o consumatori
- gestione identità esterne (CIAM)

👉 Pensato per app pubbliche e utenti non aziendali.

---

## 3. Collaborazione B2B (Business-to-Business)

La collaborazione B2B permette di invitare utenti esterni nel tenant aziendale.

### Come funziona

- l’utente usa la propria identità (Google, Microsoft, aziendale)
- non crea nuove credenziali
- viene aggiunto come **guest nel tenant**

---

### Quando usarlo

- condivisione di app Microsoft 365
- accesso a SaaS aziendali
- applicazioni interne aziendali

---

### Caratteristiche

- controllo completo dei dati aziendali
- accesso limitato alle risorse condivise
- autenticazione tramite provider esterno

---

## 4. CIAM (Customer Identity and Access Management)

Usato per applicazioni rivolte ai clienti.

### Funzionalità principali

- registrazione self-service
- login personalizzato
- Single Sign-On (SSO)
- supporto social login (Google, Facebook, ecc.)

---

### Vantaggi

- esperienza utente semplificata
- onboarding rapido dei clienti
- sicurezza e scalabilità Microsoft Entra

---

## 5. Connessione diretta B2B

È una modalità avanzata di collaborazione tra organizzazioni.

### Caratteristiche principali

- non crea account guest nel tenant
- accesso diretto alle risorse condivise
- relazione trust tra due tenant Entra

---

### Utilizzo principale

- Microsoft Teams Connect (canali condivisi)
- chat e collaborazione in tempo reale
- condivisione file tra aziende

---

## 6. Differenze principali

| Modello | Scopo | Come accede l’utente |
|----------|------|----------------------|
| B2B Collaboration | partner/guest aziendali | account guest nel tenant |
| CIAM | clienti/consumatori | login diretto app |
| B2B Direct Connect | collaborazione tra aziende | accesso diretto senza guest |

---

## 7. Vantaggi di Microsoft Entra External ID

- collaborazione sicura con utenti esterni
- nessuna gestione manuale di account esterni
- autenticazione tramite provider fidati
- separazione tra identità interne ed esterne
- supporto a scenari moderni cloud e SaaS

---

## Conclusione

Microsoft Entra External ID consente di gestire in modo sicuro tutte le identità esterne, permettendo:

- collaborazione B2B
- gestione clienti (CIAM)
- accesso diretto tra organizzazioni

👉 È la base della collaborazione moderna senza compromettere sicurezza e controllo.
