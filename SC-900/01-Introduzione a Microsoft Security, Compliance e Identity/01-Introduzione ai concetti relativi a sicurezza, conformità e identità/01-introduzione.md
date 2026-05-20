# Sicurezza e Conformità – Concetti fondamentali

Questo modulo introduce i concetti base di sicurezza, conformità e gestione delle identità in ambienti cloud, locali e servizi AI.

---

## 1. Modello di responsabilità condivisa

Nel cloud la sicurezza è suddivisa tra provider e cliente.

- **Provider cloud**
  - Protegge infrastruttura fisica
  - Gestisce data center, rete e hardware
- **Cliente**
  - Protegge dati, identità e accessi
  - Gestisce configurazioni e permessi

📌 Più il servizio è “alto” (SaaS), più responsabilità passa al provider, ma il cliente mantiene sempre una parte di responsabilità.

---

## 2. Difesa in profondità (Defense in Depth)

Strategia di sicurezza basata su più livelli di protezione.

Strati principali:
- Sicurezza fisica
- Sicurezza di rete
- Identità e accessi
- Sicurezza applicativa
- Protezione dei dati

📌 Se un livello viene compromesso, gli altri continuano a proteggere il sistema.

---

## 3. Modello CIA

Fondamento della sicurezza informatica:

- **Confidenzialità** → accesso solo a utenti autorizzati
- **Integrità** → dati non modificati in modo non autorizzato
- **Disponibilità** → sistemi e dati sempre accessibili quando necessario

---

## 4. Zero Trust

Modello di sicurezza moderno basato sul principio:

> “Non fidarti mai, verifica sempre”

Caratteristiche:
- Verifica continua di utenti e dispositivi
- Controllo accessi basato su identità e contesto
- Nessuna fiducia implicita, nemmeno nella rete interna

Componenti principali:
- Identità
- Dispositivi
- Rete
- Applicazioni
- Dati
- Infrastruttura
- Visibilità e analisi

---

## 5. Crittografia e Hashing

Metodi per proteggere i dati.

### Crittografia
- Rende i dati leggibili solo con chiave
- Usata per dati in transito, inattivi e in uso

### Hashing
- Trasforma i dati in una stringa irreversibile
- Usato per password e verifica integrità

---

## 6. Governance, Rischi e Conformità (GRC)

Approccio strutturato per gestione sicurezza e normative.

- **Governance** → politiche e regole interne
- **Rischi** → identificazione e mitigazione minacce
- **Conformità** → rispetto di leggi e standard

Concetti chiave:
- **Residenza dei dati** → dove sono archiviati i dati
- **Sovranità dei dati** → leggi del paese sui dati
- **Privacy dei dati** → protezione delle informazioni personali

---

## Sintesi finale

La sicurezza moderna si basa su:
- responsabilità condivisa
- difesa multilivello
- modello Zero Trust
- protezione dei dati tramite crittografia e hashing
- rispetto di governance e normative (GRC)
