# Descrivere i metodi di autenticazione

## Cos'è l'autenticazione

L'autenticazione è il processo di verifica dell'identità di una persona prima di concedere l'accesso a una risorsa, applicazione, servizio, dispositivo o rete.

Microsoft Entra ID supporta numerosi metodi di autenticazione che bilanciano:
- sicurezza
- facilità d’uso
- semplicità di distribuzione

---

# Password

Le password rappresentano il metodo di autenticazione più comune, ma anche uno dei più vulnerabili.

## Problemi delle password
- Password semplici → facili da violare
- Password complesse → difficili da ricordare
- Password dimenticate → perdita di produttività
- Possibilità di phishing e furto credenziali

Per questo motivo Microsoft consiglia di:
- integrare le password con MFA
- oppure sostituirle con metodi passwordless

---

# Autenticazione basata su telefono

## SMS Authentication

L’utente:
1. inserisce il numero di telefono registrato
2. riceve un codice via SMS
3. inserisce il codice nella schermata di accesso

Può essere usata:
- come autenticazione primaria
- come secondo fattore MFA
- per SSPR (Self-Service Password Reset)

### Limiti
Microsoft sconsiglia SMS come metodo principale di sicurezza avanzata a causa dei rischi di phishing e SIM swapping.

---

## Verifica tramite chiamata vocale

Il sistema:
1. chiama automaticamente il numero registrato
2. chiede all’utente di premere `#`

Usata solo come:
- autenticazione secondaria
- MFA
- SSPR

Non supportata come autenticazione primaria.

---

# OATH (Open Authentication)

OATH è uno standard aperto per la generazione di password monouso temporanee (TOTP).

## Token software OATH
Applicazioni come:
- Microsoft Authenticator
- Google Authenticator

Generano codici OTP temporanei.

## Token hardware OATH
Dispositivi fisici simili a portachiavi che:
- mostrano codici temporanei
- aggiornano il codice ogni 30/60 secondi

Usati come:
- MFA
- SSPR

---

# Altri metodi di autenticazione

## Temporary Access Pass (TAP)

Codice temporaneo generato da un amministratore.

Usato per:
- onboarding nuovi utenti
- recupero accesso
- registrazione metodi passwordless

---

## QR Code Authentication

Pensato per:
- frontline workers
- dispositivi condivisi

L’utente:
1. scansiona un QR code
2. inserisce un PIN

---

## OTP via Email

Codice inviato tramite email.

Usato per:
- SSPR
- accesso utenti guest

---

## Platform Credentials per macOS

Autenticazione passwordless su macOS usando:
- Touch ID
- PIN/password dispositivo

Supporta:
- SSO
- credenziali resistenti al phishing

---

## Authenticator Lite

Funzionalità integrata in app come Outlook Mobile.

Permette:
- notifiche push MFA
- codici OTP

senza installare Microsoft Authenticator.

---

## Metodi di autenticazione esterni

Microsoft Entra può integrarsi con provider MFA esterni come:
- Duo Security
- RSA SecurID

---

# Autenticazione Passwordless

L’obiettivo moderno è eliminare completamente le password.

I metodi passwordless combinano:
- qualcosa che possiedi
- qualcosa che conosci o sei

---

# Windows Hello for Business

Sostituisce la password con:
- PIN
- biometria (impronta, volto)

Utilizza:
- chiavi crittografiche
- TPM del dispositivo

## Vantaggi
- protezione dal furto credenziali
- autenticazione locale sicura
- resistente al phishing

---

# Passkey (FIDO2)

Le passkey usano crittografia a chiave pubblica.

## Funzionamento
Durante la registrazione:
- viene generata una coppia di chiavi
- la chiave privata resta sul dispositivo
- la chiave pubblica viene registrata in Microsoft Entra ID

---

## Tipi di passkey

### Passkey associate al dispositivo
Esempi:
- chiavi FIDO2 USB/NFC/Bluetooth
- Microsoft Authenticator

### Passkey sincronizzate
Sincronizzate tramite:
- iCloud Keychain
- Google Password Manager

---

## Vantaggi delle passkey
- resistenti al phishing
- impossibili da riutilizzare su siti fake
- maggiore sicurezza rispetto alle password

---

# Microsoft Authenticator

Applicazione disponibile per:
- Android
- iOS

Supporta:
- MFA
- passwordless
- passkey

---

## Modalità supportate

### Passwordless Sign-In
L’utente:
1. vede un numero a schermo
2. conferma nell’app
3. usa biometria o PIN

---

### Push Notification MFA
L’app invia una notifica push con:
- approvazione richiesta
- number matching

---

### Codici OATH
Generazione OTP temporanei.

---

# Autenticazione basata su certificati (CBA)

Usa certificati X.509 per autenticazione diretta.

## Caratteristiche
- passwordless
- elimina dipendenza da AD FS
- supporta MFA

---

# Autenticazione resistente al phishing

Microsoft raccomanda metodi resistenti al phishing contro attacchi AI-based.

## Caratteristiche
Usano:
- crittografia a chiave pubblica
- binding al dominio originale

Le credenziali:
- non possono essere riutilizzate
- non funzionano su siti fake

---

## Metodi resistenti al phishing

- Windows Hello for Business
- Credenziali macOS
- Passkey FIDO2
- Microsoft Authenticator passkey
- Certificate Based Authentication (CBA)

---

# Autenticazione primaria e secondaria

| Metodo | Primaria | Secondaria |
|---|---|---|
| Windows Hello for Business | Sì | MFA |
| Credenziali macOS | Sì | MFA |
| Passkey FIDO2 | Sì | MFA |
| Passkey in Microsoft Authenticator | Sì | MFA |
| Passkey sincronizzate | Sì | MFA |
| Certificate Based Authentication | Sì | MFA |
| Microsoft Authenticator Passwordless | Sì | No |
| Microsoft Authenticator Push | Sì | MFA + SSPR |
| Authenticator Lite | No | MFA |
| Token hardware OATH | No | MFA + SSPR |
| Token software OATH | No | MFA + SSPR |
| Metodi MFA esterni | No | MFA |
| Temporary Access Pass | Sì | MFA |
| SMS Sign-In | Sì | MFA + SSPR |
| Chiamata vocale | No | MFA + SSPR |
| QR Code | Sì | No |
| OTP Email | No | SSPR |
| Password | Sì | No |

---

# Conclusione

Microsoft Entra ID offre un ecosistema completo di metodi di autenticazione moderni.

L’approccio consigliato da Microsoft è:
1. ridurre la dipendenza dalle password
2. adottare MFA
3. implementare autenticazione passwordless resistente al phishing

I metodi più sicuri oggi sono:
- Passkey FIDO2
- Windows Hello for Business
- Certificate Based Authentication
- Microsoft Authenticator passwordless
