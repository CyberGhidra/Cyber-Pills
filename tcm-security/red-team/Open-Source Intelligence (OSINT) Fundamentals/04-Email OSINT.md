# Open-Source Intelligence (OSINT) Fundamentals
## Email OSINT — 1. Discovering Email Addresses

La fase di **Email Discovery** consiste nell'individuare, correlare e verificare indirizzi e-mail aziendali o personali associati a un target, a un nome utente o a uno specifico dominio aziendale. Nelle attività OSINT consente di mappare l'organigramma di un'organizzazione, identificare i pattern di denominazione interni (naming convention) e trovare potenziali vettori per test di sicurezza o indagini digitali.

---

### Strumenti e Risorse di Riferimento

* **Motori di Ricerca Dominio ed Enumerazione E-mail:**
  * **[Hunter.io](https://hunter.io/)**  
    Piattaforma di riferimento per la *Domain Search*: analizza il web per indicizzare tutti gli indirizzi e-mail associati a un'azienda, rivelando lo schema di denominazione più probabile (es. `{first}.{last}@target.com`) e le fonti pubbliche dove l'indirizzo è comparso.
  * **[Phonebook.cz](https://phonebook.cz/)**  
    Servizio gestito da Intelligence X per interrogare gratuitamente enormi collezioni di dati pubblici, dump e registri. Permette di estrarre tutte le e-mail, i sottodomini o gli URL collegati a un dominio target con un solo clic.
  * **[VoilaNorbert](https://www.voilanorbert.com/)**  
    Tool concepito per rintracciare l'indirizzo e-mail di una persona specifica fornendo come dati di input: **Nome**, **Cognome** e **Dominio aziendale**.

* **Estensioni Browser per Prospecting e Arricchimento Dati:**
  * **[Clearbit Connect (Chrome Web Store)](https://chrome.google.com/webstore/detail/clearbit-connect-supercha/pmnhcgfcafcnkbengdcanjablaabjplo?hl=en)**  
    Estensione per Chrome integrata direttamente in Gmail che consente di trovare recapiti e profili aziendali semplicemente digitando il nome della compagnia o il nominativo del dipendente.

* **Servizi di Verifica e Convalida della Casella Postale:**
  * **[Email Hippo](https://tools.verifyemailaddress.io/)**  
    Tool per la validazione in tempo reale dell'indirizzo e-mail senza l'invio effettivo di un messaggio (verifica della sintassi, presenza dei record MX DNS e test di handshake SMTP per confermare l'esistenza della casella postale).
  * **[Email Checker](https://email-checker.net/validate)**  
    Servizio di convalida immediata che verifica se un indirizzo e-mail è attivo, se il server di posta accetta connessioni o se si tratta di una casella "catch-all" / inesistente.

---

### Metodologia Operativa nel Processo di Discovery

1. **Identificazione del Dominio e del Pattern Aziendale:**  
   Si interroga [Hunter.io](https://hunter.io/) o [Phonebook.cz](https://phonebook.cz/) inserendo il dominio bersaglio (`target.com`) per estrarre le e-mail già indicizzate e determinare lo schema predefinito usato dall'organizzazione (es. `m.rossi@target.com` o `mario.rossi@target.com`).
2. **Generazione e Ricerca Mirata dei Nominativi:**  
   Avendo a disposizione nome e cognome ricavati da OSINT su LinkedIn o organigrammi aziendali, si impiega [VoilaNorbert](https://www.voilanorbert.com/) o combinazioni manuali basate sul pattern scoperto per generare l'indirizzo candidato.
3. **Validazione Senza Interazione Diretta:**  
   Prima di archiviare l'indirizzo nelle prove dell'indagine, lo si passa attraverso [Email Hippo](https://tools.verifyemailaddress.io/) o [Email Checker](https://email-checker.net/validate) per testare l'handshake SMTP con i record MX del provider ed escludere falsi positivi senza allertare il destinatario con messaggi non recapitabili (bounce).
