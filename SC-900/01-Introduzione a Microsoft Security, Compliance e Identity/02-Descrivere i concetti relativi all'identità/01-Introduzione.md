# Identità, Autenticazione e Autorizzazione in Microsoft Security

## Introduzione

L’identità rappresenta il modo in cui persone, dispositivi e applicazioni dimostrano chi sono quando accedono a risorse digitali. In uno scenario moderno in cui il lavoro è distribuito tra cloud e sistemi locali, l’identità è diventata il principale perimetro di sicurezza delle organizzazioni, sostituendo il tradizionale concetto di “rete protetta”.

---

## 1. Autenticazione e Autorizzazione

### Autenticazione
L’autenticazione è il processo con cui un sistema verifica l’identità di un utente, dispositivo o applicazione. In pratica risponde alla domanda:
> “Chi sei?”

Esempi:
- Password
- MFA (Multi-Factor Authentication)
- Certificati digitali
- Token di accesso

### Autorizzazione
L’autorizzazione determina cosa un’identità autenticata è autorizzata a fare. Risponde alla domanda:
> “Cosa puoi fare?”

Esempi:
- Accesso a file specifici
- Permessi su applicazioni
- Ruoli amministrativi

### Relazione tra i due
- L’autenticazione avviene prima
- L’autorizzazione si basa sull’identità verificata
- Insieme controllano in modo completo l’accesso alle risorse

---

## 2. L’identità come nuovo perimetro di sicurezza

Tradizionalmente la sicurezza si basava sul perimetro di rete (firewall e confini aziendali). Oggi questo modello non è più sufficiente perché:
- Gli utenti lavorano da remoto
- Le applicazioni sono distribuite nel cloud
- I dati risiedono su più piattaforme

Di conseguenza, l’identità è diventata il nuovo perimetro di sicurezza.

### Perché è importante
- Ogni accesso viene verificato in base all’identità
- La sicurezza segue l’utente ovunque si trovi
- Riduce la dipendenza dalla posizione fisica o dalla rete

---

## 3. Provider di identità (Identity Provider)

Un Identity Provider (IdP) è un sistema che gestisce e verifica le identità digitali.

### Funzioni principali:
- Autenticazione centralizzata
- Gestione credenziali e utenti
- Emissione di token di accesso
- Supporto Single Sign-On (SSO)

### Single Sign-On (SSO)
Consente agli utenti di accedere a più applicazioni con una sola autenticazione, migliorando:
- Sicurezza
- Esperienza utente
- Riduzione delle password da gestire

### Autenticazione moderna
Utilizza:
- Token invece di sessioni tradizionali
- Protocolli sicuri (es. OAuth, OpenID Connect)

---

## 4. Servizi directory: Active Directory e Microsoft Entra ID

### Active Directory (AD)
- Sistema tradizionale on-premises
- Gestisce utenti, gruppi e dispositivi in ambienti locali
- Basato su domini Windows

### Microsoft Entra ID (ex Azure Active Directory)
- Evoluzione cloud di Active Directory
- Gestione identità moderna per applicazioni cloud
- Supporta accesso remoto e applicazioni SaaS

### Differenze principali
| Active Directory | Microsoft Entra ID |
|------------------|-------------------|
| On-premises      | Cloud-based       |
| Dominio locale   | Accesso globale   |
| LDAP/Kerberos    | OAuth/OpenID      |

---

## 5. Federazione e Trust tra organizzazioni

La federazione permette a più organizzazioni di condividere l’autenticazione senza creare account separati.

### Come funziona:
- Le organizzazioni stabiliscono una relazione di trust
- Un’identità può essere riconosciuta tra domini diversi
- L’utente accede alle risorse senza nuove credenziali

### Vantaggi:
- Accesso semplificato tra aziende
- Collaborazione più efficiente
- Riduzione gestione account duplicati

---

## 6. Concetti chiave finali

Al termine del modulo è importante comprendere che:

- L’autenticazione verifica l’identità, l’autorizzazione definisce i permessi
- L’identità è il nuovo perimetro di sicurezza nel cloud moderno
- Gli Identity Provider centralizzano e modernizzano la gestione degli accessi
- Active Directory e Microsoft Entra ID rappresentano l’evoluzione dei servizi directory
- La federazione consente accesso sicuro tra organizzazioni diverse

---

## Sintesi finale

La sicurezza moderna si basa sull’identità come elemento centrale. In un mondo cloud-first e distribuito, controllare *chi sei* è più importante di *da dove accedi*. Microsoft Entra ID e i moderni sistemi di autenticazione permettono accessi sicuri, flessibili e scalabili, mentre la federazione estende questa sicurezza oltre i confini aziendali.
