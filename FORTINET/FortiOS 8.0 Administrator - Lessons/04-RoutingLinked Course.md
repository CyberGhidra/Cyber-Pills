# FortiOS: Architettura di Routing, FIB/RIB e Meccanismi di Inoltro

---

## 1. Modalità Operativa e Tipologie di Traffico

Quando il FortiGate opera nella modalità predefinita **NAT Mode (Routed Mode)**, si comporta come un router di livello 3 a tutti gli effetti, instradando pacchetti IPv4 e IPv6 tra subnet differenti.

FortiOS distingue il traffico in transito ed elaborato in due macro-categorie fondamentali:

| Caratteristica | Firewall Traffic (Traffico di Transito) | Local-Out Traffic (Traffico Originato dal FortiGate) |
| :--- | :--- | :--- |
| **Sorgente** | Generato da host terzi (client LAN, server DMZ, host Internet). | Originato dal processo interno/demone di FortiOS (`localhost`). |
| **Esempi** | Client interno che naviga su un sito web; traffico tra VLAN. | Aggiornamenti FortiGuard, sincronizzazione NTP, invio log verso FortiAnalyzer/Syslog, query DNS. |
| **Ordine di analisi** | **1° Routing Lookup** $\rightarrow$ **2° Firewall Policy**.<br>Se la rotta non esiste, il pacchetto viene scartato immediatamente per *unreachable route* prima ancora di consultare le policy. | Consulta la tabella di routing per stabilire sia l'**interfaccia di uscita**, sia l'**indirizzo IP sorgente locale** da applicare alla connessione (se non forzato da CLI). |

---

## 2. Decisione di Routing e Risoluzione del Next Hop

Durante l'analisi dell'header IP del pacchetto, FortiOS esamina l'indirizzo IP di destinazione per determinare due elementi operativi:

1. **Outgoing Interface (Interfaccia di uscita):** La porta fisica o logica (es. VLAN, tunnel IPsec) attraverso la quale il frame deve essere trasmesso.
2. **Gateway / Next Hop (Salto successivo):** L'entità a cui consegnare il pacchetto:
   * **Host Remoto (Router di transito):** Tipico dell'uscita verso Internet o una WAN geografica. Il FortiGate invia il frame Ethernet al MAC address del router successivo (es. il Default Gateway del provider tramite la rotta `0.0.0.0/0`).
   * **Host Diretto (Destinazione Finale):** La destinazione appartiene a una subnet direttamente connessa a una porta del firewall (**Connected Route**). Il Next Hop coincide con l'host di destinazione: il FortiGate risolve direttamente il suo MAC address tramite richiesta **ARP** e invia il frame senza intermediari.

---

## 3. Architettura Interna: RIB vs. FIB

FortiOS separa nettamente il piano di controllo (*Control Plane*) dal piano di inoltro dati (*Data Plane*):

```text
[ Protocolli Dinamici: OSPF, BGP, RIP ]   [ Rotte Statiche ]   [ Rotte Connesse ]
                   │                              │                    │
                   └──────────────────────┬───────┴────────────────────┘
                                          ▼
                      ┌───────────────────────────────────────┐
                      │    RIB (Routing Information Base)     │  <-- Control Plane
                      │      Gestita dal demone di routing    │      (Tutte le rotte note candidate)
                      └───────────────────┬───────────────────┘
                                          │ Solo le "Best Routes"
                                          ▼
                      ┌───────────────────────────────────────┐
                      │    FIB (Forwarding Information Base)  │  <-- Data Plane
                      │       Kernel Space / NPU ASIC         │      (Interrogata per l'inoltro attivo)
                      └───────────────────────────────────────┘
```

