# Descrivere le funzionalità di gestione e protezione delle password

## Cos’è la protezione delle password in Microsoft Entra ID

La **protezione delle password** in : è una funzionalità che riduce il rischio legato all’uso di password deboli o facilmente indovinabili.

Il sistema:

- Blocca password vulnerabili conosciute
- Blocca varianti di password deboli
- Permette di definire regole specifiche aziendali
- Applica controlli sia in cloud sia (se configurato) in ambienti ibridi

L’obiettivo è prevenire attacchi basati su password come il **password spraying**.

---

## Elenco globale delle password vietate

Microsoft mantiene un **elenco globale di password escluse** basato su:

- Telemetria di sicurezza reale
- Attacchi password spraying osservati
- Password comunemente usate e compromesse

### Caratteristiche principali

- Applicato automaticamente a tutti i tenant
- Non richiede configurazione
- Non può essere disabilitato
- Viene aggiornato continuamente da Microsoft

Il sistema usa anche tecniche di **fuzzy matching**, quindi blocca non solo password identiche, ma anche varianti simili.

---

## Elenco personalizzato di password vietate

Le organizzazioni possono aggiungere un **elenco personalizzato** per bloccare termini specifici legati al contesto aziendale.

Esempi:

- Nome dell’azienda
- Brand e prodotti
- Sedi geografiche
- Abbreviazioni interne
- Termini aziendali sensibili

### Limiti

- Massimo 1.000 termini
- Non è pensato per elenchi enormi
- È più efficace usare parole chiave strategiche (non liste lunghe)

L’elenco personalizzato viene combinato con quello globale.

---

## Come viene valutata una password

Quando un utente crea o modifica una password, il sistema esegue diversi passaggi.

### 1. Normalizzazione
La password viene trasformata:

- Tutto in minuscolo
- Sostituzioni comuni (es. @ → a, 0 → o, 1 → l)

---

### 2. Confronto fuzzy

Viene confrontata con le password vietate:

- Differenza di massimo 1 modifica (inserimento, rimozione o sostituzione)
- Rileva anche varianti molto simili

---

### 3. Controllo su dati personali

La password viene confrontata con:

- Nome
- Cognome
- Nome tenant

Se contiene questi elementi, viene bloccata.

---

### 4. Sistema di punteggio

La password deve raggiungere un punteggio minimo:

- +1 punto per ogni carattere valido
- +1 punto per elementi vietati rilevati
- Deve raggiungere almeno 5 punti per essere accettata

Questo consente password lunghe anche se contengono elementi deboli, purché siano sufficientemente complesse.

---

## Protezione contro password spraying

Il **password spraying** è un attacco in cui:

- Si usano password comuni
- Si tenta l’accesso su molti account
- Si evita il blocco account distribuendo i tentativi

La protezione password di Entra:

- Blocca password comunemente usate
- Riduce drasticamente il successo di questi attacchi
- Usa dati reali sugli attacchi osservati

---

## Integrazione con Active Directory locale

In ambienti ibridi, la protezione può essere estesa a:

### Componenti principali

- **Proxy service**
  - Inoltra richieste tra AD locale e Entra ID

- **DC Agent**
  - Installato sui Domain Controller
  - Valida le password usando i criteri cloud

### Funzionamento

- Il Domain Controller non comunica direttamente con Internet
- I criteri vengono scaricati tramite proxy
- Vengono aggiornati periodicamente (circa ogni ora)
- Le regole cloud e locali devono entrambe approvare la password

---

## Ruolo nella sicurezza complessiva

La protezione password non sostituisce altri controlli, ma si integra con:

- Autenticazione a più fattori
- Accesso condizionale
- Metodi senza password

👉 Anche password complesse da sole non sono sufficienti: questa funzionalità è solo un livello della strategia Zero Trust.

---
