# FortiGate: Guida Completa a Logging & Monitoring

---

## 1. Flusso di Analisi e Scopo del Logging

### Ciclo di vita del pacchetto e registrazione
1. **Transito:** Il traffico (in ingresso o in uscita) attraversa il firewall FortiGate.
2. **Ispezione:** FortiOS analizza i pacchetti e applica le Firewall Policy e i profili di sicurezza configurati.
3. **Generazione Log:** Ogni azione eseguita (permetti, blocca, ispeziona) genera un evento registrato sotto forma di messaggio di log.
4. **Archiviazione:**
   - **Locale:** Su memoria volatile (RAM) o disco interno (SSD/HDD locale).
   - **Remota:** Invio a piattaforme dedicate di analisi, reporting e correlazione (es. **FortiAnalyzer**, FortiGate Cloud, SIEM di terze parti).

### Obiettivi Operativi
- **Monitoraggio del traffico:** Visibilità in tempo reale su flussi di rete e applicazioni Layer 7.
- **Troubleshooting:** Diagnostica rapida di disservizi, errori di instradamento o blocchi indesiderati.
- **Definizione di baseline:** Comprensione dei pattern di traffico nominali per individuare anomalie o picchi sospetti.

---

## 2. Tassonomia dei Log

### Macro-Categorie

| Categoria | Descrizione | Sottocategorie Principali |
| :--- | :--- | :--- |
| **Traffic Log** | Informazioni relative ai flussi di rete attraversanti o terminanti sul firewall. | • **Forward:** Traffico instradato attraverso il firewall.<br>• **Local:** Traffico diretto al FortiGate o generato da esso.<br>• **Sniffer:** Pacchetti catturati dal packet sniffer integrato. |
| **Event Log** | Tracciamento di eventi operativi interni e configurazioni di sistema. | • System<br>• User (autenticazione)<br>• VPN (tunnel IPsec/SSL)<br>• Route<br>• Endpoint Control |
| **Security Log** | Eventi generati dai motori di sicurezza UTM/NGFW. | • Web Filter<br>• Antivirus<br>• Application Control<br>• IPS / Intrusion Prevention<br>• DLP (Data Loss Prevention)<br>• Antispam |

---

## 3. Livelli di Severità (Severity Levels)

I log FortiOS sono classificati su una scala numerica standard da **0** (massima gravità) a **7** (diagnostica):

| Livello | Nome | Descrizione |
| :---: | :--- | :--- |
| **0** | **Emergency** | Il sistema è instabile o compromesso; arresto imminente. |
| **1** | **Alert** | È richiesta un'azione correttiva immediata. |
| **2** | **Critical** | Funzionalità o servizi critici del dispositivo compromessi. |
| **3** | **Error** | Si è verificato un errore che impedisce il corretto funzionamento di una funzione. |
| **4** | **Warning** | Rilevata una situazione anomala che potrebbe richiedere attenzione. |
| **5** | **Notification** | Evento informativo sul normale ciclo di funzionamento. |
| **6** | **Information** | Dati e statistiche generiche di sistema. |
| **7** | **Debug** | Informazioni diagnostiche dettagliate a basso livello. |

---

## 4. Anatomia di un Log FortiOS

### Stringa Raw di Esempio
```text
date=2026-04-29 time=07:29:09 eventtime=1754576949412373904 tz="-0700" logid="0316013056" type="utm" subtype="webfilter" eventtype="ftgd_blk" level="warning" vd="root" policyid=1 poluuid="5476b1d8-8be7-51ef-d3c8-3e0539bb2ec0" policytype="policy" sessionid=35060 srcip=10.0.11.50 srcport=46830 srccountry="Reserved" srcintf="port4" srcintfrole="undefined" srcuuid="7bc87d34-7916-51e7-3d5b-71812a61b98e" dstip=151.101.67.5 dstport=80 dstcountry="United States" dstintf="port2" dstintfrole="undefined" dstuuid="7bc87d34-7916-51e7-3d5b-71812a61b98e" proto=6 httpmethod="GET" service="HTTP" hostname="cnn.com" agent="Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:139.0) Gecko/20100101" profile="default" action="blocked" reqtype="direct" url="http://cnn.com/" sentbyte=501 rcvdbyte=0 direction="outgoing" msg="URL belongs to a denied category in policy" ratemethod="domain" cat=36 catdesc="News and Media"
```

### Sezionamento e Analisi dei Campi

