# Microsoft Entra Verified ID

Microsoft Security Copilot è un servizio di credenziali verificabili basato su standard aperti che consente di **emettere, presentare e verificare identità digitali in modo sicuro e rispettoso della privacy**.

Permette di abilitare scenari di identità decentralizzata in cui utenti e organizzazioni possono scambiarsi attestazioni verificabili senza condividere dati sensibili non necessari.

---

## Perché è importante

Nel mondo digitale moderno, molte interazioni richiedono la prova di attributi o qualifiche (es. identità, ruolo lavorativo, titoli di studio).

I problemi principali dei modelli tradizionali sono:
- Difficoltà nel verificare credenziali digitali in modo affidabile
- Mancanza di controllo dell’utente sui propri dati
- Rischi di condivisione eccessiva di informazioni personali

Verified ID risolve questi problemi usando **credenziali crittograficamente verificabili e privacy-preserving**.

---

## Come funziona

Il modello si basa su tre ruoli principali:

### 1. Emittente (Issuer)
È un’organizzazione che rilascia una credenziale verificata:
- Azienda
- Università
- Ente governativo
- Provider di identità

Firma digitalmente la credenziale.

---

### 2. Utente (Holder)
- Riceve la credenziale
- La memorizza in un wallet digitale (es. Microsoft Authenticator)
- Decide quando e con chi condividerla
- Presenta solo le informazioni necessarie

---

### 3. Verificatore (Verifier)
- Richiede la prova dell’identità o di un attributo
- Verifica la credenziale tramite firma crittografica
- Controlla che sia valida, non revocata e attendibile

---

## Registro e attendibilità

Il sistema si basa su un modello decentralizzato:
- Usa identificatori decentralizzati (DID)
- Il modello :contentReference[oaicite:2]{index=2} collega l’identità dell’emittente al suo dominio web
- L’attendibilità deriva dal controllo del dominio da parte dell’organizzazione

---

## Ciclo di vita delle credenziali

Le credenziali possono:
- Scadere dopo un periodo definito
- Essere revocate dall’emittente
- Essere aggiornate o sostituite

Questo garantisce che le informazioni siano sempre attuali e affidabili.

---

## Recupero account con Verified ID

Verified ID è usato anche nel **ripristino dell’account** in :contentReference[oaicite:3]{index=3}.

Scenario tipico:
- L’utente perde tutti i metodi di autenticazione
- Non può usare SSPR (self-service password reset)

Processo:
1. Verifica identità tramite provider terzo certificato
2. Controllo documento + selfie (con AI)
3. Emissione di una credenziale verificabile
4. Rilascio di un Temporary Access Pass (TAP)
5. Reset e registrazione nuovi metodi di accesso

👉 Solo i risultati della verifica vengono condivisi, non i dati personali completi.

---

## Intelligenza artificiale e sicurezza

Con l’aumento di:
- deepfake
- identità sintetiche
- frodi digitali

Verified ID aiuta a:
- confermare che l’utente è reale
- ridurre il rischio di impersonificazione
- aumentare la fiducia nelle transazioni digitali

---

## Sintesi

Microsoft Entra Verified ID:
- abilita identità decentralizzate
- usa credenziali verificabili crittografiche
- riduce la condivisione di dati sensibili
- migliora sicurezza e privacy
- supporta scenari di onboarding e recovery avanzati
