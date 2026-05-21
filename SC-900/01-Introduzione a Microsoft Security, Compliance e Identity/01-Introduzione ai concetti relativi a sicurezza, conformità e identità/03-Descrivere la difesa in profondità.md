# Difesa in profondità (Defense in Depth)

La difesa in profondità è una strategia di sicurezza che utilizza **più livelli di protezione sovrapposti** per proteggere sistemi, dati e reti.

L’idea fondamentale è:

> Non esiste un singolo controllo di sicurezza perfetto → quindi si costruiscono più barriere.

Se un livello viene superato da un attaccante, gli altri livelli continuano a proteggere il sistema e a rallentare l’attacco.

---

## 🧱 Concetto base

È simile a un sistema di sicurezza fisico (es. banca o caveau):

- Porta esterna blindata
- Telecamere
- Sensori di movimento
- Badge di accesso
- Porta del caveau
- Cassaforti interne

👉 Anche se un livello viene violato, gli altri continuano a difendere.

--- 

## 🔐 Livelli della difesa in profondità

### 1. Sicurezza fisica
Protegge l’infrastruttura hardware.

Include:
- Accesso controllato ai data center
- Badge, guardie, telecamere
- Blocco fisico dei server

📌 Obiettivo: impedire accesso fisico non autorizzato

---

### 2. Identità e accesso (IAM)
Controlla chi può accedere ai sistemi.

Include:
- Autenticazione (password, MFA)
- Autorizzazione (ruoli e permessi)
- Accesso condizionale (posizione, dispositivo, rischio)

📌 Obiettivo: solo utenti autorizzati accedono alle risorse

---

### 3. Sicurezza perimetrale
Protegge il confine tra rete interna e Internet.

Include:
- Firewall
- Protezione DDoS
- Filtri del traffico in ingresso/uscita

📌 Obiettivo: bloccare attacchi prima che entrino nella rete

---

### 4. Sicurezza di rete
Protegge la comunicazione interna tra sistemi.

Include:
- Segmentazione della rete (divisione in zone)
- Regole di traffico (NSG, ACL)
- Isolamento tra sistemi

📌 Obiettivo: limitare il movimento laterale di un attaccante

---

### 5. Sicurezza di calcolo
Protegge server, VM e container.

Include:
- Patch e aggiornamenti
- Hardening del sistema operativo
- Chiusura porte inutilizzate
- Monitoraggio attività sospette

📌 Obiettivo: ridurre vulnerabilità nei sistemi attivi

---

### 6. Sicurezza applicativa
Protegge le applicazioni software.

Include:
- Sviluppo sicuro (Secure SDLC)
- Validazione input (anti SQL injection, XSS)
- Autenticazione e autorizzazione a livello app
- Test di sicurezza (SAST/DAST)

📌 Obiettivo: prevenire exploit nelle applicazioni

---

### 7. Sicurezza dei dati
È il livello più importante e interno.

Include:
- Crittografia dei dati
- Controllo accessi ai dati
- Classificazione delle informazioni (pubblico, riservato, critico)

📌 Obiettivo: proteggere ciò che ha più valore

---

## 🧠 Perché è importante

La difesa in profondità è fondamentale perché:

- Nessun sistema è perfetto
- Gli attacchi possono bypassare singoli controlli
- Errori di configurazione o bug possono esistere sempre

👉 Più livelli = più difficoltà per l’attaccante

---

## ⏱️ Effetto pratico della difesa a strati

Ogni livello:

- rallenta l’attaccante
- aumenta le possibilità di rilevamento
- riduce il danno finale

📌 Anche se un attacco riesce a entrare, non significa che possa muoversi liberamente.

---

## 🎯 Collegamento con la triade CIA

La difesa in profondità protegge tre obiettivi fondamentali:

### 🔒 Riservatezza (Confidentiality)
Solo utenti autorizzati possono vedere i dati

### 🧩 Integrità (Integrity)
I dati non vengono modificati o corrotti

### ⏱️ Disponibilità (Availability)
Sistemi e dati sono accessibili quando servono

---

## ⚖️ Equilibrio della CIA

Le tre proprietà devono essere bilanciate:

- Troppa sicurezza → sistemi lenti o difficili da usare
- Troppa disponibilità → meno sicurezza
- Troppa crittografia → rischio perdita accesso ai dati

📌 La sicurezza efficace è sempre un compromesso controllato.

---

## 🎯 In sintesi

La difesa in profondità significa:

> Usare più livelli di sicurezza indipendenti per proteggere sistemi e dati, assumendo che almeno uno possa fallire.

Obiettivo finale:
- rallentare gli attaccanti
- limitarne i danni
- proteggere dati, identità e sistemi anche in caso di compromissione
