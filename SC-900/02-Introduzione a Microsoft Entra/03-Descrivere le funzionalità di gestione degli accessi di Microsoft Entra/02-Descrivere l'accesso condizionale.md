# Accesso condizionale in Microsoft Entra

L’accesso condizionale è una funzionalità di Microsoft Entra che aggiunge un livello di sicurezza prima di consentire agli utenti autenticati di accedere a dati, applicazioni o altre risorse. È il motore di criteri Zero Trust di Microsoft, che valuta diversi segnali prima di prendere decisioni di accesso.

In pratica, i criteri di accesso condizionale funzionano come istruzioni **if-then**:
> Se un utente vuole accedere a una risorsa, allora deve soddisfare determinate condizioni (ad esempio MFA).

Esempio:
- Se un utente accede a Microsoft 365 → deve completare l’autenticazione a più fattori.

> Nota: l’accesso condizionale viene applicato dopo il primo fattore di autenticazione e non protegge contro attacchi come DoS, ma può usare segnali di tali eventi per le decisioni di accesso.

---

## Componenti dei criteri di accesso condizionale

Un criterio è composto da due elementi principali:
- **Assegnazioni**
- **Controlli di accesso**

---

## Assegnazioni

Le assegnazioni definiscono **chi, cosa, dove e quando** si applica un criterio. Se sono presenti più assegnazioni, devono essere tutte soddisfatte (logica AND).

### Utenti
- Tutti gli utenti
- Utenti e gruppi specifici
- Ruoli della directory
- Guest e identità esterne
- Identità di workload e agenti AI

### Risorse di destinazione
- App cloud (Microsoft 365 e app registrate)
- Azioni utente (es. registrazione sicurezza, join dispositivi)
- Profili di traffico di accesso sicuro globale
- Contesto di autenticazione

### Rete / Posizione
- IP attendibili
- Posizioni denominate
- Reti conformi
- Posizioni geografiche

### Condizioni
- Rischio utente e rischio accesso (Microsoft Entra ID Protection)
- Rischio insider (Microsoft Purview)
- Piattaforma del dispositivo
- Tipo di app client (browser, mobile, desktop)
- Filtri per dispositivi (attributi specifici)

---

## Controlli di accesso

Definiscono cosa succede quando le assegnazioni sono soddisfatte.

### Blocca accesso
- Impedisce completamente l’accesso

### Concedi accesso
Può richiedere:
- Autenticazione a più fattori (MFA)
- Forza dell’autenticazione
- Dispositivo conforme
- Dispositivo ibrido Entra joined
- App client approvata o app protection policy
- Cambio password
- Termini di utilizzo

### Controlli di sessione
- Limitazioni nelle app cloud
- Controllo download/copia/stampa (Defender for Cloud Apps)
- Frequenza di accesso
- Restrizioni basate su applicazione

---

## Livelli di forza dell’autenticazione

Microsoft Entra permette di definire livelli di sicurezza per l’autenticazione:

### MFA standard
- Password + secondo fattore (SMS, app, token OATH, ecc.)

### MFA senza password
- Microsoft Authenticator
- Windows Hello for Business
- FIDO2 (passkey / chiavi di sicurezza)
- Credenziali piattaforma

### MFA resistente al phishing
- Windows Hello for Business
- FIDO2
- Autenticazione basata su certificati

Gli amministratori possono anche creare livelli personalizzati per requisiti specifici.

---

## Protezione dei servizi di intelligenza artificiale

L’accesso condizionale può proteggere servizi AI come:
- Microsoft 365 Copilot
- Microsoft Security Copilot

Esempi di protezione:
- Richiedere MFA resistente al phishing
- Richiedere dispositivo conforme
- Bloccare accesso con rischio insider elevato

---

## Licenze

- Accesso condizionale richiede **Microsoft Entra ID P1 o P2**
- Il livello gratuito può usare le **impostazioni predefinite di sicurezza**
