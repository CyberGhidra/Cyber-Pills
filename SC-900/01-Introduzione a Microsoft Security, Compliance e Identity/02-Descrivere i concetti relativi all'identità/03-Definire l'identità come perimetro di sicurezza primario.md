# L’identità come perimetro di sicurezza primario

## Introduzione

Il modo di lavorare moderno è cambiato radicalmente: utenti, dispositivi e applicazioni accedono alle risorse aziendali da qualsiasi luogo e attraverso qualsiasi rete.

- Uffici
- Smart working
- Reti pubbliche
- Dispositivi personali
- Cloud e sistemi locali

Questo ha reso obsoleto il concetto tradizionale di sicurezza basato solo sul **perimetro di rete**. Oggi, il nuovo centro della sicurezza è l’**identità**.

---

## 1. Il perimetro di sicurezza tradizionale

### Cos’era il modello tradizionale

In passato la sicurezza si basava su un confine fisico/logico della rete aziendale:

- Firewall
- VPN
- Gateway di rete

### Principio di base
> Tutto ciò che è dentro la rete è considerato affidabile, tutto ciò che è fuori è non affidabile.

---

### Limiti del modello tradizionale

Questo approccio funzionava quando:
- Tutti lavoravano in ufficio
- Le applicazioni erano on-premises
- I dispositivi erano aziendali e controllati

Oggi però non è più sufficiente a causa di:

- **Cloud e SaaS** → applicazioni fuori dalla rete aziendale
- **Lavoro remoto e ibrido** → accesso da qualsiasi rete
- **BYOD (Bring Your Own Device)** → dispositivi personali non gestiti
- **Accesso di partner e clienti** → utenti esterni alla rete aziendale
- **IoT** → dispositivi connessi difficili da controllare

---

### Problema principale

Il perimetro di rete non è più affidabile perché:
- Il traffico spesso non passa più dalla rete aziendale
- Un attaccante con credenziali rubate può bypassare le difese perimetrali

---

## 2. L’identità come nuovo perimetro di sicurezza

### Concetto fondamentale

Quando la rete non è più un confine affidabile, il nuovo punto centrale diventa:

> **l’identità**

---

### Cos’è un’identità

Un’identità è un insieme di attributi che definiscono un soggetto digitale:

#### Identità utente:
- Nome utente
- Ruolo aziendale
- Permessi
- Metodi di autenticazione

#### Identità dispositivo:
- ID hardware
- Stato di conformità
- Registrazione nel sistema di gestione

---

### Nuovo modello di sicurezza

Invece di chiedere:
> “È dentro la rete aziendale?”

Si chiede:
> “Chi o cosa sta facendo questa richiesta e possiamo fidarci?”

---

### Perché l’identità è il nuovo perimetro

- Ogni accesso parte da un’identità
- Tutti i controlli di sicurezza si basano su identità
- La fiducia non dipende più dalla posizione di rete

---

## 3. Tipi di identità

In un ambiente moderno esistono diversi tipi di identità:

### 1. Identità umane
- Dipendenti
- Partner
- Clienti
- Terzisti

---

### 2. Identità dei dispositivi
- PC e laptop
- Smartphone e tablet
- Sensori IoT

Possono essere:
- registrati
- conformi
- attendibili o limitati

---

### 3. Identità dei carichi di lavoro
- Applicazioni
- Servizi cloud
- Container
- Processi automatizzati

Non richiedono intervento umano diretto.

---

### 4. Identità degli agenti (AI)
- Agenti di intelligenza artificiale
- Sistemi autonomi o assistivi

Richiedono gestione avanzata perché:
- agiscono autonomamente
- accedono a risorse e API
- introducono nuovi rischi di sicurezza

---

### Importanza

Tutte le identità devono essere:
- autenticate
- autorizzate

Un solo account compromesso può causare gravi danni, sia umano che applicativo o dispositivo.

---

## 4. Quattro pilastri dell’infrastruttura di identità

Una gestione moderna dell’identità si basa su quattro pilastri fondamentali:

---

### 1. Amministrazione (Lifecycle Management)

Gestione completa delle identità:

- Creazione (provisioning)
- Aggiornamento (cambio ruolo)
- Rimozione (deprovisioning)

Include anche politiche e governance.

---

### 2. Autenticazione

Verifica dell’identità:

- conferma che l’utente è chi dichiara di essere
- basata su credenziali e fattori di sicurezza

---

### 3. Autorizzazione

Definisce cosa può fare un’identità:

- basata su ruoli
- permessi
- contesto

Obiettivo:
> garantire accesso minimo necessario

---

### 4. Controllo (Auditing)

Monitoraggio delle attività:

- log di accesso
- tracciamento azioni
- analisi comportamenti sospetti
- supporto alle indagini forensi

Serve per:
- compliance
- sicurezza
- analisi incidenti

---

## 5. Sintesi finale

- Il modello di sicurezza basato sulla rete non è più sufficiente
- Il nuovo perimetro di sicurezza è l’**identità**
- Le identità includono utenti, dispositivi, applicazioni e agenti AI
- Tutte le identità devono essere gestite lungo tutto il loro ciclo di vita
- La sicurezza moderna si basa su quattro pilastri:
  - Amministrazione
  - Autenticazione
  - Autorizzazione
  - Controllo

---

## Conclusione

In un mondo cloud-first e distribuito, la sicurezza non dipende più da dove ci si trova, ma da **chi o cosa sta accedendo alle risorse**. L’identità diventa quindi il nuovo confine di sicurezza fondamentale su cui si basa ogni decisione di accesso.
