# Microsoft Entra Agent ID

## Introduzione

Microsoft Entra Agent ID è un framework di identità e sicurezza progettato per gli **agenti di intelligenza artificiale**.

Serve a fornire identità sicure agli agenti AI che:

- automatizzano attività
- prendono decisioni dinamiche
- interagiscono con sistemi e API
- operano in modo assistito o autonomo

👉 È un’estensione di Microsoft Entra ID per il mondo AI.

---

## 1. Perché gli agenti AI hanno bisogno di identità

Gli agenti AI sono diversi dalle applicazioni tradizionali perché:

- agiscono in modo dinamico e adattivo
- possono eseguire azioni autonome
- interagiscono con sistemi esterni
- prendono decisioni basate sul contesto

### Principali rischi di sicurezza

#### Superficie di attacco più ampia
- interazione con sistemi esterni
- vulnerabilità a attacchi (es. prompt injection)

#### Rischi di autorizzazione
- permessi spesso troppo ampi
- accesso a dati non necessari

#### Proliferazione degli agenti
- creazione incontrollata di agenti
- mancanza di monitoraggio e ciclo di vita
- agenti “dimenticati” in produzione

👉 Serve quindi una gestione identitaria dedicata e controllata.

---

## 2. Come funziona Microsoft Entra Agent ID

Il sistema si basa su Microsoft Entra ID e introduce due concetti principali:

---

### Blueprint di identità agente (Agent Identity Blueprint)

È un **modello riutilizzabile** che definisce:

- tipo di agente
- autorizzazioni base
- criteri di sicurezza
- configurazioni comuni

📌 Esempio:
- “Agente assistente vendite”
- “Agente supporto clienti”

👉 Tutti gli agenti dello stesso tipo seguono le stesse regole.

---

### Identità dell’agente (Agent Identity)

È una **singola istanza reale dell’agente**.

Caratteristiche:

- identificatore univoco
- nome
- sponsor (utente o gruppo responsabile)
- appartiene a un blueprint

📌 Importante:
- non ha credenziali proprie
- usa il blueprint per ottenere token di accesso

---

### Vantaggio del modello

- gestione centralizzata
- controllo su larga scala
- coerenza delle policy
- governance semplificata

---

## 3. Modalità di autenticazione degli agenti

### 1. Modalità assistita (on behalf of)

- l’agente agisce per conto di un utente
- usa autorizzazioni delegate
- opera con il contesto dell’utente

📌 Esempio:
un assistente AI che accede alla posta dell’utente per aiutarlo

---

### 2. Modalità autonoma (unattended)

- l’agente agisce in modo indipendente
- usa i propri ruoli e permessi
- non richiede presenza dell’utente

📌 Esempio:
bot che esegue processi di analisi dati automaticamente

---

## 4. Governance e sicurezza

Microsoft Entra Agent ID si integra con le funzionalità di sicurezza Entra:

---

### Accesso condizionale
- controlli basati su rischio
- regole specifiche per agenti AI

---

### Identity Protection
- rilevamento del rischio in tempo reale
- risposta automatica agli eventi sospetti

---

### Identity Governance
- gestione ciclo di vita degli agenti
- assegnazione e revoca accessi
- audit e compliance

---

### Registro degli agenti
- inventario centralizzato degli agenti
- visibilità completa nel tenant
- tracciamento delle attività

---

## 5. Principio dei privilegi minimi

Microsoft Entra Agent ID applica il principio di **least privilege**:

- gli agenti ricevono solo i permessi necessari
- limitazione dei ruoli amministrativi
- riduzione del rischio di abuso o escalation

👉 Anche gli amministratori non possono assegnare privilegi eccessivi agli agenti.

---

## 6. Sintesi finale

Microsoft Entra Agent ID:

- assegna identità sicure agli agenti AI
- separa agenti da utenti e workload tradizionali
- introduce blueprint + istanze di agenti
- supporta modalità assistita e autonoma
- integra governance e sicurezza Entra
- applica il principio di privilegi minimi

---

## Conclusione

Microsoft Entra Agent ID estende il modello Zero Trust agli agenti di intelligenza artificiale, garantendo che anche entità non umane siano:

- identificate
- autorizzate
- monitorate
- governate in modo sicuro
