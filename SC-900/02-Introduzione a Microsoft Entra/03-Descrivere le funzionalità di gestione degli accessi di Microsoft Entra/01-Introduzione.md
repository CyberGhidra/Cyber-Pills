# Accesso condizionale, Accesso globale sicuro e RBAC in Microsoft Entra

## Introduzione

In un modello di sicurezza moderno, il perimetro non è più la rete aziendale, ma le **identità, i dispositivi e le applicazioni**.

Per questo motivo :contentReference[oaicite:0]{index=0} utilizza controlli avanzati per decidere **chi può accedere a cosa, quando e da dove**.

I tre concetti fondamentali sono:

- Accesso condizionale
- Accesso globale sicuro (Security Service Edge)
- Ruoli e controllo degli accessi (RBAC)

---

## Accesso condizionale in Microsoft Entra

L’**accesso condizionale** è il motore Zero Trust che applica regole basate su segnali di sicurezza.

### Come funziona

Quando un utente tenta di accedere a una risorsa, il sistema valuta:

- Identità dell’utente
- Stato del dispositivo
- Posizione geografica
- Livello di rischio
- Applicazione richiesta

In base a questi segnali, viene applicata una decisione:

- Consenti accesso
- Blocca accesso
- Richiedi ulteriori verifiche (es. MFA)

---

### Esempio pratico

Un utente può:

- Accedere normalmente dalla rete aziendale
- Essere obbligato a usare MFA se si connette da rete pubblica
- Essere bloccato se il dispositivo è non conforme

---

### Punti di forza

- Controllo granulare degli accessi
- Basato su condizioni dinamiche
- Integrato con autenticazione a più fattori
- Protegge anche servizi cloud e AI

---

## Accesso globale sicuro (Secure Global Access)

L’**Accesso globale sicuro** è una soluzione Security Service Edge che unisce:

- Controlli di identità
- Controlli di rete
- Sicurezza dell’accesso Internet e SaaS

### Obiettivo

Proteggere l’accesso a:

- Internet
- Applicazioni SaaS
- Risorse aziendali private

---

### Componenti principali

- :contentReference[oaicite:1]{index=1}  
  Protegge l’accesso alle risorse Internet e SaaS

- :contentReference[oaicite:2]{index=2}  
  Protegge l’accesso alle risorse private senza VPN tradizionale

---

### Vantaggi

- Eliminazione delle VPN tradizionali
- Controllo centralizzato del traffico
- Sicurezza basata sull’identità
- Esperienza utente più semplice

---

## Ruoli Microsoft Entra e controllo degli accessi (RBAC)

Il **Role-Based Access Control (RBAC)** è il modello che assegna autorizzazioni in base ai ruoli.

### Concetto principale

Invece di assegnare permessi direttamente agli utenti, si assegnano:

- Ruoli
- Gruppi
- Funzioni amministrative

---

### Tipi di ruoli

- Ruoli amministrativi (es. Global Administrator)
- Ruoli limitati (es. Helpdesk Administrator)
- Ruoli personalizzati

---

### Principio del minimo privilegio

RBAC segue il principio Zero Trust:

> Ogni utente deve avere solo i permessi necessari per svolgere il proprio lavoro.

---

### Vantaggi

- Riduzione dei rischi di sicurezza
- Maggiore controllo sugli accessi
- Gestione semplificata delle autorizzazioni
- Audit più efficace

---

## Integrazione tra i componenti

Questi tre elementi lavorano insieme:

- **Accesso condizionale** → decide *se* l’accesso è consentito
- **Accesso globale sicuro** → protegge *come* avviene la connessione
- **RBAC** → definisce *cosa* l’utente può fare

---

## Conclusione

L’unione di questi strumenti in :contentReference[oaicite:3]{index=3} permette di costruire un modello di sicurezza Zero Trust completo, basato su identità, contesto e privilegi minimi.