#### A. Log Header (Metadati Generali)
- `date=2026-04-29 time=07:29:09`: Data e ora locale dell'evento.
- `eventtime=1754576949412373904`: Timestamp Unix in nanosecondi per correlazioni precise su SIEM.
- `tz="-0700"`: Fuso orario configurato sul firewall (UTC-7).
- `logid="0316013056"`: ID univoco FortiOS per identificare il tipo di messaggio.
- `type="utm"`: Categoria principale (motori di sicurezza avanzata).
- `subtype="webfilter"`: Sottotipo specifico (filtro URL e navigazione web).
- `eventtype="ftgd_blk"`: Codice evento FortiGuard Block (bloccato dal cloud FortiGuard).
- `level="warning"`: Severità 4.
- `vd="root"`: Virtual Domain (VDOM) in cui è avvenuto l'evento.

#### B. Log Body (Dettagli di Connessione e Sessione)
- **Policy & Session:**
  - `policyid=1`: Identificativo della regola di firewall che ha matchato il traffico.
  - `poluuid`: Identificativo UUID globale della policy.
  - `sessionid=35060`: ID della sessione attiva nella state table del firewall.
- **Sorgente (Client):**
  - `srcip=10.0.11.50`: Indirizzo IPv4 privato del computer richiedente.
  - `srcport=46830`: Porta sorgente effimera del client TCP.
  - `srcintf="port4"`: Interfaccia fisica di ingresso su FortiGate.
- **Destinazione (Server Web):**
  - `dstip=151.101.67.5`: Indirizzo IPv4 pubblico del web server remoto.
  - `dstport=80`: Porta di destinazione standard HTTP.
  - `dstcountry="United States"`: Nazione individuata tramite database GeoIP.
  - `dstintf="port2"`: Interfaccia fisica di uscita (WAN).
- **Protocollo e Dati Applicativi:**
  - `proto=6`: Protocollo TCP a livello di trasporto.
  - `httpmethod="GET"`: Metodo HTTP utilizzato.
  - `hostname="cnn.com"` / `url="http://cnn.com/"`: FQDN e risorsa richiesta.
  - `agent="..."`: User-Agent del browser (Firefox su Ubuntu Linux).
  - `direction="outgoing"`: Direzione del flusso (dall'interno verso l'esterno).
  - `sentbyte=501` / `rcvdbyte=0`: Byte inviati dal client prima dell'interruzione; 0 byte ricevuti.
- **Causa del Blocco (Security Context):**
  - `profile="default"`: Profilo Web Filter applicato alla policy.
  - `action="blocked"`: Azione eseguita (connessione abbattuta).
  - `cat=36` / `catdesc="News and Media"`: ID e nome della categoria FortiGuard assegnata al dominio.
  - `msg="URL belongs to a denied category in policy"`: Causa formale del drop.

---

## 5. Memorizzazione Locale su Disco (Disk Logging)

### Funzionamento
- **Abilitato:** I log vengono registrati in un database SQL locale su disco, consentendo consultazioni storiche.
- **Disabilitato:** Visibilità limitata al monitoraggio in tempo reale (ad esempio tramite FortiView).

### Modelli Hardware FortiGate (Regola "0" vs "1")
- **Modelli terminanti con 0** (es. *FortiGate 40F*): Non dispongono di storage interno (SSD/HDD). I log vengono mantenuti temporaneamente in RAM e sovrascritti al riempimento o azzerati al riavvio. Non supportano dischi esterni per l'espansione dei log. È mandatorio l'invio remoto (FortiAnalyzer, FortiGate Cloud o SIEM).
- **Modelli terminanti con 1** (es. *FortiGate 41F*): Integrano un'unità SSD dedicata per il salvataggio persistente dei log.

### Configurazione
- **GUI:** `Log & Report` > `Log Settings` > `Local Logs` > Abilitare **Disk logging**.
- **CLI:**
```fortios
config log disk setting
    set status enable
    set maximum-log-age <valore_in_giorni>
end
```

---

## 6. Integrazione con FortiAnalyzer

### Come Funziona
1. FortiGate invia i log in formato compresso non elaborato (**raw**).
2. FortiAnalyzer riceve i file grezzi, li archivia e li indicizza all'interno di un database SQL dedicato.
3. L'amministratore può eseguire query rapide, reportistiche automatizzate e analisi storiche.

