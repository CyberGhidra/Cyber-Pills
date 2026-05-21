# 🔐 Zero Trust

> **"Trust nessuno, verifica tutto."**

## Cos'è Zero Trust?

Zero Trust non è un singolo prodotto, ma una **strategia di sicurezza**. Nasce per superare il modello tradizionale *castle-and-moat* (perimetro sicuro = rete sicura), che non regge più nell'era del lavoro remoto, dei dispositivi personali e dei dati distribuiti tra cloud e data center.

**Il problema del vecchio modello:**
- Tutto ciò che era dentro la rete veniva considerato affidabile per default
- Gli attaccanti, una volta dentro (tramite phishing, furto di credenziali, ecc.), si muovevano liberamente

**La soluzione Zero Trust:**  
Nessuna rete, dispositivo, utente o applicazione è considerata attendibile per default — nemmeno se già autenticata o connessa alla rete interna.

---

## I 3 Principi Guida

### 1. Verificare in modo esplicito
Autenticare e autorizzare **sempre**, basandosi su tutti i segnali disponibili:

| Segnale | Esempi |
|---|---|
| Identità | Chi è l'utente? |
| Posizione | Da dove si connette? |
| Dispositivo | È conforme e registrato? |
| Applicazione | A quale risorsa vuole accedere? |
| Dati | Che classificazione hanno? |
| Anomalie | Orari o posizioni insolite? |

Più segnali vengono valutati, più alta è la certezza che l'accesso sia legittimo.

---

### 2. Usare l'accesso con privilegi minimi (Least Privilege)
Concedere solo l'accesso **strettamente necessario**, nel **momento in cui serve**:

- **JIT – Just-In-Time**: l'accesso viene concesso al momento del bisogno e revocato subito dopo
- **JEA – Just-Enough-Access**: solo le autorizzazioni specifiche per il task, non generali "per sicurezza"
- Le policy adattive basate sul rischio possono limitare automaticamente l'accesso in presenza di attività anomale

---

### 3. Presupporre la violazione (Assume Breach)
Progettare la sicurezza **come se un attaccante riuscisse sempre ad entrare**:

- Segmentare l'accesso → un account compromesso non raggiunge tutto il resto
- Crittografare i dati end-to-end → i dati intercettati risultano illeggibili
- Monitorare continuamente → rilevare la violazione in fretta e limitare i danni

---

## I 7 Pilastri di Zero Trust

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   Identità ── Dispositivi ── Applicazioni               │
│       │                          │                      │
│       └──────► CONTROLLO ◄───────┘                      │
│                    │                                    │
│   Dati ── Infrastruttura ── Reti                        │
│                    │                                    │
│   ┌────────────────▼────────────────────┐               │
│   │  Visibilità, Automazione,           │               │
│   │  Orchestrazione (SIEM / SOAR)       │               │
│   └─────────────────────────────────────┘               │
└─────────────────────────────────────────────────────────┘
```

### 🪪 1. Identità
- Gli utenti (e servizi/dispositivi) devono autenticarsi con **autenticazione avanzata** (MFA, ecc.)
- L'accesso è regolato da principi di least privilege
- Quasi tutti gli attacchi riusciti compromettono un'identità: è il piano di controllo principale

### 💻 2. Dispositivi
- Ogni dispositivo è un potenziale punto d'ingresso per gli attaccanti
- I dispositivi vengono monitorati per **stato di integrità e conformità**
- Dispositivi non conformi o compromessi vengono bloccati o hanno accesso limitato

### 📦 3. Applicazioni
- Serve visibilità su **tutte** le app usate, incluso lo *shadow IT* (app usate senza approvazione formale)
- Gestione delle autorizzazioni richieste dalle app e concesse dagli utenti

### 🗄️ 4. Dati
- I dati devono essere **classificati, etichettati e crittografati** in base alla loro riservatezza
- Devono portare la propria protezione anche fuori dagli ambienti controllati dall'organizzazione

### 🏗️ 5. Infrastruttura
- Valutazione continua di versioni, configurazioni e comportamenti anomali
- Accesso **JIT** alle interfacce amministrative per ridurre l'esposizione
- Telemetria per rilevare e rispondere agli attacchi

### 🌐 6. Reti
- **Micro-segmentazione**: confini non solo al perimetro, ma anche all'interno
- Crittografia end-to-end del traffico
- Monitoraggio e protezione dalle minacce in tempo reale

### 👁️ 7. Visibilità, Automazione e Orchestrazione *(pilastro centrale)*
- Aggrega e correla i segnali di tutti gli altri 6 pilastri
- Senza questo livello → troppi alert isolati → minacce perse + affaticamento degli analisti
- **SIEM**: raccoglie e correla i segnali per rilevare le minacce
- **SOAR**: usa i rilevamenti per attivare playbook di risposta automatizzati

---

## Zero Trust in Pratica – Esempio Reale

**Scenario:** un utente si connette da una posizione non familiare, su un dispositivo personale non gestito, tentando di accedere a un'app finanziaria sensibile.

| Pilastro | Cosa succede |
|---|---|
| **Identità** | Rileva la posizione insolita come segnale di rischio |
| **Dispositivi** | Segnala che il dispositivo non è registrato/conforme |
| **Applicazioni** | Applica la policy: accesso solo da dispositivi conformi |
| **Reti** | Registra e monitora la connessione |
| **Dati** | I dati finanziari restano crittografati e con audit di accesso |
| **Infrastruttura** | Registra il tentativo nella telemetria |
| **Visibilità** | Correla tutti i segnali → rilevamento ad alta confidenza → blocco automatico + alert al team |

Il risultato: accesso bloccato, team di sicurezza avvisato, **zero intervento manuale necessario**.

---

## In Sintesi

| Aspetto | Sicurezza Tradizionale | Zero Trust |
|---|---|---|
| Presupposto di base | Dentro = sicuro | Nessuno è sicuro per default |
| Perimetro | Confine di rete | Identità + contesto |
| Accesso | Ampio una volta dentro | Minimo, JIT, JEA |
| Risposta alle minacce | Reattiva | Continua e automatizzata |
| Adatto al cloud/remoto | ❌ No | ✅ Sì |

---