* **RIB (Routing Information Base):** È il database di instradamento del piano di controllo, gestito dal demone software di routing. Raccoglie tutte le rotte note al firewall (connesse, statiche, BGP, OSPF, RIP). Non viene interrogata pacchetto per pacchetto per non appesantire la CPU.
* **FIB (Forwarding Information Base):** È la tabella compilata nel Kernel di FortiOS e programmata a livello hardware nei chip acceleratori **NPU (Network Processing Unit)**. Contiene esclusivamente le *Best Routes* selezionate dalla RIB, integrate da voci di sistema (loopback locali, broadcast, rotte interne di gestione). È la struttura dati consultata attivamente per ogni nuova sessione.

---

## 4. Gerarchia di Selezione delle Rotte (Route Lookup)

Quando un pacchetto deve essere instradato, FortiOS valuta le rotte disponibili applicando una sequenza decisionale rigida:

```text
+--------------------------------------------------------------+
| 1. Longest Prefix Match (LPM)                                |
|    Vince la rotta con subnet mask più specifica (/32 > /24)  |
+------------------------------+-------------------------------+
                               | (Se il prefisso è identico)
                               v
+--------------------------------------------------------------+
| 2. Administrative Distance (AD)                              |
|    Affidabilità della sorgente (valore numerico più basso)    |
+------------------------------+-------------------------------+
                               | (Se la Distance è identica)
                               v
+--------------------------------------------------------------+
| 3. Metric                                                    |
|    Costo del percorso stabilito dal protocollo (più basso)   |
+------------------------------+-------------------------------+
                               | (Se la Metric è identica)
                               v
+--------------------------------------------------------------+
| 4. Priority (Proprietà specifica di FortiOS)                 |
|    Attributo di spareggio manuale (valore numerico più basso)|
+------------------------------+-------------------------------+
                               | (Se TUTTI i parametri coincidono)
                               v
+--------------------------------------------------------------+
| Risultato: ECMP (Equal-Cost Multi-Path)                      |
| Entrambe le rotte entrano in FIB per bilanciare il traffico  |
+--------------------------------------------------------------+
```

### Esempio di Longest Prefix Match
Un client vuole raggiungere l'IP `10.50.1.10`. Nel firewall sono presenti:
* `0.0.0.0/0` (Default Route, lunghezza prefisso: `/0`)
* `10.50.0.0/16` (Prefisso aggregato, lunghezza prefisso: `/16`)
* `10.50.1.0/24` (Subnet specifica, lunghezza prefisso: `/24`)

> **Risultato:** Viene selezionata **`10.50.1.0/24`**, perché 24 bit contigui corrispondono all'IP di destinazione. La regola del Longest Prefix Match ha la precedenza assoluta: ignora Distance e Metric finché la maschera è più stretta.

### Valori di Administrative Distance Predefiniti in FortiOS

L'Administrative Distance misura l'affidabilità della sorgente di instradamento (valore più basso = priorità maggiore):

| Sorgente di Routing | Administrative Distance (Default) | Descrizione |
| :--- | :---: | :--- |
| **Connected** | **0** | Reti direttamente attestate sulle interfacce fisiche o logiche |
| **Static Route (WAN DHCP / PPPoE)** | **1 - 5** | Rotte apprese dinamicamente dal provider |
| **Static Route (Manuale)** | **10** | Rotte configurate manualmente dall'amministratore |
| **eBGP (External BGP)** | **20** | Routing tra Autonomous System distinti (es. AWS DirectConnect) |
| **OSPF** | **110** | Interior Gateway Protocol più diffuso in ambito enterprise |
| **IS-IS** | **115** | Protocollo Link-State comune nei Service Provider |
| **RIP** | **120** | Protocollo Distance-Vector legacy |
| **iBGP (Internal BGP)** | **200** | Routing BGP interno allo stesso Autonomous System |

---

## 5. Ciclo di Vita del Route Lookup: TCP 3-Way Handshake

FortiGate è un firewall a ispezione di stato (*Stateful*). All'avvio di una nuova connessione TCP, la tabella di routing viene consultata esattamente **due volte**:

