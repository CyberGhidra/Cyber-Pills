# 🏛️ Governance, Rischio e Conformità (GRC) 

> **GRC** è il framework integrato di procedure con cui le organizzazioni gestiscono regole interne, minacce e obblighi normativi in modo strutturato e coordinato.

### Perché serve un approccio GRC?

- Stabilire **responsabilità chiare** su chi decide e chi protegge le risorse
- **Identificare le minacce** prima che diventino incidenti
- **Dimostrare conformità** a leggi, normative e standard di settore
- **Creare fiducia** con clienti, partner e autorità

---

## 🔷 Le 3 componenti del GRC

```
┌─────────────────────────────────────────────┐
│                                             │
│   GOVERNANCE ──► RISK ──► COMPLIANCE        │
│   (Regole)      (Rischi)   (Normative)      │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 1. 📋 Governance

> Il sistema di **regole, procedure e processi** con cui un'organizzazione dirige e controlla le proprie attività.

In ambito sicurezza, la governance include:

- Definizione di policy per **classificazione, gestione e conservazione dei dati**
- Standard per la **gestione delle identità e degli accessi** (IAM)
- Processi di approvazione per **accessi privilegiati o amministrativi**
- Assegnazione di **ownership e responsabilità** sui controlli di sicurezza
- Definizione della **strategia di sicurezza complessiva**

> 💡 La governance crea la struttura su cui si appoggiano gestione dei rischi e conformità.

---

## 2. ⚠️ Risk Management (Gestione dei Rischi)

> Processo di **identificazione, valutazione e risposta** alle minacce che possono impattare gli obiettivi organizzativi.

L'obiettivo non è eliminare tutti i rischi, ma **comprenderli abbastanza da prendere decisioni informate**.

### Fonti di rischio

| Tipo | Esempi |
|---|---|
| **Esterni** | Attacchi informatici, disastri naturali, cambiamenti normativi, fornitori terzi |
| **Interni** | Esposizione accidentale dati, minacce insider, frodi, lacune nei processi |

### Il processo di gestione dei rischi

```
1. IDENTIFICARE
   └─ Interviste, vulnerability assessment, audit, monitoraggio continuo

2. VALUTARE
   └─ Probabilità × Impatto = Punteggio di rischio → priorità

3. RISPONDERE
   ├─ Accettare   → impatto basso, rischio tollerabile
   ├─ Mitigare    → implementare controlli di sicurezza
   ├─ Trasferire  → assicurazione, contratti con terzi
   └─ Evitare     → modificare il processo che genera il rischio

4. MONITORARE
   └─ Tracciare i rischi nel tempo, misurare l'efficacia dei controlli
```

---

## 3. ✅ Compliance (Conformità)

> Rispettare **leggi, normative, standard e policy** applicabili all'organizzazione in base a settore, area geografica e tipologia di dati.

### ⚠️ Conformità ≠ Sicurezza

| | Conformità | Sicurezza |
|---|---|---|
| Definisce | Standard **minimi** richiesti dalla legge | Protezione **completa** di dati e sistemi |
| Portata | Prescritta da enti esterni | Più ampia, include tecnologie e processi interni |
| Rischio | Si può essere conformi ma avere vulnerabilità significative | Mira a coprire anche ciò che la legge non prevede |

> Un'organizzazione può essere tecnicamente conforme pur avendo gravi falle di sicurezza.

### Framework e normative principali

| Standard | Ambito |
|---|---|
| **HIPAA** | Protezione dei dati sanitari (USA) |
| **ISO 27001** | Sistemi di gestione della sicurezza delle informazioni (internazionale) |
| **SOC 2** | Provider di servizi che archiviano/elaborano dati dei clienti |

---

## 📌 Concetti chiave sulla conformità dei dati

### 🌍 Residenza dei Dati
Le normative stabiliscono **dove** i dati possono essere fisicamente archiviati e come possono essere trasferiti internazionalmente. Le regole variano per giurisdizione.

> ➡️ Chi usa servizi cloud deve verificare dove il provider archivia fisicamente i dati e se soddisfa i requisiti applicabili.

---

### 🏳️ Sovranità dei Dati
I dati sono soggetti alle **leggi del paese in cui vengono raccolti, archiviati o elaborati**. Se un dato attraversa più giurisdizioni, è potenzialmente soggetto alle leggi di ognuna.

```
Raccolta (Paese A) ──► Archiviazione (Paese B) ──► Elaborazione (Paese C)
      └── legge A ──────── legge B ──────────────── legge C
```

> ➡️ Le organizzazioni globali devono mappare le giurisdizioni applicabili e capire come interagiscono.

---

### 🔏 Privacy dei Dati
La privacy riguarda la corretta gestione dei **dati personali** — qualsiasi informazione riconducibile a un individuo (nome, email, telefono, ma anche cronologia posizione, dati di navigazione, ecc.).

Le leggi sulla privacy richiedono di:

1. Essere **trasparenti** sui dati raccolti e sull'uso che ne viene fatto
2. Ottenere il **consenso** appropriato prima della raccolta
3. Garantire agli utenti **diritti** sui propri dati (accesso, correzione, cancellazione)
4. **Proteggere** i dati personali da accessi non autorizzati

> ➡️ Le organizzazioni devono rispettare le leggi sulla privacy ovunque operino e ovunque si trovino i loro clienti.

---

## In Sintesi

```
GRC
├── GOVERNANCE   → regole, policy, responsabilità, strategia di sicurezza
├── RISK         → identificare → valutare → rispondere → monitorare
└── COMPLIANCE   → leggi e standard minimi (HIPAA, ISO 27001, SOC 2)
                    ├── Residenza dei dati  → dove sono archiviati
                    ├── Sovranità dei dati  → leggi della giurisdizione
                    └── Privacy dei dati    → gestione dati personali
```


