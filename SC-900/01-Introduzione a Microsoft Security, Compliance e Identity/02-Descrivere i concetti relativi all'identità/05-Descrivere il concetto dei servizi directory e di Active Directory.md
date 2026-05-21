# Servizi di directory e Active Directory

## Introduzione

Per gestire utenti, dispositivi e risorse in una rete, le organizzazioni hanno bisogno di un sistema centralizzato in grado di:

- archiviare informazioni sulle identità
- controllare gli accessi
- gestire autenticazione e autorizzazione

Questo sistema è chiamato **servizio di directory**.

---

## 1. Cos’è un servizio di directory

Un **servizio di directory** è un sistema software che:

- archivia informazioni sugli oggetti di rete
- rende i dati accessibili a utenti, applicazioni e amministratori
- supporta la gestione centralizzata delle identità

---

### Cosa contiene una directory

Una directory può includere:

- utenti
- dispositivi
- gruppi
- applicazioni
- criteri di sicurezza

---

### Funzioni principali

Un servizio di directory risponde a domande come:

- Chi è l’utente?
- A quali gruppi appartiene?
- A quali risorse può accedere?

👉 È la base dell’autenticazione e dell’autorizzazione nelle reti aziendali.

---

## 2. Active Directory (AD)

### Cos’è

**Active Directory (AD)** è un insieme di servizi directory sviluppati da Microsoft per la gestione delle reti aziendali locali (on-premises).

È stato introdotto con Windows 2000 Server.

---

## 3. Active Directory Domain Services (AD DS)

Il componente principale di Active Directory è:

> **Active Directory Domain Services (AD DS)**

---

### Funzioni principali di AD DS

AD DS:

- memorizza informazioni su utenti, gruppi e dispositivi
- autentica gli utenti durante il login
- gestisce i diritti di accesso alle risorse

---

### Controller di dominio

Un server che esegue AD DS è chiamato:

> **Domain Controller (DC)**

---

## 4. Cosa permette di fare AD DS

Active Directory Domain Services consente di:

- usare una sola identità per accedere a più sistemi locali
- gestire configurazioni tramite **Group Policy**
- autenticare utenti con protocolli come:
  - Kerberos
  - NTLM
- organizzare utenti e computer in **OU (Organizational Units)**

---

## 5. Limiti di Active Directory in ambienti moderni

AD DS è stato progettato per ambienti **on-premises**, quindi presenta limiti nel mondo moderno cloud-first.

---

### 1. Cloud e SaaS

- non supporta nativamente applicazioni cloud
- richiede sincronizzazione o federazione

---

### 2. Dispositivi mobili

- non gestisce nativamente dispositivi iOS e Android
- servono strumenti aggiuntivi

---

### 3. Protocolli moderni

- non supporta nativamente:
  - OAuth 2.0
  - OpenID Connect

---

### 4. Accesso remoto

- richiede VPN per accedere dall’esterno della rete aziendale
- introduce complessità e possibili rischi

---

## 6. Evoluzione: Microsoft Entra ID

### Cos’è

**Microsoft Entra ID** è il servizio di identità cloud di Microsoft.

È l’evoluzione moderna di Active Directory per ambienti cloud e Internet.

---

### Modello

È un servizio **IDaaS (Identity as a Service)**:

- Microsoft gestisce l’infrastruttura
- le aziende gestiscono utenti e policy

---

## 7. Vantaggi di Microsoft Entra ID

Microsoft Entra ID supera i limiti di AD DS:

- supporta autenticazione moderna (OAuth, OpenID Connect)
- funziona su cloud e dispositivi mobili
- non richiede VPN per accesso remoto
- integra applicazioni SaaS e cloud native

---

## 8. Identità ibride

Le organizzazioni possono combinare:

- Active Directory (on-premises)
- Microsoft Entra ID (cloud)

👉 Questo modello si chiama **identità ibrida**

---

### Vantaggi dell’identità ibrida

- sincronizzazione tra cloud e on-premises
- accesso unificato per utenti
- uso delle stesse credenziali ovunque

---

## 9. Sintesi finale

- Un servizio di directory gestisce identità e risorse in una rete
- Active Directory è la soluzione Microsoft per ambienti on-premises
- AD DS è il componente principale che gestisce utenti e autenticazione
- Active Directory è limitato nel mondo cloud moderno
- Microsoft Entra ID è l’evoluzione cloud-based
- Le identità ibride collegano on-premises e cloud in un unico sistema

---

## Conclusione

I servizi di directory sono fondamentali per la gestione delle identità. Active Directory ha dominato gli ambienti tradizionali, mentre Microsoft Entra ID rappresenta l’evoluzione moderna per il cloud, la mobilità e il lavoro remoto.
