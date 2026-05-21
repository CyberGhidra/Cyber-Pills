# Concetto di Federazione

## Introduzione

La federazione è un modello di gestione dell’identità che permette agli utenti di accedere a risorse di organizzazioni diverse utilizzando le proprie credenziali originali, senza creare nuovi account.

È la soluzione al problema dell’accesso tra sistemi di identità separati.

---

## 1. Cos’è la federazione

La **federazione** è un sistema che consente l’accesso tra organizzazioni o domini diversi stabilendo una **relazione di trust (fiducia)** tra i rispettivi provider di identità.

👉 In pratica:
- un’identità può essere riconosciuta fuori dalla propria organizzazione
- non è necessario creare nuovi account

---

### Idea chiave

> “Un’organizzazione si fida dell’identità autenticata da un’altra organizzazione.”

---

## 2. Come funziona la federazione

La federazione si basa su un principio di fiducia tra Identity Provider (IdP).

### Flusso semplificato:

1. L’utente si autentica nel proprio Identity Provider (Organizzazione B)
2. L’utente richiede accesso a una risorsa dell’Organizzazione A
3. L’Organizzazione A verifica la fiducia verso l’IdP di B
4. Accetta l’autenticazione già effettuata
5. L’utente ottiene accesso senza creare un nuovo account

---

### Analogia (passaporto)

La federazione è simile al sistema dei passaporti:

- il paese di origine emette il passaporto (identità)
- altri paesi lo accettano come prova di identità
- non serve ottenere un nuovo documento per ogni paese

---

## 3. Relazioni di trust

La federazione si basa su **relazioni di fiducia tra organizzazioni**.

### Importante:

- Il trust non è automaticamente bidirezionale
- Se A si fida di B, non significa che B si fida di A

👉 Per un trust reciproco serve configurazione esplicita.

---

## 4. Federazione in pratica

La federazione è resa possibile da protocolli standard di sicurezza che permettono lo scambio di token tra sistemi diversi.

---

## 5. Casi d’uso principali

### 1. Collaborazione B2B (Business-to-Business)

Quando due aziende collaborano:

- condividono applicazioni e risorse
- ogni azienda usa il proprio Identity Provider
- non si creano account separati per i partner

👉 Esempio:
- progetto comune
- area di lavoro condivisa
- portale collaborativo

---

### 2. Login con provider social

Molte app consentono login tramite:

- Google
- Microsoft
- GitHub

👉 L’app si fida del provider esterno per autenticare l’utente.

Vantaggi:
- meno registrazioni
- onboarding più rapido
- maggiore adozione

---

### 3. Integrazione con Active Directory locale

Le organizzazioni on-premises possono estendere l’identità al cloud.

- Active Directory locale
- Active Directory Federation Services (AD FS)

👉 AD FS permette:
- autenticazione locale
- accesso a servizi cloud tramite trust

---

## 6. Active Directory Federation Services (AD FS)

### Cos’è

AD FS è un servizio Microsoft che consente la federazione tra:

- Active Directory locale
- applicazioni esterne o cloud

---

### Funzione principale

- autentica utenti internamente
- emette token accettati da sistemi esterni

---

## 7. Federazione e Microsoft Entra ID

Microsoft Entra ID supporta la federazione per:

- aziende partner
- clienti
- utenti esterni
- applicazioni cloud

---

### Vantaggi

- nessuna creazione di account multipli
- accesso sicuro tra organizzazioni
- gestione centralizzata della fiducia
- riduzione della complessità per gli utenti

---

## 8. Sintesi finale

- La federazione consente accesso tra organizzazioni diverse
- Si basa su relazioni di trust tra Identity Provider
- Permette agli utenti di usare le proprie credenziali ovunque
- Evita la creazione di account duplicati
- È usata in scenari B2B, social login e cloud ibrido
- AD FS e Microsoft Entra ID supportano la federazione moderna

---

## Conclusione

La federazione estende il concetto di identità oltre i confini di una singola organizzazione, permettendo un accesso sicuro e semplificato tra sistemi diversi grazie alla fiducia tra provider di identità.
