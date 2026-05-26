# Integrazione di Microsoft Entra con Microsoft Security Copilot

Microsoft Security Copilot  è una soluzione di sicurezza basata su intelligenza artificiale generativa che aiuta i team IT e di sicurezza a **analizzare, investigare e risolvere più rapidamente incidenti e rischi**.

Security Copilot di usare dati di identità e accesso per fornire **analisi contestuali e azioni guidate sull’intero ambiente IAM**.

---

## Obiettivo dell’integrazione

L’integrazione consente di:
- analizzare rischi legati alle identità
- investigare accessi sospetti
- migliorare la risposta agli incidenti di sicurezza
- automatizzare insight e raccomandazioni
- supportare decisioni basate su Zero Trust

---

## Dati utilizzati da Entra in Security Copilot

Security Copilot può accedere e correlare informazioni da Entra, tra cui:
- utenti e gruppi
- log di accesso
- log di audit
- eventi di provisioning
- criteri di accesso condizionale
- assegnazioni di ruoli
- segnali di rischio

Questi dati vengono trasformati in **insight contestuali e comprensibili in linguaggio naturale**.

---

## Modalità di utilizzo

### 1. Esperienza autonoma (standalone)
Nel portale :contentReference[oaicite:2]{index=2}, gli amministratori possono usare prompt in linguaggio naturale, ad esempio:
- ottenere dettagli di un utente
- analizzare accessi recenti
- estrarre log di sicurezza
- investigare eventi sospetti

Esempi:
- “Mostrami i log di accesso degli ultimi 48 ore”
- “Elenca gli accessi rischiosi dell’utente X”
- “Fornisci dettagli dell’utente e del suo oggetto ID”

---

### 2. Esperienza integrata in Microsoft Entra
Security Copilot è incorporato direttamente nel portale di Entra, ad esempio in:
- :contentReference[oaicite:3]{index=3}
- report utenti a rischio
- dashboard di sicurezza e accesso

Qui Copilot:
- riassume il rischio utente
- spiega il contesto dell’incidente
- suggerisce azioni di remediation

---

## Scenari supportati

### Microsoft Entra ID
- analisi utenti, gruppi e licenze
- controllo log di accesso e audit
- valutazione criteri di accesso condizionale
- analisi metodi di autenticazione e ruoli

---

### Identity Protection
- analisi utenti a rischio
- riepilogo dei rilevamenti
- raccomandazioni di mitigazione
- valutazione del rischio applicazioni

---

### Identity Governance
- analisi access review
- monitoraggio PIM
- gestione lifecycle e entitlement
- verifica assegnazioni accesso

---

### Global Secure Access
- analisi traffico di rete
- log di accesso sicuro
- valutazione connessioni utenti-app

---

## Agenti Microsoft Entra

Gli **agenti Entra** sono componenti AI che automatizzano attività di gestione identità.

Caratteristiche:
- identità agente dedicata
- esecuzione programmata o manuale
- azioni automatiche su policy e rischi
- integrazione con principi Zero Trust

Esempi:
- ottimizzazione accesso condizionale
- analisi rischi utenti e identità
- suggerimenti di remediation automatica

---

## Store e gestione agenti
Gli agenti possono essere:
- distribuiti da Microsoft o partner
- scoperti tramite Security Store
- gestiti dal portale Microsoft Entra

---

## Feedback e miglioramento

Security Copilot utilizza feedback degli amministratori per:
- migliorare precisione delle risposte
- ridurre errori nei suggerimenti
- ottimizzare analisi dei dati di sicurezza

---

## Sintesi

L’integrazione tra Microsoft Entra e Security Copilot consente di:
- trasformare dati di identità in insight intelligenti
- accelerare indagini di sicurezza
- automatizzare analisi di rischio
- rafforzare la postura Zero Trust dell’organizzazione