### Modalità Operative di FortiAnalyzer
- **Analyzer Mode (Default):** Raccoglie i log, li indicizza nel database e genera analisi e report.
- **Collector Mode:** Utilizzato in infrastrutture complesse. Il Collector riceve i log raw da molti firewall e li inoltra a un FortiAnalyzer in modalità Analyzer senza sovraccaricare le singole sedi periferiche.
  $$\text{FortiGate (x5)} \longrightarrow \text{FortiAnalyzer (Collector)} \longrightarrow \text{FortiAnalyzer (Analyzer Database)}$$

### Metodi di Registrazione del FortiGate
1. **Inizializzazione da FortiGate:** Si specifica l'IP di FortiAnalyzer; il FortiGate comparirà su FortiAnalyzer tra gli *Unauthorized Devices* in attesa di approvazione.
2. **Inizializzazione da FortiAnalyzer:** Procedura di discovery guidata inserendo IP e credenziali amministrative del firewall.
3. **Pre-registrazione (Zero-Touch):** Inserimento preventivo su FortiAnalyzer di modello e numero di serie (Serial Number). Al primo contatto via Internet, l'apparato viene autorizzato automaticamente.

### Configurazione Passo-Passo

#### Lato FortiGate (GUI o CLI)
- **GUI:** `Security Fabric` > `Fabric Connectors` > `Logging & Analytics` > inserire l'IP di FortiAnalyzer e scegliere la frequenza di upload (*Realtime*, *Every minute*, *Every 5 minutes*).
- **CLI:**
```fortios
config log fortianalyzer setting
    set status enable
    set server "10.0.13.125"
    set serial "FAZ-VMTM24012176"
    set enc-algorithm high-medium
    set upload-option realtime
    set reliable enable
end
```
> **Nota su `set reliable enable`:** Di default FortiGate invia i log su **UDP 514**. Abilitando `reliable`, il canale passa a **TCP** con cifratura **OFTPS (SSL/TLS)**. In FortiAnalyzer comparirà un'icona con un **lucchetto verde** a conferma del canale cifrato e affidabile.

#### Lato FortiAnalyzer
1. Accedere a **Device Manager**.
2. Individuare il firewall nella tabella **Unauthorized Devices**.
3. Selezionare l'apparato e fare clic su **Authorize**.

---

## 7. Gestione dello Spazio e Retention su FortiAnalyzer

Configurabile da: `System Settings` > `Advanced` > `Device Log Settings`.

### Rotazione dei Log (Log Roll)
- **Per dimensione:** Rotazione automatica al raggiungimento della soglia (es. tra 10 e 1000 MB).
- **Pianificata:** Rotazione temporizzata (es. ogni lunedì alle 00:00).

### Backup Esterno Automatico
- **File Transfer Standard:** Invio dei log ruotati verso server esterni tramite FTP, SFTP o SCP.
- **Cloud Storage:** Trasferimento diretto su bucket cloud (es. AWS S3).

### Criteri di Eliminazione Automatica (Retention)
- `Device log files older than`: Giorni massimi di conservazione dei file di log.
- `Reports older than`: Eliminazione report generati.
- `Content archive files older than`: Cancellazione contenuti payload archiviati (DLP, pacchetti).
- `Quarantined files older than`: Pulizia dei file in quarantena oltre la soglia temporale definita.

---

## 8. Ricerca, Filtri e Consultazione Avanzata

### Risoluzione degli Indirizzi IP (Resolve Hostnames)
I log mostrano normalmente gli indirizzi IP numerici; FortiOS permette la risoluzione inversa dei nomi in due modalità:
- **On log creation:** L'IP viene risolto e memorizzato direttamente nella stringa di log al momento dell'evento.
- **When viewed:** La risoluzione DNS viene effettuata solo dinamicamente al momento dell'apertura della pagina nella GUI.

```fortios
# Modalità "On log creation"
config log setting
    set resolve-ip enable
end

# Modalità "When viewed"
config log gui-display
    set resolve-hosts enable
end
```

### Consultazione Rapida da Policy di Firewall (GUI)
1. Navigare in **Policy & Objects** > **Firewall Policy**.
2. Fare clic con il tasto destro sulla regola desiderata.
3. Selezionare **Show matching logs**.
4. La vista filtrata mostra: Data/Ora, Source, Destination, Application Name, Bytes trasferiti ed Esito dell'azione.

### Filtraggio Log da CLI
Per ispezionare i log direttamente da console:
```fortios
# 1. Imposta la categoria di log da esaminare (es. categoria 3 = Event/UTM)
execute log filter category 3

# 2. (Opzionale) Aggiunge filtri specifici (es. livello o interfaccia)
execute log filter field action blocked

# 3. Esegue la visualizzazione a video dei log filtrati
execute log display
```