# Ruoli di Microsoft Entra e controllo degli accessi in base al ruolo (RBAC)

I ruoli di Microsoft Entra controllano le autorizzazioni necessarie per gestire le risorse di Microsoft Entra, come utenti, gruppi e applicazioni. Ad esempio, un ruolo può consentire la creazione di utenti o la gestione delle informazioni di fatturazione.

La gestione dell’accesso tramite ruoli è chiamata **Role-Based Access Control (RBAC)**. In Microsoft Entra questo modello è noto come **Microsoft Entra RBAC**.

---

## Ruoli predefiniti

Microsoft Entra ID include ruoli con un set fisso di autorizzazioni che non possono essere modificati.

### Esempi principali:
- **Amministratore globale (Enterprise Administrator)**
  - Accesso completo a tutte le funzionalità amministrative di Microsoft Entra
  - È il ruolo assegnato per impostazione predefinita al creatore del tenant

- **Amministratore utenti**
  - Gestisce utenti e gruppi
  - Può gestire ticket di supporto e monitorare lo stato del servizio

- **Amministratore fatturazione**
  - Gestisce acquisti e sottoscrizioni
  - Può aprire ticket di supporto e monitorare servizi

👉 I ruoli predefiniti sono **fissi** e non modificabili.

---

## Ruoli personalizzati

I ruoli personalizzati permettono maggiore flessibilità nella gestione degli accessi.

### Come funzionano:
1. Si crea una **definizione del ruolo** scegliendo un set di autorizzazioni
2. Si assegna il ruolo a utenti o gruppi tramite una **assegnazione di ruolo**

### Caratteristiche:
- Basati su autorizzazioni disponibili nei ruoli predefiniti
- Permettono di selezionare solo i permessi necessari
- Supportano l’assegnazione con **ambito (scope)**

### Ambiti di assegnazione:
- **A livello organizzazione** → accesso a tutte le risorse
- **A livello oggetto** → accesso limitato (es. una singola applicazione)

👉 Richiedono licenza Microsoft Entra ID **P1 o P2**.

---

## Principio del privilegio minimo

È una best practice fondamentale assegnare agli utenti **solo i permessi necessari**.

Esempio:
- Un utente che gestisce utenti → ruolo “Amministratore utenti”, non “Amministratore globale”

### Vantaggi:
- Riduce i danni in caso di compromissione
- Limita accessi non necessari
- Migliora la sicurezza generale

Questo è ancora più importante con l’uso di sistemi AI che possono accedere a grandi quantità di dati.

---

## Categorie di ruoli Microsoft Entra

### 1. Ruoli specifici di Microsoft Entra
- Gestiscono risorse interne a Microsoft Entra
- Esempi: utenti, gruppi, applicazioni

### 2. Ruoli specifici del servizio
- Gestiscono singoli servizi Microsoft 365
- Esempi:
  - Amministratore Exchange
  - Amministratore Intune
  - Amministratore SharePoint
  - Amministratore Teams

### 3. Ruoli tra servizi
- Si applicano a più servizi
- Esempi:
  - Amministratore sicurezza
  - Amministratore conformità (Microsoft Purview)

---

## Microsoft Entra RBAC vs Azure RBAC

### Microsoft Entra RBAC
- Gestisce risorse di identità:
  - utenti
  - gruppi
  - applicazioni

### Azure RBAC
- Gestisce risorse Azure:
  - macchine virtuali
  - storage
  - risorse infrastrutturali

👉 Differenza principale:
- Entra RBAC → identità
- Azure RBAC → infrastruttura cloud

---

## Sintesi

Microsoft Entra RBAC permette di:
- controllare accessi amministrativi
- assegnare permessi in modo granulare
- applicare il principio del privilegio minimo
- separare responsabilità tra servizi e identità
