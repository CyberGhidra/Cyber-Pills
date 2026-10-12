# Open-Source Intelligence (OSINT) Fundamentals
## Username OSINT — Cap-01: Hunting Usernames and Accounts

L'analisi degli **Username** costituisce uno dei vettori d'ingresso primari nelle investigazioni OSINT. La maggior parte degli individui tende a riutilizzare lo stesso pseudonimo, handle o combinazione alfanumerica attraverso molteplici servizi web (social network, forum tecnici, piattaforme di gaming, marketplace e repository). 

La fase di **Username Hunting** punta a:
* Mappare l'impronta digitale complessiva (*digital footprint*) associata a un target.
* Rintracciare account secondari o dimenticati contenenti dati personali residui (vecchi avatar, biografie, e-mail visibili, località).
* Verificare la disponibilità o l'occupazione di uno specifico alias per escludere o confermare omonimie.

---

### Strumenti e Servizi di Ricerca

* **[NameChk](https://namechk.com/)**
  * **Scopo:** Motore di ricerca rapido per la disponibilità di username e domini web.
  * **Funzionamento:** Interroga simultaneamente decine di social network (Facebook, Instagram, YouTube, Tumblr, WordPress, ecc.) evidenziando se l'handle inserito è già registrato (*Taken*) o libero (*Available*).
  * **Utilità OSINT:** Permette una panoramica immediata dei principali canali consumer occupati dal target.

* **[WhatsMyName](https://whatsmyname.app/)**
  * **Scopo:** Framework open-source specializzato nell'individuazione e validazione di account su oltre 500+ piattaforme online.
  * **Punto di forza:** Mantenuto costantemente dalla community OSINT (progetto curato originariamente da Micah Hoffman / WebBreacher), effettua controlli basati su status code HTTP e stringhe di verifica specifiche nel codice HTML per minimizzare i falsi positivi.
  * **Restituzione dati:** Fornisce l'elenco diretto dei link URL ai profili trovati (es. Reddit, TikTok, GitHub, Medium, Keybase, Steam).

* **[NameCheckup](https://namecheckup.com/)**
  * **Scopo:** Tool di ricognizione parallelo per il controllo di nomi utente e brand su piattaforme social e network professionali.
  * **Funzionamento:** Invia query asincrone verso un ampio catalogo di servizi digitali, segnalando la presenza o l'assenza del nome specificato.

---

### Tabella Comparativa degli Strumenti

| Piattaforma | Copertura Siti | Risultato Restituito | Peculiarità OSINT |
| :--- | :--- | :--- | :--- |
| **NameChk** | Piattaforme mainstream & domini | Stato (Occupato / Libero) | Controllo rapido cross-platform orientato anche a domini TLD. |
| **WhatsMyName** | Ampia (500+ servizi, forum, dev, gaming) | URL diretto al profilo attivo | Altissima precisione; motore integrato in tool CLI avanzati. |
| **NameCheckup** | Social network principali | Stato (Occupato / Libero) | Alternativa rapida per riscontro incrociato e verifica ridondante. |

---

### Metodologia Operativa Consigliata

1. **Scansione Multi-Strumento:**
   Non basarsi mai sui dati di una sola piattaforma. Eseguire sempre un controllo combinato (ad esempio validando i risultati di *NameChk* tramite *WhatsMyName*) per evitare discrepanze dovute a rate-limiting o blocchi anti-scraping delle piattaforme target.
2. **Ispezione Manuale dei Profili Trovati (Ground-Truth Check):**
   * Verificare la coerenza anagrafica: lo stesso username potrebbe appartenere a due persone completamente diverse in paesi differenti.
   * Controllare gli indicatori visivi e testuali: stile dell'avatar, bio, data di creazione dell'account, contatti menzionati, fuso orario di pubblicazione dei post.
3. **Generazione di Permutazioni:**
   Se il nome utente standard non produce risultati o sembra generico, testare le varianti tipiche adottate dagli utenti:
   * Aggiunta di prefissi/suffissi (es. `user_official`, `real_user`, `user99`).
   * Separatori (es. `nome.cognome`, `nome_cognome`, `nome-cognome`).
   * Variazioni leet-speak (es. sostituzione di vocali con numeri: `o` $\rightarrow$ `0`, `e` $\rightarrow$ `3`).
