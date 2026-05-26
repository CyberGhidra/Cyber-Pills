# Gestione delle autorizzazioni in Microsoft Entra

La gestione delle autorizzazioni è una funzionalità di governance delle identità che consente alle organizzazioni di gestire il ciclo di vita delle identità e degli accessi su larga scala.

Automatizza:
- flussi di lavoro per richieste di accesso
- assegnazioni di autorizzazioni
- verifiche di accesso
- scadenza degli accessi

---

## Sfide comuni nella gestione degli accessi

Le organizzazioni spesso affrontano problemi come:

- Gli utenti non sanno quale livello di accesso richiedere o a chi rivolgersi per l’approvazione
- Gli accessi vengono mantenuti più a lungo del necessario
- La gestione degli utenti esterni (guest) è complessa e spesso incoerente

---

## Funzionalità principali

### 1. Pacchetti di accesso (Access Packages)

La gestione delle autorizzazioni si basa sui **pacchetti di accesso**, che sono insiemi di risorse necessarie per lavorare su un progetto o ruolo.

Un pacchetto può includere:
- gruppi di sicurezza
- gruppi Microsoft 365
- applicazioni aziendali
- siti SharePoint Online

---

### 2. Delegare la gestione degli accessi

Anche utenti non amministratori possono:
- creare pacchetti di accesso
- definire criteri di richiesta

I criteri includono:
- chi può richiedere accesso
- chi approva l’accesso
- quando l’accesso scade

---

### 3. Gestione utenti esterni (B2B)

Quando un utente esterno richiede accesso:

- viene invitato automaticamente nella directory
- ottiene i permessi richiesti
- alla scadenza, l’account guest può essere rimosso automaticamente

---

### 4. Governance delle identità AI

La gestione delle autorizzazioni supporta anche:
- identità di agenti di intelligenza artificiale
- controllo degli accessi per agenti AI
- sponsor che verificano la necessità continuativa dell’accesso

---

## Condizioni per l’utilizzo (Terms of Use)

Le condizioni per l’utilizzo di Microsoft Entra permettono di mostrare informazioni legali o di conformità agli utenti prima dell’accesso alle risorse.

### Casi d’uso tipici:
- accesso a dati o applicazioni sensibili
- promemoria periodici di conformità
- condizioni basate su ruolo o attributi utente
- applicazione a tutti gli utenti dell’organizzazione

---

## Caratteristiche delle condizioni per l’utilizzo

- Presentate in formato PDF personalizzato dall’organizzazione
- Supportano dispositivi mobili
- Richiedono accettazione prima dell’accesso

---

## Integrazione con Accesso Condizionale

Le condizioni per l’utilizzo sono applicate tramite:
- criteri di **Accesso Condizionale**
- verifica obbligatoria prima dell’accesso alle applicazioni

Gli amministratori possono:
- tracciare chi ha accettato o rifiutato le condizioni
- applicare regole basate su utenti o scenari specifici

---

## Obiettivo

La gestione delle autorizzazioni garantisce:
- accesso controllato e automatizzato alle risorse
- riduzione degli accessi inutili o troppo lunghi
- gestione efficiente di utenti interni ed esterni
- conformità e sicurezza migliorate
