# 🔒 Crittografia e Hashing 

> La crittografia rende i dati **illeggibili** agli utenti non autorizzati. Per leggere dati crittografati serve una **chiave privata** — senza di essa, i dati intercettati sono inutili.

---

## Tipi di Crittografia

### 🔑 Crittografia Simmetrica

> Una sola chiave per crittografare **e** decrittografare.

```
Mittente ──[chiave condivisa]──► Dati cifrati ──[stessa chiave]──► Destinatario
```

| ✅ Pro | ❌ Contro |
|---|---|
| Computazionalmente veloce | Problema di distribuzione della chiave |
| Adatta a grandi quantità di dati (file, database) | Entrambe le parti devono avere la stessa chiave privata |

---

### 🔐 Crittografia Asimmetrica

> Una **coppia** di chiavi matematicamente correlate: pubblica + privata.

```
Mittente cifra con ──► chiave PUBBLICA del destinatario
Destinatario decifra con ──► sua chiave PRIVATA
```

- La chiave pubblica può essere condivisa liberamente
- La chiave privata non è derivabile da quella pubblica
- **Risolve** il problema di distribuzione della crittografia simmetrica

**Usata in:** HTTPS, email cifrata, VPN, certificati digitali

---

### ✍️ Firme Digitali (bonus: autenticità + integrità)

La crittografia asimmetrica permette anche di **firmare** digitalmente i dati:

```
Mittente firma con ──► chiave PRIVATA
Chiunque verifica con ──► chiave PUBBLICA del mittente
```

Una firma digitale garantisce che:
1. I dati provengono dal mittente previsto **(autenticità)**
2. I dati non sono stati modificati dopo la firma **(integrità)**

**Usata in:** download di software, email autenticate, documenti e transazioni elettroniche

---

## Crittografia per Stato dei Dati

| Stato | Descrizione | Protezione |
|---|---|---|
| **A riposo** (at rest) | Dati archiviati su disco, database, cloud storage | Cifratura del supporto → illeggibili senza chiave, anche se rubati fisicamente |
| **In transito** (in transit) | Dati che si spostano in rete, tra servizi, su Internet | TLS/HTTPS, VPN → protetti da intercettazione e eavesdropping |
| **In uso** (in use) | Dati caricati in RAM o elaborati dalla CPU | **Confidential Computing** → enclave sicure che elaborano i dati senza esporli nemmeno all'OS |

> ⚠️ La crittografia tradizionale copre *at rest* e *in transit*. I dati vengono normalmente decrittografati quando entrano in memoria, creando una finestra di esposizione — il **confidential computing** chiude anche quella.

---

## Gestione delle Chiavi

> La crittografia è **sicura quanto le sue chiavi**. Una chiave esposta annulla tutta la protezione.

### Best Practice

- 🗂️ **Archivia le chiavi separatamente dai dati** — chi ottiene i dati non deve ottenere automaticamente anche le chiavi
- 🔩 **Usa Hardware Security Module (HSM)** — dispositivi fisici resistenti alle manomissioni, progettati per custodire chiavi crittografiche
- 🔄 **Ruota le chiavi periodicamente** — limita i danni in caso di compromissione
- 🔒 **Controllo accesso rigoroso** — MFA + least privilege per chi accede alle chiavi

### Servizi Cloud

Strumenti come **Azure Key Vault** offrono gestione centralizzata di chiavi, certificati e segreti, con rotazione automatica e integrazione nativa con altri servizi — senza esporre il materiale crittografico al codice o agli amministratori.

---

## Hashing

> L'hashing è una funzione **unidirezionale** che trasforma qualsiasi input in una stringa a lunghezza fissa (hash o digest).

```
"password123" ──[algoritmo hash]──► "ef92b778..." (lunghezza fissa)
```

### Differenze rispetto alla crittografia

| | Crittografia | Hashing |
|---|---|---|
| Reversibile? | ✅ Sì (con chiave) | ❌ No |
| Usa chiavi? | ✅ Sì | ❌ No |
| Scopo principale | Proteggere i dati | Verificare l'integrità / autenticare |

### Come protegge le password

Anziché salvare la password in chiaro, il sistema salva **solo il suo hash**:

```
Login:  hash("password inserita") == hash archiviato? → accesso concesso
```

Anche se il database viene rubato, l'attaccante ottiene solo hash — non le password originali.

---

### ⚠️ Vulnerabilità: Attacchi con Tabelle Arcobaleno

Poiché le funzioni hash sono **deterministiche** (stesso input → stesso output), un attaccante può precompilare milioni di hash di password comuni e confrontarli con il database rubato (*rainbow table attack* o *dizionario*).

### 🧂 Soluzione: Salting

Un **salt** è un valore casuale univoco aggiunto alla password **prima** dell'hashing:

```
hash("password123" + "xK9#mQ2!") ──► hash completamente diverso
hash("password123" + "aZ4@nW7$") ──► hash completamente diverso
```

- Ogni utente ottiene un salt diverso → due utenti con la stessa password producono hash diversi
- Le tabelle arcobaleno precalcolate diventano inutili
- **Standard obbligatorio** per qualsiasi sistema di autenticazione correttamente implementato

---

## In Sintesi

```
CRITTOGRAFIA
├── Simmetrica   → 1 chiave, veloce, grandi dati (problema: distribuzione chiave)
├── Asimmetrica  → coppia pubblica/privata (HTTPS, email, firme digitali)
└── Per stato    → at rest | in transit | in use (confidential computing)

HASHING
├── Unidirezionale, niente chiavi, lunghezza fissa
├── Usato per: password, integrità file, firme
└── Salting → difesa contro rainbow table / attacchi dizionario
```

---
