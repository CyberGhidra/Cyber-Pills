# Appunti e Sintesi ISO/IEC 27001

Raccolta strutturata dei concetti chiave, requisiti e controlli dello standard ISO/IEC 27001 (aggiornato all'edizione 2022).

---

## Modulo 1: La Logica di Base della ISO 27001

### 1. Approccio Prescrittivo vs Basato sul Rischio

Il malinteso più comune è credere che la ISO 27001 indichi requisiti tecnici precisi (es. ogni quante ore fare il backup, a quanti km collocare il sito di disaster recovery o quali apparati di rete acquistare).

* **Perché NON è prescrittiva:** Ogni azienda ha un contesto differente. Un backup ogni 24 ore è un costo inutile per chi gestisce dati statici, ma è insufficiente per una banca o un e-commerce ad alto traffico che richiede backup orari o continui.
* **Cosa fa invece la ISO 27001:** Fornisce un **framework sistemico** per:
  1. Identificare cosa può andare storto (**Risk Assessment** / Valutazione del rischio).
  2. Decidere quali contromisure implementare per mitigarlo (**Risk Treatment** / Trattamento del rischio).

---

### 2. Criterio di Scelta dei Controlli

La sicurezza deve essere un **abito su misura** (*tailor-made*):

| Regola | Principio guida |
|---|---|
| **Cosa fare** | Implementare tutti i controlli resi necessari dall'analisi del rischio. |
| **Cosa NON fare** | Implementare controlli solo perché "di moda" o escluderne altri solo perché complessi o sgraditi. |

---

### 3. IT Security vs Information Security

L'IT copre solo circa il **50%** della sicurezza complessiva:
* La maggioranza degli incidenti non dipende da guasti tecnici, ma dall'uso errato dei sistemi da parte degli utenti interni.
* Parte delle informazioni critiche non è in formato digitale (es. archivi cartacei, accordi confidenziali).
* Per funzionare, le misure tecniche devono essere affiancate da:
  - Policy e procedure operative chiare.
  - Corsi di formazione e consapevolezza (*awareness*).
  - Clausole contrattuali, aspetti legali e misure disciplinari.

---

### 4. Responsabilità del Top Management

Senza l'impegno concreto dei vertici aziendali, le misure di sicurezza non vengono applicate dal resto dell'organizzazione. La ISO 27001 stabilisce compiti precisi per la Direzione:

* [ ] **Obiettivi:** Fissare le aspettative e gli obiettivi di sicurezza allineati al business.
* [ ] **Policy:** Emettere e diffondere la politica di sicurezza dell'informazione.
* [ ] **Ruoli:** Assegnare formalmente le responsabilità del sistema.
* [ ] **Risorse:** Stanziare budget economico e personale sufficiente.
* [ ] **Riesame:** Verificare periodicamente se i risultati corrispondono agli obiettivi.

---

### 5. Prevenzione del Degrado e Miglioramento Continuo

Con il tempo, le tecnologie cambiano, l'organigramma muta e i progetti di sicurezza rischiano l'obsolescenza e l'abbandono. Per evitarlo, la ISO 27001 include processi ciclici obbligatori:

1. **Monitoraggio e Misurazione:** Tracciamento periodico dell'efficacia delle misure.
2. **Audit Interni:** Controlli regolari e imparziali sullo stato di conformità.
3. **Azioni Correttive:** Risoluzione definitiva delle cause alla radice dei problemi emersi.
