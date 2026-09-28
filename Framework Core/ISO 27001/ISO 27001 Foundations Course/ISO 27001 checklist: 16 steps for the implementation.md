# 03 - Implementazione ISO 27001: Guida Operativa in 16 Passi
---

## 1. La Checklist dei 16 Passi per l'Implementazione

L'iter di conformità e certificazione alla ISO 27001 si articola in 16 passaggi sequenziali:

- [ ] **1. Ottenere il supporto del Top Management**  
  Causa primaria di fallimento del progetto: l'assenza di investimenti economici e la mancata allocazione di tempo/persone da parte della Direzione.
- [ ] **2. Gestire l'implementazione come un vero progetto (Project Management)**  
  Definire WBS, attività, ruoli, responsabilità, scadenze (*milestone*) e deliverable chiari fin dal primo giorno.
- [ ] **3. Definire lo Scope (Ambito del SGSI/ISMS)**  
  - *Aziende fino a 50 dipendenti:* conviene generalmente includere l'intera organizzazione.  
  - *Grandi organizzazioni:* è consigliabile limitare lo scope iniziale a un reparto o linea di business critica per ridurre i rischi di progetto.
- [ ] **4. Redigere l'Information Security Policy di vertice**  
  Documento quadro di alto livello non eccessivamente dettagliato, in cui la Direzione dichiara le aspettative strategiche per la sicurezza e i criteri di controllo generali.
- [ ] **5. Definire la metodologia di Risk Assessment**  
  Stabilire a priori i criteri formali: regole di identificazione delle minacce/vulnerabilità, scale di probabilità e impatto, e definizione della soglia di rischio accettabile.
- [ ] **6. Eseguire Risk Assessment e Risk Treatment**  
  Mappare i pericoli interni ed esterni, redigere il *Risk Assessment Report*, pianificare l'uso dei controlli di mitigazione e formalizzare l'approvazione formale dei rischi residui (*residual risks*).
- [ ] **7. Redigere la Statement of Applicability (SoA)**  
  Documento cardine che censisce tutti i controlli dell'Annex A, dichiarando quali sono applicabili e quali esclusi, con le relative motivazioni e modalità di attuazione.
- [ ] **8. Redigere il Risk Treatment Plan (RTP)**  
  Piano operativo per l'adozione pratica dei controlli: definisce chi fa cosa, entro quale data e con quale budget assegnato.
- [ ] **9. Definire le metriche di misurazione dell'efficacia dei controlli**  
  Stabilire KPI oggettivi per monitorare se i controlli implementati e i processi di sicurezza raggiungono gli obiettivi prefissati.
- [ ] **10. Implementare i controlli di sicurezza (Security Controls)**  
  La fase operativa a maggior impatto: adozione di nuove tecnologie, emissione di procedure/policy e cambiamento delle abitudini operative delle persone.
- [ ] **11. Eseguire programmi di Formazione e Consapevolezza (Awareness)**  
  Seconda causa tipica di fallimento: spiegare al personale le ragioni dei nuovi requisiti e formarlo per prevenire errori e resistenze al cambiamento.
- [ ] **12. Esercire il SGSI nella routine quotidiana (Operate the ISMS)**  
  Trasformare le misure in prassi consolidate. Elemento fondamentale: **le evidenze documentali (records e log)**, indispensabili per dimostrare la conformità agli auditor di certificazione.
- [ ] **13. Monitorare e misurare l'ISMS**  
  Analizzare metriche, volumi di incidenti, anomalie e scostamenti rispetto agli obiettivi target stabiliti.
- [ ] **14. Eseguire l'Audit Interno**  
  Verifica ispettiva interna indipendente volta a far emergere deviazioni e non conformità prima dell'audit ufficiale dell'ente.
- [ ] **15. Convocare il Riesame della Direzione (Management Review)**  
  Incontro formale in cui il Top Management analizza le prestazioni del SGSI, delibera risorse e allinea la sicurezza alla strategia aziendale.
- [ ] **16. Gestire le Azioni Correttive**  
  Risoluzione strutturata delle non conformità tramite individuazione ed eliminazione della causa radice (*root cause analysis*).

---

## 2. Tempi di Implementazione (Fasi Plan & Do)

Durata media stimata per le fasi di progettazione e messa a terra dei controlli, ipotizzando l'ausilio di un consulente o di tool software dedicati:

| Dimensione Azienda | Tempistica Stimata |
|---|---|
| **Fino a 20 dipendenti** | Fino a 3 mesi |
| **Da 20 a 50 dipendenti** | Da 3 a 5 mesi |
| **Da 50 a 200 dipendenti** | Da 5 a 8 mesi |
| **Oltre 200 dipendenti** | Da 8 a 20 mesi |

> **Nota:** La mancanza di sponsorship della Direzione o l'assenza di un project manager competente allungano sensibilmente questi tempi.

---

## 3. Ruoli, Responsabilità ed Effort Richiesto

### Attività che richiedono il coinvolgimento del personale interno
1. **Risk Assessment:** individuazione dei rischi su dati e asset specifici del reparto.
2. **Risk Treatment:** selezione delle opzioni di mitigazione più praticabili.
3. **Revisione di policy e procedure:** allineamento dei documenti alle reali prassi operative.
4. **Approvazione formale:** delibera di obiettivi, budget e rischi residui da parte del Top Management (CEO, CIO, CTO).

### Matrice di Effort per l'Implementazione Iniziale

