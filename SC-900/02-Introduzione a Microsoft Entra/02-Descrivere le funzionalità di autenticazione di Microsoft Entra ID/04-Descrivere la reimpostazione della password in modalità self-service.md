# Descrivere la reimpostazione della password in modalità self-service (SSPR)

## Cos’è la reimpostazione della password self-service

La **Self-Service Password Reset (SSPR)** è una funzionalità che consente agli utenti di modificare o reimpostare autonomamente la password senza l’intervento dell’help desk o degli amministratori IT.

Se un account viene bloccato oppure l’utente dimentica la password, può seguire una procedura guidata per recuperare rapidamente l’accesso al proprio account.

I vantaggi principali sono:

- Riduzione delle richieste all’help desk
- Maggiore produttività degli utenti
- Ripristino rapido dell’accesso
- Miglior tracciabilità tramite log e audit
- Integrazione con sistemi SIEM tramite API

---

# Funzionamento della SSPR

Quando un utente avvia il processo di reimpostazione password, la piattaforma Microsoft Entra esegue diversi controlli:

1. Verifica che l’account esista e sia valido
2. Controlla che la SSPR sia abilitata per l’utente
3. Verifica che siano registrati i metodi di autenticazione richiesti
4. Determina dove viene gestita la password (cloud o Active Directory locale)

Se tutti i controlli vengono superati, l’utente può:

- Reimpostare la password
- Modificare la password
- Sbloccare l’account

---

# Requisiti per usare SSPR

Per utilizzare la reimpostazione della password self-service, l’utente deve:

- Avere una licenza Microsoft Entra ID
- Essere abilitato alla SSPR da un amministratore
- Registrare almeno un metodo di autenticazione

Microsoft consiglia di configurare **due o più metodi di autenticazione** per evitare blocchi di accesso.

---

# Metodi di autenticazione supportati per SSPR

Microsoft Entra ID supporta i seguenti metodi:

- Notifiche push di Microsoft Authenticator
- Token software OATH
- Token hardware OATH (preview)
- SMS
- Chiamata vocale
- OTP via email
- Domande di sicurezza *(ritiro previsto marzo 2027)*

## Importante

Le **domande di sicurezza** verranno ritirate a marzo 2027.

Dopo quella data:

- Non potranno più essere usate per reimpostare password
- Le organizzazioni dovranno usare metodi moderni alternativi

---

# Numero di metodi richiesti

Gli amministratori possono scegliere se richiedere:

- 1 metodo di autenticazione
- 2 metodi di autenticazione

## Nota sugli amministratori

Gli account amministrativi:

- Sono abilitati automaticamente alla SSPR
- Devono usare obbligatoriamente due fattori
- Non possono usare domande di sicurezza

---

# Registrazione e riconferma

Gli amministratori possono obbligare gli utenti a:

- Registrare i metodi di autenticazione al primo accesso
- Riconfermare periodicamente le informazioni registrate

L’intervallo di riconferma può essere configurato da:

- 0 giorni
- fino a 730 giorni

Questo aiuta a mantenere aggiornati numeri di telefono, email e dispositivi.

---

# Notifiche

La SSPR può inviare notifiche email per informare utenti e amministratori.

## Utenti

Gli utenti ricevono una mail quando:

- La password viene modificata
- La password viene reimpostata

## Amministratori globali

Ricevono una notifica quando:

- Un altro amministratore usa SSPR per reimpostare la password

Questo aumenta il monitoraggio sugli account privilegiati.

---

# Integrazione con Active Directory locale

Negli ambienti ibridi, :contentReference[oaicite:1]{index=1} permette il **password writeback**.

## Password writeback

Con il writeback:

- La password cambiata nel cloud viene sincronizzata anche in Active Directory locale
- Gli utenti possono usare una sola password sia cloud che on-premises

Supporta ambienti con:

- Password Hash Sync
- Pass-through Authentication
- Federazione

Gli amministratori possono anche consentire:

- Solo lo sblocco dell’account
- Reimpostazione completa della password

---

# Ripristino dell’account

La SSPR funziona solo se l’utente possiede ancora almeno un metodo di autenticazione registrato.

## Problema

Se l’utente perde:

- Telefono
- Token
- Accesso email
- Tutti i metodi MFA

la SSPR non è più sufficiente.

---

# Account Recovery

Per questi casi esiste il **ripristino dell’account**.

Il recupero account consente di:

- Verificare nuovamente l’identità dell’utente
- Ripristinare l’accesso anche senza metodi MFA disponibili

---

# Differenze tra SSPR e Account Recovery

| Aspetto | SSPR | Account Recovery |
|---|---|---|
| Scenario | Password dimenticata | Perdita totale dei metodi MFA |
| Requisito | Metodo registrato disponibile | Verifica completa identità |
| Obiettivo | Reimpostare password | Ripristinare accesso completo |
| Sicurezza | Basata su metodi preregistrati | Verifica identità avanzata |

---

# Microsoft Entra Verified ID e Face Check

Il recupero account usa:

- :contentReference[oaicite:2]{index=2}
- Face Check basato su servizi AI di Azure

Questa soluzione:

- Riduce il rischio di social engineering
- Elimina verifiche manuali dell’help desk
- Migliora la sicurezza
- Riduce i costi operativi

È particolarmente utile per:

- Organizzazioni distribuite
- Lavoratori remoti
- Ambienti enterprise ad alta sicurezza
