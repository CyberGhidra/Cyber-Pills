# Privileged Identity Management (PIM)

Privileged Identity Management (PIM) è un servizio di Microsoft Entra ID che consente di gestire, controllare e monitorare l’accesso a risorse critiche dell’organizzazione.

Supporta risorse in:
- Microsoft Entra ID
- Azure
- Microsoft 365
- Microsoft Intune
- altri servizi Microsoft

PIM riduce il rischio di accessi eccessivi o non necessari tramite l’attivazione dei ruoli basata su tempo e approvazione.

---

## Principi fondamentali di PIM

PIM è:

- **Just-in-time (JIT)** → accesso solo quando necessario
- **Limitato nel tempo** → accessi con durata definita
- **Basato su approvazione** → richiede approvazione per l’attivazione
- **Visibile** → notifiche su attivazioni e utilizzo dei ruoli
- **Controllabile** → cronologia completa degli accessi disponibile

---

## Perché usare PIM

PIM aiuta a:
- ridurre il rischio di account compromessi con privilegi elevati
- minimizzare il numero di utenti con accesso permanente
- limitare il tempo di esposizione ai privilegi
- monitorare le attività amministrative

---

## Cosa si può gestire con PIM

PIM può essere utilizzato per:

### 1. Ruoli di Microsoft Entra
- ruoli di directory (predefiniti e personalizzati)
- gestione identità e servizi Microsoft 365

### 2. Ruoli di Azure
- controllo accessi su:
  - sottoscrizioni
  - gruppi di risorse
  - risorse Azure

### 3. PIM per gruppi
- appartenenza just-in-time ai gruppi
- gestione proprietari e membri dei gruppi
- accesso a servizi come:
  - Azure SQL
  - Azure Key Vault
  - Intune
  - applicazioni enterprise

---

## Flusso di lavoro di PIM

### 1. Assegnazione
Gli amministratori assegnano ruoli a:
- utenti
- gruppi
- entità servizio
- identità gestite

Tipi di assegnazione:
- **Idonea (Eligible)** → richiede attivazione prima dell’uso
- **Attiva (Active)** → accesso immediato senza attivazione

Parametri:
- membri o proprietari
- ambito (scope)
- durata (temporanea o permanente)

---

### 2. Attivazione
Gli utenti con ruolo idoneo devono:
- attivare il ruolo
- specificare durata dell’attivazione
- indicare un motivo

---

### 3. Approvazione o rifiuto
- i revisori ricevono notifiche
- possono approvare o negare richieste
- solo dopo approvazione il ruolo viene attivato

---

### 4. Estensione e rinnovo
- estensione → per prolungare un ruolo in scadenza
- rinnovo → per riattivare un ruolo in scadenza o scaduto

---

## Monitoraggio e controllo

PIM include:
- audit log delle assegnazioni e attivazioni
- storico fino a 30 giorni
- tracciamento completo delle attività privilegiate

---

## Obiettivo di PIM

PIM aiuta a:
- applicare il principio del minimo privilegio
- ridurre i rischi di sicurezza
- garantire controllo e trasparenza sugli accessi privilegiati