| Ruolo Coinvolto | < 200 Dipendenti | 200 - 2.000 Dipendenti | > 2.000 Dipendenti |
|---|---|---|---|
| **Project Manager** | ~20% del tempo (1 gg/sett.) *(ruolo accorpato al CISO)* | 50% del tempo | 100% del tempo (Full-time) |
| **Security Officer / CISO** | Accorpato al PM | 50% del tempo | 100% del tempo (Full-time) |
| **Project Team** | Non necessario formalizzarlo | Capi reparto interni | Capi reparto interni |
| **Responsabili di Reparto** | ~7 ore complessive per reparto | ~15 ore complessive per reparto | ~30 ore complessive per reparto |
| **Top Management** | ~5 ore complessive | ~10 ore complessive | ~15 ore complessive |

> **Mantenimento a regime:** L'impegno annuo per mantenere e migliorare il sistema dopo la certificazione richiede mediamente il **25%** dell'effort investito nella fase iniziale di implementazione.

---

## 4. Struttura dei Costi dell'Implementazione

Il costo totale dipende da: organico nello scope, criticità delle informazioni gestite, complessità infrastrutturale e vincoli normativi di settore.

```text
                      STRUTTURA DEI COSTI ISO 27001
                                    │
           ┌────────────────────────┼────────────────────────┐
           ▼                        ▼                        ▼
   ┌───────────────┐        ┌───────────────┐        ┌───────────────┐
   │  Formazione   │        │   Supporto    │        │  Operatività  │
   │ & Letteratura │        │   Esterno     │        │  & Audit      │
   └───────┬───────┘        └───────┬───────┘        └───────┬───────┘
           │                        │                        │
           ├► Norma ISO (~$100)     ├► Consulenza (~$15.000) ├► Tempo uomo dipendenti
           │                        │                        │
           └► Corsi ($250-$1.700)   └► Software SaaS (~$2.000)├► Audit Ente (~$7.500)
                                                             │
                                                             └► Uso sicuro asset IT
```

1. **Letteratura e Formazione:** Acquisto ufficiale della norma (~$100) e corsi per il personale ($250 - $1.700 a persona).
2. **Supporto Esterno:** Consulenza specialistica (~$15.000 per piccole imprese USA) o software dedicato/SaaS (~$2.000/anno).
3. **Tempo Uomo Interno:** Costo associato alle ore lavorative spese dai collaboratori nelle varie fasi.
4. **Tecnologia:** Raramente richiede nuovi acquisti hardware/software; consiste principalmente nell'uso più rigoroso e sicuro degli asset già in dotazione.
5. **Certificazione (Ente Terzo):** Tariffa per l'audit svolto dall'Organismo di Certificazione accreditato (~$7.500 per piccole imprese).

---

## 5. Strategie di Implementazione a Confronto

| Strategia | Vantaggi Principali | Limiti / Prerequisiti |
|---|---|---|
| **1. Completamente autonoma (In-house)** | Spese dirette minime, nessun fornitore esterno a contatto con i dati. | Richiede competenze avanzate già presenti in azienda; alto rischio di ritardi o errori. |
| **2. Fai-da-te assistita da tool dedicati** | Ottimo bilanciamento costi/efficacia, forte crescita delle competenze interne. | Richiede costanza e tempo interno per seguire il percorso guidato dal software. |
| **3. Consulenza integrale esterna** | Massima velocità, minor carico di stesura documentale per il team interno. | Opzione più costosa; rischio di scarso recepimento dei processi dopo l'uscita del consulente. |

---

## 6. I 4 Benefici Chiave da Presentare al Management (Business Case)

Per ottenere l'approvazione del budget, il progetto va proposto in ottica di **ritorno sull'investimento (ROI)**:

1. **Compliance (Conformità Normativa e Contrattuale):**  
   Fornisce un modello strutturato per soddisfare obblighi di legge (es. privacy/GDPR) e requisiti formali di qualifica imposti dai grandi clienti.
2. **Marketing Edge (Vantaggio Competitivo):**  
   Funge da elemento differenziatore (*Unique Selling Point*) per rassicurare il mercato sul trattamento sicuro dei dati rispetto ai concorrenti.
3. **Riduzione dei Costi da Incidenti:**  
   Previene le perdite economiche legate a interruzioni di servizio, furti di dati, sanzioni legali o sabotaggi da parte di dipendenti infedeli.
4. **Ottimizzazione Organizzativa:**  
   Risolve le inefficienze tipiche delle aziende in forte espansione definendo chiaramente chi fa cosa, chi autorizza gli accessi e chi è responsabile degli asset.

---

## 7. Criteri di Selezione del Project Manager e dei Software di Supporto

### Profilo del Project Manager
* Comprensione dei processi di business e solida conoscenza operativa dell'IT.
* Disponibilità di tempo adeguata per seguire il progetto.
* **Autorità gerarchica formale** per guidare e rendere vincolanti i cambiamenti di processo.

### Requisiti di un Software di Supporto all'Implementazione
* Percorso guidato allineato ai requisiti della ISO/IEC 27001:2022.
* Wizard preconfigurati per la redazione della documentazione obbligatoria.
* Modulo di Risk Assessment assistito con cataloghi di asset, minacce, vulnerabilità e controlli.
* Compilazione integrata della Dichiarazione di Applicabilità (SoA) legata al Risk Treatment Plan.
* Dashboard collaborativa per delega compiti, scadenziario, audit interni e riesami periodici.
