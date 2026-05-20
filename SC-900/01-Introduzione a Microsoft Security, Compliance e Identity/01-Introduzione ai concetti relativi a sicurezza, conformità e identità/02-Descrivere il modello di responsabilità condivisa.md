# Modello di responsabilità condivisa

Il modello di responsabilità condivisa definisce come vengono distribuite le responsabilità di sicurezza tra cliente e provider cloud. Serve a capire chi protegge cosa nei diversi modelli di servizio (on-premise, IaaS, PaaS, SaaS e AI).

---

## 1. Modello tradizionale (on-premise)

In un ambiente locale:

- L’organizzazione gestisce **tutto lo stack tecnologico**
  - Edificio e sicurezza fisica
  - Hardware e server
  - Rete
  - Sistema operativo
  - Applicazioni
  - Dati

📌 Vantaggio: controllo completo  
📌 Svantaggio: carico di gestione e sicurezza completamente interno

---

## 2. Modello cloud e suddivisione responsabilità

Con il cloud, parte della sicurezza passa al provider.

### IaaS (Infrastructure as a Service)
- Provider:
  - Data center
  - Hardware
  - Rete fisica
- Cliente:
  - Sistema operativo
  - Applicazioni
  - Configurazione
  - Dati

---

### PaaS (Platform as a Service)
- Provider:
  - Infrastruttura
  - Sistema operativo
  - Runtime
- Cliente:
  - Applicazione
  - Dati
  - Configurazioni e accessi

---

### SaaS (Software as a Service)
- Provider:
  - Intero stack applicativo
- Cliente:
  - Utenti e identità
  - Dati
  - Configurazioni del servizio (tenant)
<img width="754" height="481" alt="image" src="https://github.com/user-attachments/assets/82289b90-56dd-40c3-bf52-5725ab8eb30b" />

---

## 3. Responsabilità sempre del cliente

Indipendentemente dal modello cloud:

### 📊 Dati
- Classificazione e protezione
- Crittografia
- Accesso e conservazione

si è responsabili dei dati, inclusa la classificazione della riservatezza, la scelta di come proteggerli, prendere decisioni di crittografia e garantire la conformità ai requisiti di governance dei dati. Il provider di servizi cloud protegge l'infrastruttura che archivia i dati, ma si decide quali dati raccogliere, chi può accedervi, per quanto tempo conservarli e come proteggerli.


### 🔐 Identità e accessi
- Account e permessi
- MFA
- Principio del minimo privilegio

la gestione di account utente, metodi di autenticazione e autorizzazioni di accesso è sempre responsabilità dell'utente. Un'identità compromessa può concedere a un utente malintenzionato l'accesso alle risorse cloud anche quando tutto il resto è bloccato. L'implementazione di procedure di identità complesse, ad esempio la richiesta di autenticazione a più fattori (MFA) e l'applicazione dell'accesso con privilegi minimi, è uno degli obblighi di sicurezza più importanti.

### 💻 Endpoint
- Sicurezza di PC, smartphone, tablet
- Patch e aggiornamenti
- Protezione da malware

I dispositivi che si connettono ai tuoi servizi cloud, ovvero portatili, tablet, telefoni e desktop, sono responsabilità tua da proteggere. Un utente malintenzionato che compromette un endpoint può usarlo per accedere ai servizi cloud come utente legittimo del dispositivo. La protezione degli endpoint tramite la gestione dei dispositivi, gli aggiornamenti della sicurezza e il rilevamento delle minacce fanno parte della responsabilità condivisa.

### ⚙️ Configurazioni
- Impostazioni del tenant cloud
- Policy di sicurezza
- Controllo accessi alle risorse

le impostazioni e le configurazioni di sicurezza all'interno del tenant cloud, ovvero la configurazione di servizi, autorizzazioni delle risorse e criteri di rete, sono responsabilità dell'utente. Un account di archiviazione configurato in modo errato o un criterio di accesso eccessivamente permissivo può esporre i dati anche quando la piattaforma sottostante è protetta correttamente.

---

## 4. Vantaggi del cloud

Il cloud non sposta solo responsabilità, ma migliora anche la sicurezza:

- Investimenti del provider in sicurezza fisica e infrastrutturale
- Monitoraggio e threat intelligence su larga scala
- Patch e aggiornamenti gestiti dal provider
- Riduzione del carico operativo interno

📌 Il cliente può concentrarsi su:
- dati
- identità
- configurazioni

---

## 5. Modello di responsabilità nell’AI

Nei sistemi di intelligenza artificiale il modello si estende su tre livelli:

### 🧠 Piattaforma AI (provider)
- Infrastruttura e hosting
- Modelli AI
- Sicurezza della piattaforma (filtri, controlli base)

### ⚙️ Applicazione AI (cliente)
- Configurazione dell’app
- Connessione a dati e servizi
- Plugin e integrazioni

### 👤 Uso dell’AI (cliente)
- Input degli utenti
- Policy di utilizzo
- Monitoraggio output

---

## 6. Responsabilità del provider AI

- Sicurezza infrastrutturale
- Gestione modelli e piattaforma
- Controlli di sicurezza integrati
- Protezione ambiente di esecuzione

A seconda che la funzionalità di intelligenza artificiale venga distribuita come servizio SaaS (ad esempio, Microsoft 365 Copilot) o un servizio PaaS (ad esempio, Azure AI Foundry), il provider gestisce in genere:

- Protezione dell'infrastruttura fisica e dell'ambiente di hosting del modello di intelligenza artificiale
- Fornire controlli di sicurezza a livello di piattaforma, ad esempio il filtro del contenuto e le misure di sicurezza predefinite, a seconda del livello di servizio
- Gestione dell'infrastruttura di calcolo sottostante e della sicurezza operativa del servizio di intelligenza artificiale

---

## 7. Responsabilità del cliente AI

- Protezione dei dati usati dall’AI
- Gestione identità e accessi
- Configurazione sicura del sistema
- Controllo rischi (es. prompt injection)
- Formazione utenti e policy di utilizzo

---

## 8. Punto chiave

Il cloud e l’AI riducono la gestione dell’infrastruttura, ma:

> La responsabilità su dati, identità, endpoint e configurazioni resta sempre al cliente.

📌 Capire “chi fa cosa” è fondamentale per una strategia di sicurezza efficace.
