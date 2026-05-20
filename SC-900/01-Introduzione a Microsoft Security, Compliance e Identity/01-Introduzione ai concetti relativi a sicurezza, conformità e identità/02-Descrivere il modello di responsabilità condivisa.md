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

---

## 3. Responsabilità sempre del cliente

Indipendentemente dal modello cloud:

### 📊 Dati
- Classificazione e protezione
- Crittografia
- Accesso e conservazione

### 🔐 Identità e accessi
- Account e permessi
- MFA
- Principio del minimo privilegio

### 💻 Endpoint
- Sicurezza di PC, smartphone, tablet
- Patch e aggiornamenti
- Protezione da malware

### ⚙️ Configurazioni
- Impostazioni del tenant cloud
- Policy di sicurezza
- Controllo accessi alle risorse

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
