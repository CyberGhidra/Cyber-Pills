# Accesso globale sicuro in Microsoft Entra

Global Secure Access è la soluzione **Security Service Edge (SSE)** di Microsoft basata sui principi Zero Trust:
- verifica esplicita
- privilegio minimo
- presunzione di violazione

Combina diversi servizi Microsoft per unificare i controlli di accesso:
- Microsoft Entra Internet Access
- Microsoft Entra Internet Access per i servizi Microsoft
- Microsoft Entra Private Access
- Microsoft Defender for Cloud Apps

L’obiettivo è proteggere l’accesso a qualsiasi applicazione o risorsa, da qualsiasi luogo, dispositivo o identità.

---

## Problemi di sicurezza che risolve

- Riduzione del rischio di movimenti laterali tramite VPN compromesse
- Creazione di un perimetro di sicurezza per il traffico Internet
- Miglioramento della sicurezza nelle sedi remote e filiali

---

## Architettura e funzionamento

Global Secure Access utilizza un **client installato sui dispositivi degli utenti**, che:
- controlla il traffico di rete
- instrada il traffico verso il servizio SSE
- applica criteri basati su identità, dispositivo, posizione e rischio

Si integra con:
- Accesso condizionale Microsoft Entra
- valutazione del rischio in tempo reale

---

# Microsoft Entra Internet Access

È un **Secure Web Gateway (SWG)** basato sull’identità per il traffico Internet e SaaS.

### Funzionalità principali:
- Filtro web per categorie e domini
- Accesso condizionale universale anche per siti non federati
- Continuous Access Evaluation (CAE)
  - aggiorna continuamente lo stato dell’accesso
  - può bloccare o limitare l’accesso in tempo reale

👉 Richiede licenza:
- Microsoft Entra Suite oppure
- licenza standalone Internet Access

---

# Internet Access per servizi Microsoft

Ottimizza la connessione ai servizi Microsoft 365:
- Exchange Online
- SharePoint Online
- Microsoft Teams

### Funzionalità principali:
- Verifica di rete conforme (Conditional Access)
- Protezione contro furto di token
- Restrizioni tenant universali (blocca accessi non autorizzati)
- Ripristino IP di origine nei log
- Log dettagliati del traffico Microsoft 365

👉 Incluso con Microsoft Entra ID P1/P2

---

# Microsoft Entra Private Access

Fornisce accesso sicuro alle risorse private sostituendo le VPN tradizionali con un modello **ZTNA (Zero Trust Network Access)**.

L’accesso è basato su applicazioni, non su reti intere.

## Modalità di configurazione

### Accesso rapido
- Basato su FQDN, IP e porte
- Raggruppa risorse in un’unica applicazione
- Usa criteri di accesso condizionale

### Accesso per app
- Configurazione granulare per singola applicazione
- Criteri separati per ogni app o gruppo di risorse

---

## Componenti principali

- Applicazioni aziendali (contenitori logici)
- Connettori di rete (broker del traffico)
- Politiche di accesso condizionale

---

# Dashboard Accesso globale sicuro

Fornisce visibilità centralizzata su:
- traffico di rete degli utenti
- accessi tra tenant
- filtri web applicati
- stato dei dispositivi

Permette di:
- rilevare attività sospette
- analizzare traffico
- ottimizzare i criteri di sicurezza

---

# Protezione dei carichi di lavoro AI

Global Secure Access applica i principi Zero Trust anche ai servizi AI:

- controllo del traffico AI verso risorse cloud e on-premises
- integrazione con Accesso Condizionale
- limitazione accessi in base a:
  - rischio utente
  - stato del dispositivo
  - posizione

---

# Sintesi

Global Secure Access unifica:
- sicurezza di rete
- identità
- accesso alle applicazioni

Sostituisce VPN e controlli tradizionali con un modello Zero Trust moderno e centralizzato.
