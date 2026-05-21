# Autenticazione e Autorizzazione

## Introduzione

La sicurezza dell’accesso alle risorse digitali si basa su due processi fondamentali e complementari:

- **Autenticazione** → verifica l’identità dell’utente, dispositivo o applicazione
- **Autorizzazione** → determina cosa quell’identità può fare

Questi due processi lavorano insieme per garantire che solo le identità corrette accedano alle risorse appropriate.

---

## 1. Autenticazione

### Definizione
L’autenticazione è il processo che conferma **chi sei realmente**.

Risponde alla domanda:
> **“Chi sei?”**

Avviene ogni volta che:
- si accede a un’applicazione
- si sblocca un dispositivo
- si entra in un sistema aziendale

---

### Credenziali e fattori di autenticazione

L’autenticazione si basa su **credenziali**, cioè prove dell’identità.

Tuttavia, un singolo elemento (come un username) non è sufficiente.

#### I tre fattori di autenticazione:

1. **Qualcosa che sai**
   - Password
   - PIN
   - Domande di sicurezza

2. **Qualcosa che possiedi**
   - Smartphone
   - Chiave hardware
   - Smart card

3. **Qualcosa che sei**
   - Impronta digitale
   - Riconoscimento facciale
   - Biometria

---

### Tipi di autenticazione

#### Autenticazione a singolo fattore (SFA)
- Usa un solo fattore (es. password)
- È semplice ma **poco sicura**
- Vulnerabile a phishing, furto o indovinamento

#### Autenticazione a più fattori (MFA)
- Usa **due o più fattori diversi**
- Esempio: password + conferma sul telefono
- Molto più sicura perché richiede più prove indipendenti

---

### Esito dell’autenticazione
- **Successo** → l’identità viene confermata
- **Fallimento** → l’accesso viene negato

⚠️ Importante: l’autenticazione NON decide cosa puoi fare, solo chi sei.

---

## 2. Autorizzazione

### Definizione
L’autorizzazione è il processo che determina **cosa puoi fare dopo che sei stato autenticato**.

Risponde alla domanda:
> **“Cosa ti è permesso fare?”**

---

### Come funziona

L’autorizzazione valuta:
- **Ruoli** (utente, manager, amministratore)
- **Permessi** (lettura, scrittura, eliminazione)
- **Attributi** (reparto, posizione, funzione lavorativa)

In base a queste regole decide:
- accesso consentito
- accesso negato
- livello di accesso alle risorse

---

### Principio dei privilegi minimi

Uno dei concetti fondamentali dell’autorizzazione è il:

> **Least Privilege Principle**

Significa che ogni utente deve avere **solo i permessi strettamente necessari** per svolgere il proprio lavoro.

#### Vantaggi:
- Riduce i danni in caso di account compromesso
- Limita errori umani
- Migliora sicurezza e conformità

---

## 3. Relazione tra autenticazione e autorizzazione

I due processi sono distinti ma strettamente collegati.

### Ordine corretto:
1. **Autenticazione** → identifica l’utente
2. **Autorizzazione** → decide cosa può fare

---

### Perché l’ordine è importante

- Senza autenticazione → non puoi sapere chi sta facendo la richiesta
- Senza autorizzazione → tutti avrebbero accesso a tutto

---

### Esempio pratico (hotel)

- **Autenticazione** → mostri un documento alla reception
- **Autorizzazione** → ricevi una chiave che apre solo:
  - la tua stanza
  - alcune aree autorizzate (es. palestra)

Non puoi accedere alle stanze degli altri ospiti o aree riservate.

---

## 4. Sintesi finale

- L’**autenticazione** verifica l’identità (chi sei)
- L’**autorizzazione** definisce i permessi (cosa puoi fare)
- L’autenticazione può essere singola o multi-fattore (MFA)
- L’autorizzazione si basa su ruoli, permessi e attributi
- Il principio dei privilegi minimi riduce i rischi di sicurezza
- Entrambi sono essenziali per proteggere le risorse digitali moderne