```text
[ PC Client (LAN) ]                [ FortiGate ]                [ Web Server (WAN) ]
         │                               │                               │
         │ ─── 1. SYN ─────────────────► │                               │
         │                               ├─ [1° ROUTE LOOKUP]            │
         │                               ├─ [POLICY LOOKUP + NAT]        │
         │                               │ ─── 1. SYN ─────────────────► │
         │                               │                               │
         │                               │ ◄── 2. SYN/ACK ────────────── │
         │                               ├─ [2° ROUTE LOOKUP (RPF)]      │
         │                               ├─ [SCRITTURA SESSIONE IN RAM]  │
         │ ◄── 2. SYN/ACK ────────────── │                               │
         │                               │                               │
         │ ─── 3. ACK (Dati...) ───────► │                               │
         │                               ├─ [FAST PATH / ACCELERAZIONE]  │
         │                               │  (Nessun lookup di routing!)  │
         │                               │ ─── 3. ACK (Dati...) ───────► │
```

### Momento 1: Primo Pacchetto SYN (Andata)
* Il client genera un segmento TCP `SYN` verso Google (`142.250.180.206`) che entra nella porta LAN.
* **1° Route Lookup:** FortiOS verifica la FIB per identificare l'interfaccia di uscita (`port1` WAN) e il gateway del provider.
* **Policy Lookup:** Trovata l'interfaccia di uscita, verifica la compatibilità con le regole di sicurezza (Firewall Policy), applica l'eventuale SNAT e invia il pacchetto verso la WAN.

### Momento 2: Risposta SYN/ACK (Ritorno e Controllo RPF)
* Il server remoto risponde con il pacchetto TCP `SYN/ACK` sulla porta `port1`.
* **2° Route Lookup (Reverse Path Forwarding - RPF):** Il firewall non si fida ciecamente della direzione inversa: interroga nuovamente la tabella di instradamento per l'IP sorgente di ritorno (l'IP del client LAN).
* *Finalità:* Verifica anti-spoofing e prevenzione del routing asimmetrico errato. Se la tabella conferma che per raggiungere il client l'interfaccia corretta è la LAN interna, il pacchetto viene validato. In caso di incongruenza, il pacchetto viene scartato per motivi di sicurezza.

### Momento 3: Registrazione nella Session Table e Fast Path Hardware
* Superato il secondo controllo, FortiOS memorizza la connessione nella **Session Table** in RAM:
  * *Esempio record:* `Session ID 1234: Andata -> In: port3, Out: port1 | Ritorno -> In: port1, Out: port3`.
* Dal terzo pacchetto (`ACK`) in poi e per tutti i dati applicativi successivi, **FortiOS non consulta più la tabella di routing**.
* Il traffico viene gestito direttamente a livello hardware dai processori di rete ASIC (**NPU / Fast Path**) a velocità di linea, bypassando la CPU centrale.

---

## 6. Configurazione delle Rotte Statiche

### Parametri di Configurazione (`Network > Static Routes`)

* **Destination:**
  * **Subnet:** Inserimento esplicito di IP e Subnet Mask (es. `0.0.0.0/0.0.0.0` per la Default Route).
  * **Internet Service:** Servizi cloud categorizzati dal database FortiGuard (es. Microsoft 365, AWS) con indirizzi gestiti in modo automatico.
  * **Named Address:** Associa la destinazione a un oggetto indirizzo predefinito.
* **Gateway Address:**
  * **Specify:** Inserimento manuale del Next Hop (es. `100.65.0.254`).
  * **Dynamic:** Assegnazione automatica del gateway tramite DHCP o PPPoE.
* **Interface:** Interfaccia fisica o logica di trasmissione.
* **Administrative Distance:** Metrica di affidabilità (predefinita a `10` per le rotte manuali).

### Utilizzo dei Named Addresses nel Routing

Per rendere un oggetto indirizzo (`Policy & Objects > Addresses`) visibile e selezionabile all'interno del modulo di routing, è necessario abilitare il flag:

```text
Policy & Objects > Addresses > Modifica Oggetto:
[x] Routing configuration
```

