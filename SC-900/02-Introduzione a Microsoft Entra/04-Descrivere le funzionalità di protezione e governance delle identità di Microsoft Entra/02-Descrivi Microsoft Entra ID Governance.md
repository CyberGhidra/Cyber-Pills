# Microsoft Entra ID Governance

Microsoft Entra ID Governance è una soluzione di governance delle identità che consente alle organizzazioni di migliorare la produttività, rafforzare la sicurezza e soddisfare più facilmente i requisiti normativi e di conformità. Sfrutta informazioni basate sull’intelligenza artificiale per garantire automaticamente che le persone giuste abbiano accesso alle risorse appropriate.

Questo obiettivo viene raggiunto tramite:
- Automazione dei processi di identità e accesso  
- Delega ai gruppi aziendali  
- Maggiore visibilità e controllo  

## Obiettivi principali della governance delle identità

La governance delle identità aiuta a rispondere a quattro domande fondamentali:

- Quali identità devono avere accesso a quali risorse?
- Che cosa fanno le identità con tale accesso?
- Sono presenti controlli organizzativi per la gestione dell’accesso?
- I revisori possono verificare l’efficacia dei controlli?

Con Microsoft Entra ID Governance è possibile implementare scenari per:
- Dipendenti
- Partner commerciali
- Fornitori

---

## Gestione del ciclo di vita delle identità

La gestione del ciclo di vita delle identità è centrale nella governance.

Il ciclo tipico di un utente include:
- **Joiner**: ingresso nell’organizzazione → creazione identità digitale
- **Mover**: cambio ruolo → aggiornamento permessi
- **Leaver**: uscita → rimozione accesso e possibile eliminazione identità

Molte organizzazioni integrano questo ciclo con sistemi HR (es. Workday o SuccessFactors).

### Automazione del ciclo di vita

Microsoft Entra ID Governance consente di automatizzare il ciclo di vita tramite:

- **Provisioning in ingresso (HR-driven)**  
  Creazione e gestione automatica delle identità in Entra ID e Active Directory

- **Lifecycle Workflows**  
  Automazione di attività in momenti chiave (onboarding, cambi ruolo, offboarding)

- **Criteri di assegnazione automatica**  
  Assegnazione/rimozione automatica di gruppi, app e ruoli SharePoint in base agli attributi utente

- **Provisioning verso applicazioni**  
  Gestione automatica degli account su app cloud e on-premises tramite connettori

---

## Ciclo di vita degli accessi

Il ciclo di vita degli accessi riguarda la gestione continua dei diritti di accesso durante il tempo in azienda.

Microsoft Entra ID Governance supporta:

- **Gruppi dinamici**  
  Regole basate su attributi utente che aggiornano automaticamente l’appartenenza ai gruppi

- **Entitlement Management (gestione diritti)**  
  Accesso tramite pacchetti che includono gruppi, app e ruoli (anche con controllo separazione dei compiti)

- **Access Reviews (revisioni accessi)**  
  Ricertificazione periodica dei diritti di accesso con suggerimenti basati su AI

---

## Accesso con privilegi (Privileged Access Lifecycle)

La gestione degli accessi amministrativi è fondamentale per ridurre i rischi.

Microsoft Entra Privileged Identity Management (PIM) permette di:
- Ridurre il numero di utenti con privilegi elevati
- Assegnare accessi temporanei o just-in-time
- Monitorare e controllare ruoli amministrativi in Entra, Azure e altri servizi Microsoft

---

## Governance per agenti di intelligenza artificiale

Con l’aumento dell’uso dell’IA, Microsoft Entra ID Governance estende il controllo anche agli agenti AI.

Le identità degli agenti:
- Sono identità digitali dedicate agli agenti AI
- Hanno autenticazione e autorizzazione proprie
- Sono gestite come identità umane

Questo garantisce:
- Controllo del ciclo di vita degli agenti
- Supervisione umana
- Accesso limitato nel tempo e alle sole risorse necessarie
