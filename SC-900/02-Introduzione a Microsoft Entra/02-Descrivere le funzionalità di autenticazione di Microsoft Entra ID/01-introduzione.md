# Autenticazione in Microsoft Entra ID

## Introduzione

L’autenticazione è il processo che verifica l’identità di un utente, dispositivo o applicazione prima di concedere l’accesso alle risorse.

Tradizionalmente l’autenticazione si basa sulle password, ma le password da sole non sono più sufficienti per garantire sicurezza moderna.

Con la crescita di:

- phishing
- credential theft
- attacchi automatizzati
- AI-driven cyber attacks

le organizzazioni devono adottare metodi di autenticazione più sicuri, moderni e resistenti agli attacchi.

Microsoft Entra ID fornisce diverse funzionalità per migliorare la sicurezza dell’autenticazione.

---

# 1. Metodi di autenticazione in Microsoft Entra ID

Microsoft Entra ID supporta diversi metodi di autenticazione.

---

## Password

Metodo tradizionale basato su:

- username
- password

### Problemi principali
- password deboli
- riutilizzo password
- phishing
- brute force
- credential stuffing

👉 Per questo motivo Microsoft spinge verso autenticazione moderna e passwordless.

---

## Metodi basati sul telefono

Consentono autenticazione tramite:

- SMS
- chiamata vocale
- notifiche push

Spesso usati come secondo fattore MFA.

---

## Microsoft Authenticator

Applicazione mobile Microsoft che consente:

- approvazione login tramite notifica push
- generazione codici OTP
- autenticazione passwordless

👉 Uno dei metodi più comuni in ambienti Microsoft Entra.

---

## Windows Hello for Business

Metodo passwordless basato su:

- PIN locale
- biometria
  - impronta digitale
  - riconoscimento facciale

### Vantaggi
- credenziali legate al dispositivo
- resistente al phishing
- esperienza utente rapida

---

## Security Keys (FIDO2)

Chiavi hardware fisiche usate per autenticazione sicura.

### Caratteristiche
- standard FIDO2
- resistente al phishing
- autenticazione senza password

📌 Esempi:
- YubiKey
- Titan Security Key

---

## Certificati e Smart Card

Usati spesso in ambienti enterprise o governativi.

Consentono:
- autenticazione forte
- elevato livello di sicurezza

---

# 2. Passwordless Authentication

Microsoft Entra ID supporta autenticazione senza password.

## Obiettivo
Eliminare completamente l’uso delle password.

---

## Vantaggi

- riduzione phishing
- riduzione password reuse
- migliore esperienza utente
- minore gestione IT

---

## Metodi passwordless supportati

- Windows Hello for Business
- Microsoft Authenticator
- FIDO2 Security Keys

---

# 3. Multi-Factor Authentication (MFA)

## Cos’è

MFA richiede almeno due fattori di autenticazione.

---

## Tipi di fattori

### Qualcosa che conosci
- password
- PIN

### Qualcosa che possiedi
- telefono
- token hardware
- smart card

### Qualcosa che sei
- biometria

---

## Esempio pratico

- password + approvazione app Authenticator
- PIN + impronta digitale

---

## Benefici MFA

- riduce rischio account compromise
- protegge da phishing
- aumenta sicurezza identità

👉 Anche se la password viene rubata, l’attaccante non può accedere senza il secondo fattore.

---

# 4. Self-Service Password Reset (SSPR)

## Cos’è

Permette agli utenti di reimpostare autonomamente la password senza intervento IT.

---

## Funzionalità

- reset password self-service
- recupero account
- verifica identità tramite metodi configurati

---

## Benefici

- riduzione ticket helpdesk
- recupero account rapido
- maggiore produttività utenti

---

## Verifiche possibili

- email
- SMS
- Microsoft Authenticator
- domande di sicurezza

---

# 5. Password Protection

Microsoft Entra ID include protezione avanzata delle password.

---

## Funzionalità principali

### Password banned list
Blocca password:
- troppo comuni
- facilmente indovinabili
- compromesse

---

### Password smart lockout
Blocca tentativi di accesso sospetti.

---

### Protezione da password deboli
Impedisce:
- varianti semplici
- parole comuni
- pattern prevedibili

---

# 6. Sicurezza moderna e Zero Trust

Microsoft Entra ID segue il modello Zero Trust:

> “Never trust, always verify”

Ogni accesso viene valutato considerando:

- identità
- rischio
- dispositivo
- posizione
- comportamento

---

# 7. Sintesi finale

Microsoft Entra ID offre:

- autenticazione moderna
- MFA
- passwordless authentication
- gestione password sicura
- reimpostazione self-service
- protezione da phishing e attacchi AI

---

# Conclusione

Le password tradizionali non sono più sufficienti per proteggere ambienti moderni cloud e ibridi.

Microsoft Entra ID introduce metodi di autenticazione avanzati e resistenti al phishing, migliorando:

- sicurezza
- esperienza utente
- gestione centralizzata delle identità
- resilienza contro gli attacchi moderni
