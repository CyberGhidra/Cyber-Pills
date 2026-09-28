# 01 - La Logica di Base della ISO 27001

> **Tema centrale:** Come funziona concretamente la sicurezza delle informazioni secondo lo standard.

---

## 1. Approccio Prescrittivo vs Approccio Basato sul Rischio

Uno dei principali errori di chi si approccia alla ISO 27001 è pensare che lo standard contenga ricette o requisiti tecnici dettagliati (es. frequenza dei backup, distanza del sito di disaster recovery, configurazioni dei router o brand tecnologici da adottare).

* **Perché lo standard non è prescrittivo?**  
  Ogni organizzazione ha esigenze diverse. Un backup ogni 24 ore può risultare eccessivo per chi manipola pochi dati statici, ma completamente inadeguato per chi registra migliaia di transazioni al secondo e necessita di backup orari o in tempo reale.
* **La vera funzione della ISO 27001:**  
  Fornire un **framework sistemico** per:
  1. Identificare cosa può andare storto (**Risk Assessment** / Valutazione del rischio).
  2. Stabilire quali contromisure adottare per evitarlo (**Risk Treatment** / Trattamento del rischio).

---

## 2. Il Principio di Selezione dei Controlli

La sicurezza deve essere un **abito su misura** (*tailor-made*):

| Cosa FARE | Cosa NON FARE |
|---|---|
| Implementare **tutti** i controlli necessari emersi dalla valutazione del rischio | Implementare controlli solo perché considerati "di tendenza" o moderni |
| Giustificare ogni salvaguardia in funzione di un rischio reale | Escludere controlli necessari solo perché sgraditi, complessi o scomodi |

---

## 3. La Sicurezza non è solo un Problema IT

L'IT da solo rappresenta circa il **50%** della sicurezza delle informazioni. La maggior parte degli incidenti non deriva da guasti hardware, ma da errori operativi, usi impropri o comportamenti non corretti da parte del personale di business.

Inoltre, molte informazioni critiche risiedono ancora su supporti non digitali (es. documenti cartacei). Per questo, la sicurezza richiede una difesa su più livelli:
