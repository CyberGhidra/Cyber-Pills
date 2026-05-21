# Ruolo del Provider di Identità (IdP)

## Introduzione

L’autenticazione moderna si basa su un modello centralizzato in cui un servizio dedicato, chiamato **Identity Provider (IdP)**, gestisce:
- autenticazione
- autorizzazione
- gestione delle identità
- controllo degli accessi

Questo approccio sostituisce il modello tradizionale in cui ogni applicazione gestiva autonomamente utenti e credenziali.

---

## 1. Cos’è un Identity Provider

Un **provider di identità (IdP)** è un servizio centrale che:

- crea e gestisce identità digitali
- autentica utenti, dispositivi e applicazioni
- emette token di sicurezza
- supporta autorizzazione e controllo accessi
- centralizza la gestione dell’identità

👉 Le applicazioni non gestiscono più direttamente utenti e password, ma delegano tutto all’IdP.

---

## 2. Come funziona il processo di autenticazione

Il flusso tipico è:

1. L’utente tenta di accedere a un’applicazione
2. L’applicazione reindirizza l’utente al provider di identità
3. L’utente si autentica presso l’IdP (es. password, MFA)
4. L’IdP verifica le credenziali
5. L’IdP rilascia un **token di sicurezza**
6. L’utente presenta il token all’applicazione
7. L’applicazione verifica il token e concede l’accesso

👉 L’applicazione si fida del provider di identità, non delle credenziali dirette.

---

## 3. Perché centralizzare l’autenticazione

Senza un IdP centrale:

- ogni app gestisce utenti separati
- password duplicate o deboli
- politiche di sicurezza incoerenti
- gestione complessa del ciclo di vita degli utenti

---

### Problemi principali senza IdP

- **Credenziali frammentate**
  - più password per ogni servizio
  - alto rischio di riutilizzo e phishing

- **Sicurezza incoerente**
  - MFA e policy diverse tra applicazioni

- **Alto costo amministrativo**
  - gestione utenti ripetuta per ogni sistema
  - account dimenticati = rischio sicurezza

---

### Vantaggi della centralizzazione

Con un IdP centrale si ottiene:

- policy di sicurezza uniformi (MFA, password, accesso)
- monitoraggio centralizzato degli accessi
- gestione completa del ciclo di vita utente (onboarding/offboarding)
- maggiore sicurezza e controllo

---

## 4. Token di sicurezza e attestazioni

### Cos’è un token

Un **token di sicurezza** è un pacchetto di dati emesso dall’IdP dopo l’autenticazione.

L’applicazione lo usa come prova di identità.

---

### Attestazioni (claims)

I token contengono informazioni chiamate **attestazioni**, ad esempio:

- ID univoco dell’utente
- nome e email
- ruoli e gruppi
- timestamp di emissione e scadenza
- permessi associati

---

## 5. Tipi di token

### Token ID
- conferma che l’utente è autenticato
- contiene informazioni sull’identità
- serve per sapere “chi è l’utente”

👉 riguarda l’**autenticazione**

---

### Token di accesso
- consente l’accesso a risorse specifiche
- definisce cosa l’utente/app può fare
- usato per API e servizi protetti

👉 riguarda l’**autorizzazione**

---

### Caratteristiche comuni

- hanno durata limitata
- scadono dopo un tempo definito
- possono essere revocati indirettamente tramite IdP
- sono validati tramite trust tra applicazione e IdP

---

## 6. Protocolli di autenticazione

I provider di identità comunicano con le applicazioni tramite protocolli standard.

---

### OpenID Connect (OIDC)
- basato su OAuth 2.0
- usato per autenticazione moderna
- consente accesso a profili utente
- comune in app cloud e mobile

---

### OAuth 2.0
- usato per autorizzazione
- permette accesso a risorse tramite token
- non autentica direttamente l’utente

---

### SAML (Security Assertion Markup Language)
- usato in ambienti enterprise
- comune per sistemi legacy
- spesso utilizzato per federazione tra organizzazioni

---

## 7. Single Sign-On (SSO)

### Cos’è

Il **Single Sign-On** consente agli utenti di autenticarsi una sola volta e accedere a più applicazioni senza reinserire credenziali.

---

### Come funziona

- l’utente si autentica sull’IdP
- l’IdP rilascia token
- le applicazioni accettano il token senza nuovo login

---

### Vantaggi del SSO

#### 1. Esperienza utente migliore
- una sola login
- meno password da ricordare

#### 2. Maggiore sicurezza
- riduce phishing e riuso password
- meno richieste di credenziali

#### 3. Controllo centralizzato
- disabilitando un account si revoca tutto l’accesso
- onboarding/offboarding più rapido

---

## 8. Federazione

Quando più provider di identità si fidano tra loro, si parla di **federazione**.

👉 Permette agli utenti di accedere a sistemi tra organizzazioni diverse senza account separati.

---

## 9. Esempi di Identity Provider

Esempi di provider di identità moderni:

- Microsoft Entra ID
- Google Identity
- Amazon
- LinkedIn
- GitHub

---

## 10. Sintesi finale

- L’Identity Provider centralizza la gestione delle identità
- Le applicazioni delegano autenticazione e autorizzazione all’IdP
- L’IdP rilascia token di sicurezza con informazioni sull’utente
- I protocolli standard (OIDC, OAuth, SAML) garantiscono interoperabilità
- Il Single Sign-On migliora sicurezza ed esperienza utente
- La federazione estende l’accesso tra organizzazioni
