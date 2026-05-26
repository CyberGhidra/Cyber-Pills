# Microsoft Entra ID Protection

Microsoft Entra Verified ID  è una funzionalità che consente alle organizzazioni di **rilevare, analizzare e correggere i rischi legati alle identità utente e ai workload**.

L’obiettivo principale è ridurre il rischio di compromissione degli account attraverso il monitoraggio continuo dei segnali di sicurezza e l’automazione delle risposte.

---

## 1. Rilevamento dei rischi

ID Protection analizza miliardi di segnali giornalieri provenienti da Microsoft e dai suoi servizi per identificare comportamenti sospetti.

Esempi di rilevamenti:
- Accessi da IP anonimi (proxy/Tor)
- Attacchi di password spraying
- Credential stuffing o credenziali compromesse
- Attività di accesso impossibili (impossible travel)
- Comportamenti anomali rispetto allo storico utente

I rischi vengono classificati in due categorie principali:

### Rischio di accesso
Probabilità che una specifica richiesta di login sia non autorizzata.
Esempi:
- Accesso da posizione geografica insolita
- IP sospetto o anonimo
- Pattern di login anomali

### Rischio utente
Probabilità che l’account sia stato compromesso.
Esempi:
- Credenziali esposte o leaked
- Attività sospetta persistente
- Segnali aggregati da più eventi

---

## 2. Analisi dei rischi

I rischi rilevati vengono aggregati in tre viste principali:

- **Rilevamenti di rischio** → singoli eventi sospetti
- **Accessi a rischio** → login con segnali di rischio associati
- **Utenti a rischio** → utenti con uno o più segnali complessivi di compromissione

Queste informazioni aiutano gli amministratori a capire **chi è a rischio e perché**.

---

## 3. Correzione dei rischi

La correzione può essere:

### Automatica (consigliata)
Integrata con :contentReference[oaicite:2]{index=2}:
- Richiede MFA
- Richiede password reset sicuro
- Richiede autenticazione forte (es. phishing-resistant MFA)

Se l’utente soddisfa i requisiti, il rischio viene automaticamente risolto.

### Manuale
Se l’automazione non è attiva, l’amministratore può:
- Confermare compromissione
- Confermare utente sicuro
- Ignorare il rischio

Le azioni possono essere eseguite da portale, API o strumenti SIEM.

---

## 4. Integrazione e automazione

I dati di ID Protection possono essere integrati in altri sistemi tramite:

- :contentReference[oaicite:3]{index=3}
- SIEM (es. Microsoft Sentinel o altri sistemi)
- :contentReference[oaicite:4]{index=4}
- Log Analytics / Event Hub / Storage Account

Questo permette correlazione avanzata e analisi centralizzata dei rischi.

---

## 5. Sintesi

Microsoft Entra ID Protection:
- Rileva rischi basati su segnali globali
- Classifica rischi utente e di accesso
- Permette analisi centralizzata degli eventi
- Automatizza la mitigazione tramite Conditional Access
- Integra i dati in sistemi SIEM per indagini avanzate

👉 È una componente chiave della strategia Zero Trust per la protezione delle identità.
