# Descrivere l'autenticazione a più fattori (MFA)

## Cos'è l'autenticazione a più fattori

L'autenticazione a più fattori (MFA - Multi-Factor Authentication) è un processo di sicurezza che richiede agli utenti di fornire due o più forme di verifica durante l'accesso a un sistema, applicazione o servizio.

L’obiettivo è aumentare la sicurezza riducendo il rischio che un account venga compromesso tramite:
- password rubate
- phishing
- credential stuffing
- attacchi automatizzati

---

# Perché MFA è importante

Usare solo una password rappresenta un rischio significativo:
- le password possono essere deboli
- possono essere riutilizzate
- possono essere rubate tramite phishing o malware

Con MFA, anche se un utente malintenzionato conosce la password, non può accedere senza il secondo fattore di verifica.

Secondo Microsoft:
> oltre il 99,9% degli attacchi comuni alle identità viene bloccato usando MFA e bloccando l'autenticazione legacy.

---

# Fattori di autenticazione

Microsoft Entra MFA utilizza almeno due dei seguenti fattori.

## 1. Qualcosa che conosci
Esempi:
- password
- PIN

---

## 2. Qualcosa che possiedi
Esempi:
- smartphone
- token hardware
- chiave FIDO2

---

## 3. Qualcosa che sei
Esempi:
- impronta digitale
- riconoscimento facciale
- biometria

---

# Come funziona Microsoft Entra MFA

Durante il login:
1. l’utente inserisce le credenziali principali
2. Microsoft Entra ID verifica l’identità
3. viene richiesta una seconda verifica MFA
4. l’utente approva usando un metodo registrato

Microsoft Entra gestisce automaticamente il processo MFA senza modificare applicazioni esistenti.

Gli utenti possono:
- aggiungere metodi MFA
- modificarli
- gestirli tramite il portale **MyAccount**

---

# Metodi di verifica supportati

Microsoft Entra MFA supporta:

- Microsoft Authenticator
- Authenticator Lite (Outlook)
- Windows Hello for Business
- Passkey FIDO2
- Passkey in Microsoft Authenticator
- Certificate Based Authentication (CBA)
- Provider MFA esterni
- Temporary Access Pass (TAP)
- Token hardware OATH
- Token software OATH
- SMS
- Chiamata vocale

---

# Microsoft Authenticator

È il metodo consigliato da Microsoft.

## Funzionamento
L’utente riceve:
- una notifica push
- oppure una richiesta con number matching

L’utente:
1. apre l’app
2. verifica il numero mostrato
3. approva usando biometria o PIN

---

# Number Matching

Per prevenire gli attacchi MFA Fatigue:
- Microsoft mostra un numero nella schermata login
- l’utente deve inserirlo nell’app Authenticator

Questo evita approvazioni accidentali o fraudolente.

---

# Security Defaults

Le **Security Defaults** sono configurazioni di sicurezza di base abilitate automaticamente nei nuovi tenant Microsoft Entra.

## Obiettivi
Garantire una protezione minima senza configurazioni avanzate.

---

## Funzionalità incluse

### Registrazione MFA obbligatoria
Tutti gli utenti devono registrare MFA.

### MFA obbligatoria per amministratori
Gli account privilegiati devono sempre usare MFA.

### MFA automatica quando necessario
Microsoft può richiedere MFA durante accessi sospetti.

### Blocco autenticazione legacy
Vengono bloccati protocolli vecchi e meno sicuri.

### Protezione accessi privilegiati
Ad esempio:
- Azure Portal
- amministrazione tenant

---

# Accesso Condizionale (Conditional Access)

Per esigenze più avanzate, Microsoft Entra offre l’Accesso Condizionale.

L’Accesso Condizionale permette di applicare MFA solo in determinate condizioni.

---

# Esempi di criteri

## In base alla posizione
- accesso normale in ufficio
- MFA obbligatoria da reti esterne

---

## In base al dispositivo
- accesso consentito solo da dispositivi conformi
- MFA richiesta su dispositivi personali

---

## In base al rischio
- MFA automatica se il login è sospetto
- blocco accessi ad alto rischio

---

## In base all'applicazione
Applicare protezioni più forti per:
- applicazioni sensibili
- dati aziendali critici

---

# Requisiti di licenza

## Security Defaults
Disponibili anche con:
- Microsoft Entra ID Free

---

## Conditional Access
Richiede:
- Microsoft Entra ID P1
oppure
- Microsoft Entra ID P2

---

# Vantaggi dell'autenticazione a più fattori

## Sicurezza maggiore
Riduce drasticamente il rischio di compromissione account.

## Protezione contro phishing
Gli attacchi basati solo sulla password diventano inefficaci.

## Controllo granulare
Con Conditional Access è possibile adattare le policy ai rischi reali.

## Compatibilità cloud
Funziona con:
- Microsoft 365
- Azure
- applicazioni SaaS
- applicazioni aziendali

---

# Conclusione

Microsoft Entra MFA rappresenta uno dei controlli di sicurezza più importanti in un ambiente moderno Zero Trust.

L’approccio consigliato è:
1. abilitare MFA per tutti gli utenti
2. usare metodi resistenti al phishing
3. ridurre gradualmente l’uso delle password
4. implementare criteri di Accesso Condizionale per protezioni avanzate
