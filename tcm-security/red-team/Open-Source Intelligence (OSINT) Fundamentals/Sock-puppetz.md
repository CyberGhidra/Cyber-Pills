# Open-Source Intelligence (OSINT) Fundamentals
## 1. Creating Sock Puppets

Un **sock puppet** è un'identità fittizia creata specificamente per condurre indagini OSINT e HUMINT senza esporre la propria identità reale, compromettere la sicurezza operativa (OPSEC) o allertare i soggetti d'indagine.

---

### Fondamenti di un Sock Puppet Efficace

* **Separazione Rigorosa (Compartimentazione):** Nessun elemento del profilo fittizio deve ricollegarsi alla vita personale o professionale dell'investigatore (stesso numero di telefono, e-mail di recupero, pattern di password o rete domestica).
* **Costruzione di una Backstory Credibile:** Un account privo di cronologia, contatti, foto o interazioni sembra immediatamente sospetto e viene spesso bloccato dagli algoritmi anti-bot delle piattaforme.
* **Isolamento dell'Ambiente Operativo:** Utilizzare browser dedicati, macchine virtuali isolate, VPN o reti mobili per evitare il fingerprinting del dispositivo reale.

---

### Fasi Operative di Configurazione

1. **Definizione dell'Identità Anagrafica:** Generazione di dati coerenti (nome, data di nascita, indirizzo plausibile, occupazione) compatibili con il contesto geografico e socioculturale del target.
2. **Generazione dell'Immagine Profilo:** Uso di volti generati tramite intelligenza artificiale per evitare violazioni di copyright o corrispondenze inverse tramite *reverse image search*. È necessario verificare che l'immagine non contenga artefatti visivi tipici delle GAN (orecchini asimmetrici, sfondi deformati, occhi non allineati).
3. **Canali di Registrazione e Verifica:**
   * Utilizzo di indirizzi e-mail anonimi o usa-e-getta dedicati esclusivamente al puppet.
   * Numeri di telefono "burner" o virtuali per superare i controlli SMS/2FA imposti dalle piattaforme social.
   * Metodi di pagamento virtuali isolati qualora sia necessario attivare account di prova o servizi a pagamento senza rivelare coordinate bancarie personali.
4. **Maturazione del Profilo (Aging & In-Character Activity):** L'account va popolato nel tempo con iscrizioni a gruppi, feed seguiti, condivisioni occasionali e collegamenti organici prima di impiegarlo in attività investigative attive.

---

### Risorse e Strumenti di Riferimento

* **Guide & Metodologie OSINT:**
  * [Creating an Effective Sock Puppet for OSINT Investigations – Introduction (Jake Creps / Archive)](https://web.archive.org/web/20210125191016/https://jakecreps.com/2018/11/02/sock-puppets/)
  * [The Art Of The Sock (Secjuice)](https://www.secjuice.com/the-art-of-the-sock-osint-humint/)
  * [Reddit - My process for setting up anonymous sockpuppet accounts (r/OSINT)](https://www.reddit.com/r/OSINT/comments/dp70jr/my_process_for_setting_up_anonymous_sockpuppet/)
* **Strumenti per la Generazione dell'Identità:**
  * [Fake Name Generator](https://www.fakenamegenerator.com/) — Creazione di profili anagrafici, indirizzi e dettagli coerenti.
  * [This Person Does Not Exist](https://www.thispersondoesnotexist.com/) — Generazione di volti fotorealistici sintetici tramite reti generative avversarie (GAN).
  * [Privacy.com](https://privacy.com/join/LADFC) — Creazione di carte di pagamento virtuali usa-e-getta o mascherate per registrazioni sicure.