> **Perché questo selettore è fondamentale:**
> Nei contesti aziendali possono esistere migliaia di oggetti indirizzo impiegati unicamente per le regole di sicurezza. Questo selettore filtra gli elementi, impedendo che i menu a tendina del modulo di routing vengano saturati da oggetti non pertinenti.

---

## 7. ECMP (Equal-Cost Multi-Path)

L'ECMP consente a FortiGate di sfruttare simultaneamente percorsi multipli verso la stessa subnet di destinazione, distribuendo il carico tra più interfacce o collegamenti ridondati.

### Requisiti Mandatori per l'Attivazione di ECMP
Due o più rotte vengono installate contemporaneamente nella **FIB** per operare in bilanciamento solo se possiedono valori identici per:
1. **Destination Subnet:** Stessa rete e stessa maschera di destinazione.
2. **Administrative Distance:** Stesso valore di distanza amministrativa.
3. **Metric:** Stesso valore di metrica.
4. **Priority:** Stesso valore di priorità FortiOS.

### Modalità di Bilanciamento da CLI (`v4-ecmp-mode`)

```fortios
config system settings
    set v4-ecmp-mode [source-ip-based | weight-based | usage-based | source-dest-ip-based]
end
```

* **`source-ip-based` (Default):** Calcola un hash matematico sull'indirizzo IP del mittente. Tutte le connessioni originate dallo stesso host utilizzeranno sempre la medesima interfaccia di uscita (fondamentale per evitare la disconnessione da portali bancari o servizi web con sessioni vincolate all'IP).
* **`source-dest-ip-based`:** L'hash viene calcolato sulla coppia IP sorgente e IP destinazione, garantendo una migliore distribuzione dei flussi generati dalla medesima macchina.
* **`weight-based`:** Distribuisce il carico proporzionalmente a un peso (*weight*) assegnato a ciascuna rotta (ideale per connessioni asimmetriche, es. 1 Gbps vs 200 Mbps).
* **`usage-based`:** Assegna le nuove sessioni al link con minor utilizzo di banda in quel momento in base a una soglia configurata.

---

### Scenario Pratico ed Esempio CLI

#### Architettura

```text
[ User A: 10.0.3.1 ]
         │
         ├─── (port3: 10.0.3.0/24) ──► [ FortiGate ] ──┬── port1 ──► Gateway: 10.0.1.254 ──┐
         │                                             │                                    ├──► [ Server: 10.0.4.0/24 ]
[ User B: 10.0.3.2 ]                                   └── port2 ──► Gateway: 10.0.2.254 ──┘
```

#### Output della Tabella di Routing (`get router info routing-table all`)

```text
S    10.0.4.0/24 [10/0] via 10.0.1.254, port1, [5/0]
                 [10/0] via 10.0.2.254, port2, [5/0]
```

#### Decodifica dei Parametri:
* **`S`:** Rotta di tipo Statico.
* **`10.0.4.0/24`:** Destinazione identica per entrambe le righe.
* **`[10/0]`:** 
  * `10`: **Administrative Distance** (identica).
  * `0`: **Metric** (identica).
* **`[5/0]` (Costrutto specifico di FortiOS):**
  * `5`: **Priority** (identica: soddisfa la condizione per l'entrata congiunta in FIB).
  * `0`: **Weight** (peso assegnato per l'algoritmo *weight-based*).

#### Flusso del Traffico:
1. Con l'algoritmo predefinito `source-ip-based`, l'hash dell'IP di **User A** (`10.0.3.1`) seleziona `port1`. Tutte le sue sessioni verso la rete server escono dal gateway `10.0.1.254`.
2. L'IP differente di **User B** (`10.0.3.2`) produce un hash che seleziona `port2`, instradando i suoi pacchetti verso il gateway `10.0.2.254`.
3. In caso di guasto o disconnessione su `port1`, FortiOS ritira la rotta dalla FIB e il 100% delle sessioni viene veicolato in modo trasparente su `port2`.
