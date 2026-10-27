## Firewall authentication

- Identificazione di utenti e gruppi di utenti:
    - include l'autenticazione formale di singoli utenti o gruppi.
    - Maggiore affidabilità:
      autenticare gli utenti tramite credenziali valide offre un livello di sicurezza superiore rispetto al semplice riconoscimento basato sull'indirizzo IP o sulla tipologia di dispositivo (entrambi facilmente falsificabili o condivisi).
    - Flusso operativo:
      L'utente tenta di accedere a una risorsa di rete e il FortiGate richiede l'autenticazione presentando una schermata di login (Captive Portal / Authentication Required).
      L'utente inserisce le credenziali valide (Username e Password).
      Una volta accertata l'identità dell'utente o del dispositivo, FortiGate applica le relative Firewall Policy e i Security Profile per autorizzare o negare l'accesso alla specifica risorsa richiesta.
      
- Metodi di Autenticazione Firewall su FortiGateLa :
    - A. Local Password Authentication (Autenticazione Locale)
      Nome utente e password sono memorizzati e gestiti direttamente all'interno del database locale del FortiGate.
      Indicata per contesti con un numero ridotto di account o per utenze di emergenza.
    - B. Server-based Password Authentication (Autenticazione Remota)
      Chiamata anche autenticazione remota, delega la verifica delle credenziali a un server di directory o di autenticazione centralizzato.
      I protocolli/server supportati includono:
      LDAP (es. Microsoft Active Directory)
      RADIUS
      TACACS+
      POP3
    - C. Two-Factor Authentication (2FA / Autenticazione a Due Fattori)
      Viene attivata come livello aggiuntivo sopra uno dei metodi preesistenti (locale o basato su server).
      Richiede la combinazione di due fattori di sicurezza:
      Something you know (qualcosa che si conosce): password o PIN.
      Something you have (qualcosa che si possiede): token hardware/software (es. FortiToken Mobile) o certificato digitale.  
