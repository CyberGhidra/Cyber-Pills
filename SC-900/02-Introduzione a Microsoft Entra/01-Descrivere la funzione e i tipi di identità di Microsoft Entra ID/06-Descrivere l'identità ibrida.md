# Identità ibrida in Microsoft Entra ID

## Introduzione

L’identità ibrida è un modello che permette a un’organizzazione di usare **una singola identità** per accedere sia a:

- applicazioni locali (on-premises)
- servizi cloud

👉 Anche se i sistemi sono distribuiti, l’utente usa sempre le stesse credenziali.

---

## 1. Concetto di identità ibrida

Un’identità ibrida nasce quando:

- Active Directory locale (AD DS)
- Microsoft Entra ID (cloud)

sono collegati tra loro.

👉 Risultato: un’unica identità per tutte le risorse aziendali.

---

## 2. Obiettivo principale

- semplificare l’accesso degli utenti
- evitare credenziali multiple
- garantire coerenza tra cloud e on-premises
- migliorare sicurezza e gestione centralizzata

---

## 3. Come funziona

L’identità ibrida si basa su due processi principali:

### Provisioning
- creazione dell’identità in Microsoft Entra ID
- parte da un utente già presente in Active Directory locale

👉 In pratica: AD locale → Entra ID

---

### Synchronization
- mantiene allineati utenti, gruppi e attributi
- garantisce coerenza tra ambiente locale e cloud

---

## 4. Strumenti per l’identità ibrida

### Microsoft Entra Cloud Sync (consigliato)

È lo strumento moderno per la sincronizzazione.

#### Caratteristiche:
- agente leggero installato on-premises o in IaaS
- configurazione gestita da cloud (Entra ID)
- alta disponibilità con più agenti
- supporto multi-foresta

👉 È la soluzione consigliata da Microsoft per nuove implementazioni.

---

### Microsoft Entra Connect Sync (legacy)

- soluzione precedente
- installata e gestita localmente
- ancora usata in ambienti esistenti
- sostituita gradualmente da Cloud Sync

---

## 5. Standard SCIM

Microsoft Entra Cloud Sync utilizza lo standard:

### SCIM (System for Cross-domain Identity Management)

Serve per:

- provisioning automatico utenti
- provisioning gruppi
- sincronizzazione tra sistemi di identità diversi

👉 È uno standard moderno per lo scambio di identità tra sistemi cloud e on-premises.

---

## 6. Architettura semplificata
Active Directory (on-premises)
↓
Cloud Sync Agent
↓
Microsoft Entra ID (cloud)
↓
App cloud + servizi SaaS


---

## 7. Vantaggi dell’identità ibrida

- accesso unico (Single Identity)
- meno gestione password
- sicurezza centralizzata
- integrazione cloud + on-premises
- esperienza utente uniforme

---

## 8. Sintesi finale

L’identità ibrida in Microsoft Entra:

- unisce Active Directory e Entra ID
- permette una sola identità per tutti i sistemi
- usa sincronizzazione e provisioning
- si basa su Cloud Sync (consigliato)
- utilizza lo standard SCIM
- supporta ambienti moderni cloud + legacy

---

## Conclusione

L’identità ibrida è la base della transizione al cloud: consente alle organizzazioni di modernizzare l’accesso senza abbandonare i sistemi locali, garantendo continuità, sicurezza e semplicità.****
