# Verifiche di accesso in Microsoft Entra

Le verifiche di accesso di Microsoft Entra consentono alle organizzazioni di gestire in modo efficiente:
- appartenenze ai gruppi
- accesso alle applicazioni aziendali
- assegnazioni di ruolo

Le verifiche periodiche assicurano che solo le persone corrette mantengano l’accesso alle risorse, riducendo il rischio di privilegi eccessivi.

---

## Perché sono importanti

Con Microsoft Entra ID, gli utenti possono:
- collaborare internamente ed esternamente
- accedere ad applicazioni cloud
- lavorare da dispositivi personali o aziendali
- essere aggiunti a gruppi e invitati come guest

Questa flessibilità aumenta la necessità di controlli regolari sugli accessi.

---

## Casi d’uso principali

Le verifiche di accesso sono utili in diversi scenari:

### Ruoli privilegiati eccessivi
- Controllo utenti con ruoli amministrativi
- Revisione di utenti guest o partner non più necessari
- Ricertificazione tramite Microsoft Entra Privileged Identity Management (PIM)

### Accesso a dati critici
- Richiesta di conferma periodica dell’accesso
- Necessità di giustificazione per mantenere accesso a risorse sensibili

### Eccezioni ai criteri di sicurezza
- Verifica periodica delle eccezioni ai criteri aziendali
- Produzione di evidenze per audit e compliance

### Gestione utenti guest nei gruppi
- I proprietari dei gruppi confermano la necessità degli accessi guest

### Revisioni periodiche
- Pianificazione settimanale, mensile, trimestrale o annuale
- Notifiche ai revisori
- Decisioni di approvazione o rifiuto con supporto AI

---

## Come funzionano le verifiche di accesso

Il processo include:

1. Creazione della verifica da parte dell’amministratore
2. Notifica ai revisori o ai proprietari delle risorse
3. Valutazione dell’accesso (approva o nega)
4. Applicazione delle modifiche al termine della verifica

Durante la verifica:
- nessun accesso viene modificato immediatamente
- è possibile monitorare lo stato della revisione
- la verifica può essere interrotta dall’amministratore

Dopo il completamento:
- le modifiche possono essere applicate automaticamente o manualmente
- eccezione: gruppi dinamici o sincronizzati on-premises (modifiche gestite alla fonte)

---

## Verifiche di accesso in più fasi

Microsoft Entra supporta fino a **tre fasi di revisione**, con diversi revisori coinvolti.

Vantaggi:
- processi di ricertificazione più complessi
- maggiore accuratezza delle decisioni
- distribuzione del carico tra più revisori
- migliore governance degli accessi

---

## Raccomandazioni basate sull’intelligenza artificiale

Le verifiche includono suggerimenti intelligenti basati su AI che analizzano:

- attività di accesso dell’utente
- appartenenza ai gruppi
- comportamento rispetto ai peer

L’AI può:
- suggerire approvazione o rifiuto
- identificare utenti “outlier” (comportamenti anomali rispetto ai colleghi)

---

## Obiettivo finale

Le verifiche di accesso garantiscono che:
- gli utenti mantengano solo gli accessi necessari
- i privilegi non necessari vengano rimossi
- la sicurezza e la conformità vengano migliorate nel tempo
