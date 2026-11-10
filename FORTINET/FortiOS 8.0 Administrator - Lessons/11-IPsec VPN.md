# IPsec VPN 
What Is IPsec? (Concetti Fondamentali e Obiettivi di Sicurezza)
1) Scopo principale (Joins remote hosts and networks together into one private network): 
   IPsec consente di estendere una rete privata sicura attraverso un'infrastruttura pubblica e non fidata (come Internet), collegando tra loro sia singole postazioni remote (scenario Client-to-Site / Remote Access, mostrato dal client con certificato digitale) sia intere     reti o sedi periferiche verso la sede centrale (scenario Site-to-Site, rappresentato dal router/firewall a destra).
2) I tre pilastri di sicurezza (Usually provides):
   Autenticazione (Authentication): garantisce l'identità certa dei peer coinvolti nella comunicazione. Impedisce che endpoint non autorizzati o contraffatti si inseriscano nel tunnel (tramite Pre-Shared Key, certificati digitali o autenticazione utente).
   Integrità dei dati (Data integrity / tamper proofing): garantisce che i pacchetti in transito non siano stati intercettati e alterati o manipolati da terze parti (come l'attaccante rappresentato sulla nuvola Internet). Si basa su funzioni di hash crittografico (es. SHA
   256).
   Riservatezza dei dati (Data confidentiality / encryption): protegge il contenuto informativo del traffico rendendolo illeggibile a chiunque intercetti il flusso di rete tramite cifratura simmetrica robusta (es. AES-128, AES-256).


What Is the IPsec Protocol? (Protocolli e Gestione del NAT)
1) Suite di Protocolli e Scelte Architetturali di FortinetSuite integrata (Multiple protocols that work together):
   IPsec non è un singolo protocollo, ma una suite formata da protocolli di negoziazione e controllo (IKE) e protocolli di protezione del payload (AH, ESP).
2) Authentication Header (AH):
   È definito negli standard RFC e fornisce autenticazione e integrità del pacchetto (compreso l'header IP esterno), ma non esegue crittografia (i dati viaggiano in chiaro).
3) Comportamento su FortiGate:
   FortiOS non supporta né utilizza AH. Tutti i tunnel IPsec su FortiGate impiegano il protocollo ESP

Protocollo IKE
RFC 2409 (IKEv1)
RFC 4306 (IKEv2)

SENZA NAT 
Protocollo IP 17 (UDP) 
Porta di ascolto: UDP 500

CON NAT
Protocollo IP 17 (UDP)   
Inizia su UDP 500 e commuta su UDP 4500 per rekey, quick mode e scambi di configurazione (mode-cfg)



Protocollo ESP
RFC 4303

SENZA NAT 
Protocollo IP 50 nativo
(Non ha numeri di porta TCP/UDP)   

CON NAT
Protocollo IP 17 (UDP)  
Viene incapsulato dentro pacchetti UDP sulla porta 4500 (UDP Encapsulation) per consentire il NAT/PAT


Se la rete o l'ISP blocca la porta UDP 500, FortiOS permette di cambiare la porta IKE da riga di comando
Range configurabile: da 1024 a 65535.

config system settings
    set ike-port <port>
end


How Does IPsec Work?
1) Livello Operativo nello Stack OSI
   IPsec opera a Livello 3 (Network Layer - IP). Questo rappresenta un vantaggio chiave rispetto a protocolli applicativi o di trasporto (come SSL/TLS a Livello 4-7):
   operando a Livello 3, IPsec è completamente trasparente per le applicazioni superiori (TCP, UDP, ICMP, HTTP).
2) Meccanismo di Incapsulamento (Encapsulation)
   I protocolli di trasporto e applicativi vengono "impacchettati" e protetti all'interno della struttura IPsec. Il contenuto varia a seconda della modalità operativa selezionata:
   a) Transport Mode: cifra soltanto il payload di trasporto (il segmento TCP o UDP originale), mantenendo invariato l'header IP di origine. Viene impiegato principalmente per comunicazioni dirette host-to-host.
   b) Tunnel Mode: cifra l'intero pacchetto IP originale (incluso il suo header IP originario con indirizzi sorgente e destinazione privati) e aggiunge un nuovo header IP esterno contenente gli indirizzi pubblici dei gateway VPN. È la modalità predefinita e standard          utilizzata da FortiGate per le connessioni Site-to-Site e Client-to-Site.
3) Negoziazione (Negotiation)
   Prima che i dati possano essere cifrati e scambiati, i due endpoint devono instaurare una serie di accordi preliminari:
   a) Autenticazione: verifica reciproca dell'identità dei partecipanti (tramite chiavi condivise o certificati PKI).
   b) Handshake di negoziazione: scambio sicuro di parametri, selezione delle suite crittografiche (algoritmi di cifratura come AES, funzioni di hash come SHA) e generazione dinamica delle chiavi di sessione (tramite scambio Diffie-Hellman). 


## 1. Schema Strutturale dei Pacchetti

### A. Senza VPN (*No VPN - Traffico in Chiaro*)
```text
┌───────────────────────┬────────────────────────────────────────┐
│  Original IP Header   │       Payload TCP / UDP / SCTP         │
│  (IP Sorgente/Dest.)  │               + Dati                   │
└───────────────────────┴────────────────────────────────────────┘
```

---

### B. Tunnel Mode (*Default FortiOS — Gateway-to-Gateway / Client-to-Site*)
L'intero pacchetto IP originario viene inserito come payload, cifrato e protetto all'interno di un'intestazione IP pubblica completamente nuova.

```text
┌──────────────┬────────────┬─────────────────────┬──────────────────────┬─────────────┬───────────────┐
│    New       │    ESP     │     Original        │     TCP / UDP        │     ESP     │ ESP Integrity │
│  IP Header   │   Header   │    IP Header        │       + Data         │   Trailer   │    (HMAC)     │
└──────────────┴────────────┴─────────────────────┴──────────────────────┴─────────────┴───────────────┘
                            │◄─────────────────────── CIFRATO ────────────────────────►│
               │◄─────────────────────────────── AUTENTICATO ──────────────────────────►│
```

* **New IP Header**: Contiene gli indirizzi IP pubblici dei gateway VPN (es. i due apparati FortiGate).
* **Zona Cifrata**: `[Original IP Header] + [TCP/UDP Payload] + [ESP Trailer]`.
* **Zona Autenticata**: `[ESP Header]` fino a `[ESP Trailer]` (validata dal campo `ESP Integrity`).

---

### C. Transport Mode (*Host-to-Host / Overlay GRE-L2TP*)
Protegge unicamente il segmento di trasporto L4. L'header IP originale non viene modificato né nascosto.

```text
┌─────────────────────┬────────────┬──────────────────────┬─────────────┬───────────────┐
│     Original        │    ESP     │     TCP / UDP        │     ESP     │ ESP Integrity │
│    IP Header        │   Header   │       + Data         │   Trailer   │    (HMAC)     │
└─────────────────────┴────────────┴──────────────────────┴─────────────┴───────────────┘
                                   │◄────── CIFRATO ─────►│
                      │◄─────────────────── AUTENTICATO ────────────────►│
```

* **Original IP Header**: Rimane **in chiaro** all'esterno e non è coperto dall'integrità crittografica di ESP.
* **Zona Cifrata**: Solo il segmento L4 `[TCP/UDP Payload] + [ESP Trailer]`.
* **Zona Autenticata**: `[ESP Header]` fino a `[ESP Trailer]`.

---

## 2. Tabella Sinottica Comparativa

| Parametro | Senza VPN (*No VPN*) | Tunnel Mode | Transport Mode |
| :--- | :--- | :--- | :--- |
| **Nuovo Header IP** | Assente | **Aggiunto** (IP pubblici dei peer) | **Assente** |
| **Header IP Originale** | In chiaro | **Cifrato** (nascosto nel payload ESP) | **In chiaro** (visibile durante il transito) |
| **Integrità Header IP Orig.** | Assente | **Sì** (incluso nel controllo HMAC) | **No** (escluso dall'HMAC ESP) |
| **Overhead sul Pacchetto** | Nullo | Elevato (~40-56 Byte aggiunti) | Minore (~20-36 Byte aggiunti) |
| **Tipologia d'Uso Tipica** | Rete LAN / Internet standard | **Site-to-Site** o **Dialup (FortiClient)** | **Host-to-Host** o **GRE/L2TP over IPsec** |
| **Comportamento FortiOS** | Default su interfacce fisiche | **Default assoluto** su tutti i tunnel | Usato solo su richiesta specifica |

---

## 3. Protocolli e Attraversamento NAT (NAT-T)

Quando il traffico IPsec deve transitare attraverso apparati che effettuano NAT/PAT, il protocollo ESP nativo (**IP 50**) non possiede porte Layer 4 e viene bloccato. Interviene quindi l'incapsulamento **NAT-Traversal (RFC 3948)**.

```text
Senza NAT (Direct ESP):
┌──────────────┬────────────────────────┬─────────────┐
│  IP Header   │ ESP Protocol (IP 50)   │ ESP Payload │
└──────────────┴────────────────────────┴─────────────┘

Con NAT-Traversal (UDP 4500 Encapsulation):
┌──────────────┬────────────────────────┬─────────────┬─────────────┐
│  IP Header   │ UDP (IP 17 - Port 4500)│ ESP Header  │ ESP Payload │
└──────────────┴────────────────────────┴─────────────┴─────────────┘
```

### Mappatura Porte IKE ed ESP

| Flusso | Senza NAT (*Native*) | Con NAT-T (*Encapsulated*) |
| :--- | :--- | :--- |
| **IKE Negotiation (Fase 1 / Fase 2)** | IP proto 17: **UDP 500** | IP proto 17: **UDP 500** $\rightarrow$ passa a **UDP 4500** |
| **ESP Data Traffic (Fase 2 SA)** | **IP proto 50** (nativo) | IP proto 17: **UDP 4500** (*ESP incapsulato in UDP*) |

---

## 4. Configurazione CLI di Riferimento (FortiOS)

### Modifica Modalità di Incapsulamento (Fase 2)
In FortiOS l'incapsulamento si imposta nella sezione Phase 2 del tunnel IPsec:
```text
config vpn ipsec phase2-interface
    edit "VPN_P2"
        set phase1name "VPN_P1"
        set encapsulation {tunnel-mode | transport-mode}
    next
end
```
*(Nota: il valore predefinito è `tunnel-mode`).*

### Personalizzazione Porta IKE di Sistema
```text
config system settings
    set ike-port <1024-65535>
end
```


IKEv1 e IKEv2
IKE si occupa di autenticare i nodi, concordare gli algoritmi crittografici (cifratura e hashing) e calcolare dinamicamente le chiavi private di sessione (tramite l'algoritmo Diffie-Hellman) senza mai trasmetterle in chiaro sulla rete.
Il processo IKE è sempre suddiviso in due step logici distinti: Phase 1 e Phase 2.
KEv1 (legacy): lo standard storico (RFC 2409), più complesso, rigido e con un numero elevato di pacchetti scambiati.   
IKEv2 (new, simpler operation): lo standard moderno (RFC 4306 / RFC 7296), riprogettato per essere più veloce, snello, affidabile e con supporto nativo per la mobilità e il NAT.  

| Feature | IKEv1 | IKEv2 |
| :--- | :--- | :--- |
| **Exchange modes** | • **Main**: 9 messaggi totali (6 per Phase 1, 3 per Phase 2)<br>• **Aggressive**: 6 messaggi totali (3 per Phase 1, 3 per Phase 2) | • Procedura di scambio unica<br>• 4 messaggi totali (una Child SA inclusa) |
| **Authentication methods** | **Simmetrico:**<br>• Pre-shared key (PSK)<br>• Certificate signature<br>• Extended authentication (XAuth) | **Asimmetrico:**<br>• PSK<br>• Certificate signature<br>• EAP (pass-through — no client support) |
| **NAT-T** | Supportato come estensione | Supporto nativo |
| **Reliability** | Non affidabile (i messaggi non prevedono ACK) | Affidabile (i messaggi prevedono ACK) |
| **Dialup phase 1 matching by ID** | • Peer ID + aggressive mode + PSK<br>• Peer ID + main mode + certificate signature | • Peer ID<br>• Network ID |
| **Traffic selector narrowing** | Non supportato | Supportato |



# FortiGate IPsec — Topologie VPN Site-to-Site

---

## 1. Topologia "Simple" (Punto-Punto Diretto)

Rappresenta l'architettura base per connettere tra loro due sedi distinte attraverso Internet (tipicamente una singola filiale verso la sede centrale).

```
┌─────────────────────┐                                ┌─────────────────────┐
│    Branch Office    │ <════════ (IPsec Tunnel) ════> │  Headquarters (HQ)  │
│     (FortiGate)     │           su Internet          │     (FortiGate)     │
└─────────────────────┘                                └─────────────────────┘
```

* **Numero di Tunnel:** 1 singolo tunnel logico.
* **Caratteristiche:**
  * Configurazione elementare su entrambi gli apparati (1 Phase 1, 1 Phase 2).
  * Scalabilità assente: concepita esclusivamente per connessioni dedicate o scenari con due soli siti.

---

## 2. Topologia "Hub-and-Spoke" (A Stella)

Architettura centrale gerarchica. Tutti i siti remoti (*Spokes*) stabiliscono un tunnel esclusivamente verso il firewall centrale (*Hub*).

```
                ┌─────────────────────┐
                │    Branch Office    │ (Spoke 1)
                │     (FortiGate)     │
                └──────────┬──────────┘
                           │
                           │ (Tunnel 1)
                           ▼
┌─────────────────┐  (Tunnel 4)  ┌─────────────────────┐  (Tunnel 2)  ┌─────────────────┐
│  Branch Office  │ <══════════> │  Headquarters (HQ)  │ <══════════> │  Branch Office  │
│   (FortiGate)   │              │     (Central Hub)   │              │   (FortiGate)   │
└─────────────────┘              └──────────┬──────────┘              └─────────────────┘
    (Spoke 4)                               │                             (Spoke 2)
                                            │ (Tunnel 3)
                                            ▼
                                 ┌─────────────────────┐
                                 │    Branch Office    │ (Spoke 3)
                                 │     (FortiGate)     │
                                 └─────────────────────┘
```

* **Numero di Tunnel:** Lineare $\rightarrow$ $N$ tunnel (con $N$ sedi periferiche).
* **Comunicazione Spoke-to-Spoke:**
  * Non ci sono collegamenti diretti tra le filiali.
  * Per comunicare tra loro, i pacchetti devono attraversare l'Hub centrale, che esegue: decifratura $\rightarrow$ verifica delle firewall policy di transito $\rightarrow$ ri-cifratura verso lo Spoke di destinazione (*Tromboning / Hairpinning*).
* **Vantaggi:**
  * **Configurazione semplice sui periferici (*Easy configuration*):** Ogni filiale necessita solo della configurazione verso l'Hub.
  * **Pochi tunnel complessivi (*Few tunnels*):** Facile gestione dell'inventario VPN.
  * **Bassi requisiti hardware per gli Spoke (*Low system requirements on average*):** Le filiali possono montare modelli FortiGate entry-level.
  * **Elevata scalabilità (*Scalable*):** Ottimale per reti con decine o centinaia di sedi.
* **Svantaggi:**
  * **Elevato consumo di banda al centro (*High central bandwidth*):** L'Hub deve sostenere tutto il traffico inter-sede.
  * **Single Point of Failure (*Not fault tolerant*):** Se l'Hub si spegne o perde connettività, l'intera rete si isola.

---

## 3. Topologia "Full Mesh" (Magliatura Completa)

Ogni singolo nodo della rete instaura un tunnel VPN diretto con ciascuno degli altri siti.

```
                  ┌───────────────┐
                  │   Site / HQ   │
                  └───────┬───────┘
                      ▲   │   ▲
             ┌────────┼───┼───┼────────┐
             │        │   │   │        │
             ▼        │   │   │        ▼
      ┌─────────────┐ │   │   │ ┌─────────────┐
      │   Site 2    │◄┼───┼───┼►│   Site 3    │
      └──────┬──────┘ │   │   │ └──────┬──────┘
             ▲        │   │   │        ▲
             │        ▼   │   ▼        │
             │   ┌────────┴───────┐    │
             └──►│     Site 4     │◄───┘
                 └────────────────┘
```

* **Formula dei Tunnel:** Esponenziale $\rightarrow$ $\frac{N(N - 1)}{2}$
  * 4 Siti = 6 tunnel
  * 5 Siti = 10 tunnel
  * 10 Siti = 45 tunnel
* **Caratteristiche e Vantaggi:**
  * **Comunicazione diretta (*Direct communication between all sites*):** Latenza minima assoluta; nessun intermediario.
  * **Massima tolleranza ai guasti (*Fault tolerant*):** La perdita di un nodo o di un link non interrompe la comunicazione tra i restanti.
  * **Bassa concentrazione di banda centrale (*Low bandwidth*):** Il traffico è distribuito in modo uniforme.
* **Limiti:**
  * **Complessità elevata (*Complex configuration*):** Aggiungere un nodo comporta riconfigurare tutti i firewall della rete.
  * **Requisiti hardware pesanti (*High system requirements*):** Tutti i firewall devono reggere molteplici associazioni SA contemporanee.
  * **Difficile da scalare (*Difficult to scale*):** Inapplicabile su larga scala senza tecnologie dinamiche automatiche (es. Fortinet ADVPN).

---

## 4. Topologia "Partial Mesh" (Magliatura Parziale)

Approccio ibrido e pragmatico: unisce la ridondanza della magliatura completa alla semplicità dell'Hub-and-Spoke.

```
       ┌──────────┐                     ┌──────────┐
       │   HQ 1   │ <═════════════════> │   HQ 2   │   (Data Centers / Core Sites:
       │  (Core)  │        (Mesh)       │  (Core)  │    Completamente Magliati)
       └────┬─────┘                     └─────┬────┘
            │                                 │
   (Tunnel) │                        (Tunnel) │
            ▼                                 ▼
     ┌──────────────┐                  ┌──────────────┐
     │ Branch Spoke │                  │ Branch Spoke │ (Filiali: Connesse solo
     └──────────────┘                  └──────────────┘  a uno o due nodi core)
```

* **Caratteristiche:**
  * I siti critici (es. Sede Centrale, Data Center primario, Cloud/DR) sono magliati direttamente tra loro.
  * Le sedi periferiche secondarie si collegano solo a uno o due Hub per ridondanza.
* **Metriche:**
  * **Configurazione e numero di tunnel medi (*Moderate configuration / Medium number of tunnels*).**
  * **Tolleranza ai guasti parziale (*Some fault tolerance*):** Garantita sui nodi nevralgici.
  * **Comunicazione diretta selettiva (*Direct communication between some sites*):** Solo dove serve realmente.

---

## 5. Matrice Comparativa Ufficiale Fortinet

| Parametro / Metrica | Hub-and-Spoke | Partial Mesh | Full Mesh |
| :--- | :--- | :--- | :--- |
| **Complessità di configurazione (*Configuration*)** | **Easy** (Bassa) | **Moderate** (Media) | **Complex** (Molto elevata) |
| **Numero di tunnel (*Number of tunnels*)** | **Few** (Pochi) | **Medium** (Medio) | **Many** (Elevatissimo, $\frac{N(N-1)}{2}$) |
| **Banda richiesta al centro (*Central bandwidth*)** | **High** (Elevata) | **Medium in hub sites** (Ripartita) | **Low bandwidth** (Distribuita) |
| **Tolleranza ai guasti (*Fault tolerance*)** | **Not fault tolerant** (SPOF al centro) | **Some fault tolerance** (Discreta) | **Fault tolerant** (Massima) |
| **Requisiti hardware apparati (*System requirements*)** | **Low** (Spoke) / **High** (Centro) | **Medium** (Medi su tutti i nodi) | **High** (Elevati su tutti i nodi) |
| **Scalabilità (*Scalable*)** | **Scalable** (Ottima ed espandibile) | **Somewhat scalable** (Discreta) | **Difficult to scale** (Scarsa) |
| **Comunicazione diretta (*Direct communication*)** | **No direct communication** (Solo via Hub) | **Between some sites** (Solo nodi scelti) | **Between all sites** (Tra tutti i nodi) |


# Fortinet FortiOS: Configurazione VPN IPsec tramite Wizard

---

## 1. Panoramica del VPN Wizard

Il **VPN Wizard** è la procedura guidata di FortiOS concepita per semplificare, accelerare e standardizzare la configurazione dei tunnel IPsec, riducendo a zero gli errori umani di disallineamento dei parametri di Fase 1, Fase 2, routing statico o firewall policy.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                               FORTIOS IPSEC VPN WIZARD                                 │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ 1. Selezione Modello (Template)                                                        │
│    • Site-to-Site (FortiGate-to-FortiGate, Cisco, altri vendor)                        │
│    • Hub and Spoke with ADVPN (Stella con rami dinamici spoke-to-spoke)                │
│    • Remote Access (Client-to-Gateway per client FortiClient)                          │
│    • FortiClient Secure Internet Access (SIA)                                          │
└──────────────────────────────────────────┬─────────────────────────────────────────────┘
                                           │
                                           ▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ 2. Configurazione Parametri Guidata                                                    │
│    • Endpoint remoti (IP pubblico o Dynamic DNS)                                       │
│    • Autenticazione (Pre-Shared Key o Certificato PKI)                                 │
│    • Interfacce locali e Subnet (Local / Remote networks)                              │
│    • Parametri Client (Pool IP, Split Tunneling, EAP / User Groups per Remote Access)  │
└──────────────────────────────────────────┬─────────────────────────────────────────────┘
                                           │
                                           ▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ 3. Schermata di Riepilogo (Review)                                                     │
│    • Mostra l'elenco esatto di TUTTI gli oggetti che FortiOS genererà in automatico    │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Modelli di Distribuzione (Template)

Nella prima schermata (`VPN > VPN Wizard`), dopo aver assegnato il nome al tunnel (`Tunnel name`), si seleziona il template appropriato:

* **Site to Site:**
  * Crea un tunnel punto-punto permanente tra due sedi fisse.
  * Consente alle filiali (*Branch Offices*) di accedere alla sede principale (*HQ*) o di collegare uffici remoti tra loro.
  * Supporta endpoint FortiGate o gateway IPsec di terze parti (Cisco, Check Point, pfSense, ecc.).
* **Hub and Spoke with ADVPN:**
  * Modello per reti a stella con un firewall centrale (*Hub*) e molteplici filiali (*Spokes*).
  * Grazie ad **ADVPN (Auto Discovery VPN)**, gli spoke possono negoziare dinamicamente scorciatoie punto-punto dirette (*on-demand*), evitando che il traffico inter-filiale continui a sovraccaricare la banda dell'Hub centrale.
* **Remote Access:**
  * Rivolto a utenti in mobilità, collaboratori esterni o telelavoratori (*smart working*) che accedono alla rete aziendale tramite **FortiClient** (Windows, macOS, Linux, iOS, Android).
* **FortiClient Secure Internet Access (SIA):**
  * Estensione del Remote Access: incanala l'intero traffico Internet dei client nel tunnel verso il FortiGate.
  * Consente al firewall centrale di ispezionare e filtrare la navigazione degli utenti remoti (Antivirus, Web Filter, IPS) prima di rilasciarla su Internet.

---

## 3. Oggetti Generati Automaticamente dal Wizard (`Review`)

Nella schermata finale di riepilogo (**Review**), FortiOS visualizza tutti gli elementi generati contestualmente per rendere immediatamente operativo il tunnel:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        SUMMARY OF OBJECTS CREATED BY THE WIZARD                        │
├──────────────────────┬─────────────────────────────────────────────────────────────────┤
│ Addresses & Groups   │ • Local address group (es. ToRemoteBackup_local)                │
│                      │ • Subnet locali (ToRemoteBackup_local_subr)                     │
│                      │ • Remote address group (es. ToRemoteBackup_remote)               │
│                      │ • Subnet remote (ToRemoteBackup_remote_su)                      │
├──────────────────────┼─────────────────────────────────────────────────────────────────┤
│ Interfaces           │ • VPN IPsec Phase 1 Interface (interfaccia logica virtuale)     │
│                      │ • VPN IPsec Phase 2 Interface                                   │
│                      │ • VPN Zone (raggruppa le interfacce correlate)                  │
├──────────────────────┼─────────────────────────────────────────────────────────────────┤
│ Policies             │ • Local to remote policy (consente il traffico in uscita)       │
│                      │ • Remote to local policy (consente il traffico in entrata)      │
├──────────────────────┼─────────────────────────────────────────────────────────────────┤
│ Routes               │ • Destination route (rotta statica verso la subnet remota)      │
│                      │ • Blackhole route (scarta il traffico se il tunnel cade)        │
└──────────────────────┴─────────────────────────────────────────────────────────────────┘
```

### Perché viene creata una Blackhole Route?
* Se il tunnel IPsec va giù (*down*), la rotta statica primaria verso l'interfaccia VPN viene rimossa dalla tabella di inoltro (FIB).
* Senza una rotta blackhole, il traffico diretto agli IP privati della sede remota verrebbe instradato verso la Default Route (`0.0.0.0/0`) e inviato in chiaro su Internet attraverso il provider.
* La **Blackhole Route** (configurata con distanza amministrativa più alta, tipicamente `254`) intercetta questi pacchetti e li scarta silenziosamente, prevenendo la fuga di dati privati non protetti.

---

## 4. Configurazione di una VPN Remote Access (FortiClient)

Nel modello **Remote Access**, il wizard predispone i parametri per i client dialup:

```
[ VPN Wizard: Remote Access ]
       │
       ├─► 1. Remote Endpoint
       │      • Client type: FortiClient (o Native / iOS / Android)
       │      • Authentication: Pre-shared key o Certificate signature
       │      • IKE Version: Version 2 (consigliata) o Version 1
       │      • User authentication method: Gruppi di utenti locali o remoti (LDAP / RADIUS)
       │
       ├─► 2. Client Settings & IP Assignment
       │      • Addressing mode: Manual o IPAM
       │      • IP Range da assegnare ai client (es. 192.168.0.1 - 192.168.0.50)
       │      • Subnet mask e DNS Server per gli endpoint remoti
       │      • Split Tunneling: 
       │          - Enabled: invia al tunnel solo il traffico per la LAN aziendale
       │          - Disabled: invia tutto il traffico (inclusa la navigazione web generale)
       │      • Impostazioni FortiClient: Save password, Auto Connect, Always Up (Keep Alive)
       │
       ├─► 3. Local Network & Interfaces
       │      • Incoming interface: porta WAN su cui FortiGate resta in ascolto (es. port2)
       │      • Local interface: interfaccia LAN aziendale da raggiungere (es. port4)
       │      • Local address: subnet o gruppo di risorse protette (es. HQ_SUBNET)
       │
       └─► 4. Review & Deploy
              • Generazione istantanea di Phase 1 dialup, Phase 2, pool IP, address group,
                zone e relative Firewall Policy di accesso remoto.
```

---

## 5. Tabella di Confronto: Wizard vs. Custom (Manuale)

| Caratteristica | VPN IPsec Wizard | Configurazione Manuale (Custom VPN) |
| :--- | :--- | :--- |
| **Tempo di deployment** | Estremamente rapido (1–2 minuti) | Più articolato (richiede passaggi separati nei vari menu) |
| **Creazione oggetti** | Crea automaticamente indirizzi, zone, rotte, Phase 1/2 e policy | Richiede la creazione manuale di ciascun singolo oggetto e regola |
| **Rischio di errori** | Minimo (applica corrispondenze e best practice standard) | Più alto (possibili disallineamenti su DH group, phase 2 selectors o NAT) |
| **Personalizzazione parametri** | Timer e algoritmi preconfigurati su standard Fortinet | Controllo totale e granulare su ogni parametro (DPD, PFS, lifetime, porte IKE) |
| **Casi d'uso ideali** | Connessioni standard Site-to-Site e FortiClient Remote Access | Tunnel verso apparati terzi con policy o cifrari crittografici non convenzionali |

*(Nota: tutti gli oggetti creati tramite il Wizard possono essere in qualunque momento modificati o convertiti in configurazioni personalizzate sia dalla Web GUI sia da riga di comando CLI).*



# FortiGate IPsec — Guida Completa alla Phase 1 (IKE SA)

---

## 1. Architettura e Ciclo di Vita di Phase 1

L'obiettivo esclusivo della **Phase 1** è negoziare e attivare una singola associazione di sicurezza **bidirezionale**, denominata **IKE SA** (*Internet Key Exchange Security Association*), dedicata esclusivamente a proteggere il traffico di controllo e le successive negoziazioni di Phase 2.

```
+-----------------------------------------------------------------------------------------+
|                                  FASE 1: CICLO IN 3 PASSI                               |
+-----------------------------------------------------------------------------------------+
|  1. Peer Authentication                                                                 |
|     • Pre-Shared Key (PSK) o Digital Signature (Certificati X.509 PKI)                 |
|     • EAP (supporto utente nativo in IKEv2, sostituisce XAuth di IKEv1)                 |
+--------------------------------------------┬--------------------------------------------+
                                             │
                                             ▼
|  2. Diffie-Hellman (DH) Exchange                                                        |
|     • Calcolo congiunto della chiave segreta condivisa (Shared Secret)                  |
|     • Nessuna chiave privata o segreta viene mai trasmessa in chiaro sulla rete         |
+--------------------------------------------┬--------------------------------------------+
                                             │
                                             ▼
|  3. IKE SA Establishment (Canale di Controllo Cifrato)                                  |
|     • IKEv1: 6 messaggi (Main Mode) oppure 3 messaggi (Aggressive Mode)                 |
|     • IKEv2: 2 soli scambi (4 messaggi totali):                                         |
|       - IKE_SA_INIT: accordo su cifrari e scambio DH nonces                             |
|       - IKE_AUTH: autenticazione peer, rilascio IKE SA e prima Child SA (Phase 2)       |
+-----------------------------------------------------------------------------------------+
```

---

## 2. Parametri Network & Local Gateway

All'interno della configurazione di Phase 1 (`VPN > IPsec Tunnels`), la sezione **Network** definisce il binding locale e l'indirizzamento remoto.

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                               PHASE 1 — NETWORK SETTINGS                                │
├────────────────────────┬────────────────────────────────────────────────────────────────┤
│ IP Version             │ Seleziona lo stack di trasporto tra IPv4 o IPv6.               │
├────────────────────────┼────────────────────────────────────────────────────────────────┤
│ Remote Gateway         │ • Static IP address: indirizzo IP pubblico fisso noto.         │
│                        │ • Dialup user: IP del client sconosciuto a priori.             │
│                        │ • Dynamic DNS: risoluzione del peer tramite FQDN.              │
├────────────────────────┼────────────────────────────────────────────────────────────────┤
│ IP Address / Interface │ • IP Address: IP pubblico del gateway remoto.                  │
│                        │ • Interface: interfaccia WAN da cui originare/ascoltare IKE.   │
├────────────────────────┼────────────────────────────────────────────────────────────────┤
│ Local Gateway          │ • Disabled (Default): usa il Primary IP configurato sulla WAN. │
│                        │ • Enabled: consente di selezionare un Secondary IP o           │
│                        │   dichiarare un IP sorgente esplicito (Specify).               │
└────────────────────────┴────────────────────────────────────────────────────────────────┘
```

### Advanced Network Settings
* **Add route:** Se impostato su **Enable**, FortiOS installa automaticamente nella tabella di routing del kernel (FIB) la rotta verso la subnet remota negoziata in Phase 2 appena il tunnel si attiva. Se impostato su **Disable**, l'amministratore deve creare manualmente la rotta statica.
* **Auto discovery sender / receiver / forwarder:** Abilita i componenti per la tecnologia **ADVPN (Auto Discovery VPN)**, consentendo agli spoke periferici di negoziare scorciatoie dirette dinamiche (spoke-to-spoke) bypassando l'Hub.
* **Exchange interface IP:** Permette lo scambio degli indirizzi IP configurati sulle interfacce tunnel, requisito per l'attivazione di protocolli di routing dinamico point-to-point (OSPF/BGP su IPsec).
* **Device creation:** Genera interfacce logiche dinamiche per ogni singolo client dialup collegato.
* **Aggregate member:** Permette di aggregare il tunnel all'interno di un'interfaccia virtuale ridondata/bilanciata (IPsec Aggregate).

---

## 3. Matrice Comparativa dei Ruoli: Remote Gateway

Il valore assegnato a **Remote Gateway** determina se il FortiGate può avviare autonomamente la negoziazione (Initiator) o se deve attendere passivamente il client (Responder).

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                          MODALITÀ DI REMOTE GATEWAY IN PHASE 1                          │
├─────────────────────┬─────────────────────────────────┬─────────────────────────────────┤
│ Tipologia Gateway   │ Ruolo Operativo IKE             │ Scenario Architetturale Tipico  │
├─────────────────────┼─────────────────────────────────┼─────────────────────────────────┤
│ 1. Dialup User      │ SOLO Responder                  │ • Client FortiClient (Remote)   │
│                     │ (Il FortiGate non conosce l'IP  │ • Sedi branch con IP dinamico   │
│                     │  del peer; non può iniziare)    │   privo di Dynamic DNS          │
├─────────────────────┼─────────────────────────────────┼─────────────────────────────────┤
│ 2. Static IP        │ Initiator o Responder           │ • Connessioni Site-to-Site con  │
│                     │ (Avvio bidirezionale istantaneo │   IP pubblici statici dedicati  │
│                     │  appena c'è traffico)           │   su entrambi i firewall        │
├─────────────────────┼─────────────────────────────────┼─────────────────────────────────┤
│ 3. Dynamic DNS      │ Initiator o Responder           │ • Connessioni Site-to-Site ove  │
│    (FQDN)           │ (Risoluzione periodica FQDN     │   uno o entrambi i siti usano   │
│                     │  tramite query DNS di sistema)  │   servizi DDNS (es. no-ip)      │
└─────────────────────┴─────────────────────────────────┴─────────────────────────────────┘
```

---

## 4. IKE Mode Config (Assegnazione Parametri Virtuali)

Disponibile esclusivamente quando il campo *Remote Gateway* è configurato come **Dialup user**. Funziona in modo analogo a un server DHCP interno al protocollo IKE.

```
       [ FortiGate Gateway ]                            [ FortiClient / Remote Peer ]
                 │                                                    │
                 │ ◄═══════════ Negoziazione IKE Phase 1 ═══════════► │
                 │                                                    │
                 │ ─── IKE Mode Config Payload (Push/Reply) ────────► │
                 │     • Virtual IP: 10.200.200.5                     │
                 │     • Netmask: 255.255.255.255                     │
                 │     • DNS Primario: 10.0.1.254                     │
                 │     • Split Tunneling Subnets: 10.0.0.0/16         │
                 │                                                    │
```

* **Assign client IP:** Definisce se allocare gli indirizzi da un **Range** locale, tramite server **DHCP** di rete o da un **Address Group**.
* **DNS:** Assegna ai client remoti i server DNS interni per risolvere i nomi di dominio dell'intranet aziendale (`Specify` o `Use System DNS`).
* **IPv4 Split Tunnel:**
  * **Abilitato:** Invia attraverso il tunnel cifrato unicamente i pacchetti diretti alle subnet aziendali dichiarate; la navigazione Internet generale dell'utente esce direttamente dal router di casa.
  * **Disabilitato:** Forza tutto il traffico del client (inclusa la navigazione web) all'interno del tunnel verso il FortiGate.
* **Vincolo:** Deve essere supportato e abilitato su entrambi gli endpoint.

---

## 5. NAT Traversal (NAT-T) & Keepalive

Risolve l'incompatibilità intrinseca tra il protocollo **ESP (IP protocol 50)** e gli apparati intermedi che eseguono NAT/PAT (privi di porte Layer 4).

```
Senza NAT (Traffico Diretto):
┌───────────────────────────┬───────────────────────────┬─────────────────────────────────┐
│   IP Header (IP Proto 50) │        ESP Header         │        ESP Cifrato + HMAC       │
└───────────────────────────┴───────────────────────────┴─────────────────────────────────┘

Con NAT-Traversal (Incapsulamento UDP 4500):
┌───────────────────────────┬───────────────────────────┬──────────────┬──────────────────┐
│   IP Header (IP Proto 17) │  UDP Port 4500 (NAT-T)    │  ESP Header  │ Payload Cifrato  │
└───────────────────────────┴───────────────────────────┴──────────────┴──────────────────┘
```

* **Opzione `Enable`:** Durante l'avvio della fase IKE (UDP 500), il firewall invia specifici payload di rilevamento. Se riscontra NAT lungo la tratta, commuta automaticamente su **UDP 4500 sia per IKE che per ESP**.
* **Opzione `Forced`:** Salta il probing e obbliga l'incapsulamento su porta UDP 4500 a prescindere dalla presenza di NAT.
* **Keepalive Frequency:** Intervallo in secondi (default `10`) per l'invio di frame fittizi leggeri allo scopo di impedire la chiusura della porta nella translation table dei router NAT intermedi durante i periodi di inattività.

---

## 6. Dead Peer Detection (DPD)

Meccanismo di polling per verificare lo stato di salute del peer remoto e abbattere rapidamente un tunnel orfano.

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                             MODALITÀ OPERATIVE DEAD PEER DETECTION                      │
├─────────────────────┬───────────────────────────────────────────────────────────────────┤
│ On Demand (Default) │ Invia probe DPD SOLO se c'è traffico in uscita da trasmettere ma  │
│                     │ non si riceve traffico di ritorno dal peer (risparmia CPU/banda). │
├─────────────────────┼───────────────────────────────────────────────────────────────────┤
│ On Idle             │ Invia probe a intervalli regolari ogni volta che non transita     │
│                     │ alcun pacchetto all'interno del tunnel.                           │
├─────────────────────┼───────────────────────────────────────────────────────────────────┤
│ Disabled            │ Non invia probe attivi; si limita a rispondere ai check del peer. │
└─────────────────────┴───────────────────────────────────────────────────────────────────┘
```

* **DPD Retry Count:** Numero di fallimenti consecutivi ammessi (default `3`).
* **DPD Retry Interval:** Secondi di attesa tra un tentativo e il successivo (default `20`).
* **Esito Failover:** Superata la soglia di mancata risposta, il FortiGate abbatte l'IKE SA, cancella le IPsec SA associate e rimuove la rotta dalla FIB, permettendo l'attivazione istantanea di percorsi ridondati.

---

## 7. Authentication & Phase 1 Proposal

### Metodi di Autenticazione (`Method`)
1. **Pre-shared Key (PSK):** Password alfanumerica condivisa che deve corrispondere esattamente su entrambi i lati.
2. **Signature:** Autenticazione basata su certificati digitali X.509 firmati da un'autorità di certificazione (CA). Supporta RFC 7427 ed estensioni Post-Quantum (ML-DSA / SLH-DSA).

### Parametri Crittografici della Proposal

| Funzione | Parametro | Opzioni Selezionabili in FortiOS | Note di Best Practice |
| :--- | :--- | :--- | :--- |
| **Cifratura** | Encryption | DES, 3DES, AES128, AES192, AES256, AES128-GCM, AES256-GCM, CHACHA20POLY1305 | DES/3DES obsoleti. Usare AES256 o suite AEAD (AES-GCM). |
| **Integrità** | Authentication | MD5, SHA1, SHA256, SHA384, SHA512, PRFSHA256, PRFSHA384 | MD5/SHA1 vulnerabili a collisioni. Standard: SHA256+. |
| **Scambio Chiavi** | Diffie-Hellman Groups | Gruppi 1, 2, 5 (legacy), 14, 16, 18, 19, 20, 21, 27, 28, 29, 30, 31, 32 | Evitare gruppi < 14. Raccomandati: 14 (2048-bit), 19/20/21 (Curve Ellittiche). |
| **Validità Chiave**| Key Lifetime | Valore in secondi (default: `86400` s = 24 ore) | Al termine, IKE avvia automaticamente il rekeying. |
| **Identificativo** | Local ID | Stringa alfanumerica facoltativa | Utilizzata per il matching del Peer ID in connessioni dialup e con terze parti. |

---

## 8. Sintassi di Riferimento CLI (FortiOS)

Configurazione manuale completa di un'interfaccia di Phase 1 da terminale:

```fortios
config vpn ipsec phase1-interface
    edit "VPN_HQ"
        set interface "port2"
        set peertype any
        set net-device disable
        set proposal aes256-sha256 aes128-sha256
        set dhgrp 14 20
        set remote-gw 10.200.3.1
        set psksecret "ChiaveSegretaMoltoComplessa!2026"
        set dpd-mode on-demand
        set dpd-retrycount 3
        set dpd-retryinterval 20
        set npu-offload enable
    next
end
```


# FortiGate IPsec — Guida Completa alla Phase 2 (IPsec SA & Hardware Offloading)


## 1. Architettura e Funzionamento di Phase 2

Mentre la Phase 1 negozia un canale di controllo bidirezionale (IKE SA), la **Phase 2** è dedicata alla protezione del traffico dati vero e proprio mediante il protocollo **ESP (Encapsulating Security Payload - IP 50)**.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                                 PHASE 1 (IKE SA Tunnel)                                │
│                                (Controllo / Bidirezionale)                             │
│  ┌──────────────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                                  │  │
│  │  ════════════► Outbound IPsec SA (Cifra il traffico uscente verso il peer) ════► │  │
│  │                                                                                  │  │
│  │  ◄════════════ Inbound IPsec SA (Decifra il traffico entrante dal peer) ◄══════  │  │
│  │                                                                                  │  │
│  └──────────────────────────────────────────────────────────────────────────────────┘  │
│                                  PHASE 2 (IPsec SAs)                                   │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

* **Due IPsec SA Unidirezionali (*Unidirectional*):**
  * Per consentire il normale scambio dati bidirezionale, vengono generate **2 Security Association distinte**:
    * **Outbound SA:** Chiave specifica usata per cifrare i dati inviati al gateway remoto.
    * **Inbound SA:** Chiave diversa usata per decifrare i dati ricevuti dal gateway remoto.
* **Phase 1 Multi-Phase 2 (*One-to-Many*):**
  * Sotto una singola Phase 1 (un unico IKE SA stabilito) è possibile definire **molteplici Phase 2**.
  * Questo consente di applicare parametri crittografici differenti in base alla criticità dei dati (es. crittografia più forte per le sottoreti di database/finanza, crittografia più leggera per traffico standard).
* **Rekeying e Perfect Forward Secrecy (PFS):**
  * Quando le IPsec SA si avvicinano alla scadenza, IKE genera una nuova coppia di SA (**rekey della Child SA** in IKEv2 / Quick Mode in IKEv1).
  * **Perfect Forward Secrecy (PFS):** Se abilitato, obbliga FortiGate a eseguire un nuovo scambio Diffie-Hellman a ogni scadenza di Phase 2. Ciò garantisce che l'ipotetica compromissione della chiave di Phase 1 o di una precedente SA non permetta di decifrare il traffico delle sessioni future.

---

## 2. Phase 2 Selectors (Traffic Selectors / Proxy IDs)

I selettori di traffico determinano quali pacchetti IP hanno diritto di essere instradati e cifrati all'interno del tunnel IPsec.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                              PHASE 2 SELECTORS MATCHING                                │
├────────────────────────────────┬───────────────────────────────────────────────────────┤
│ Sede Locale (FortiGate A)      │ Sede Remota (FortiGate B)                             │
├────────────────────────────────┼───────────────────────────────────────────────────────┤
│ Local Address: 10.0.11.0/24    │ Remote Address: 10.0.11.0/24                          │
│ Remote Address: 172.20.1.0/24  │ Local Address: 172.20.1.0/24                          │
└────────────────────────────────┴───────────────────────────────────────────────────────┘
```

* **Regola di Corrispondenza Speculare (*Must Match*):**
  * Nelle connessioni punto-punto, i selettori tra i due firewall **devono essere perfettamente speculari**:
    * La `Local Address` del FortiGate locale deve coincidere con la `Remote Address` dell'altro apparato.
    * La `Remote Address` del FortiGate locale deve coincidere con la `Local Address` dell'altro apparato.
  * In IKEv1, se i selettori non combaciano esattamente, la negoziazione della Phase 2 fallisce. In IKEv2 è supportato il *Traffic Selector Narrowing* per negoziare un sottoinsieme comune.
* **Criteri di Selezione Configurabili:**
  * **Tipologia Indirizzo:** `IP Address`, `Subnet Address` (es. `10.0.11.0/255.255.255.0`), `IP Range`.
  * **Parametri Avanzati (L4):**
    * `Protocol number`: Tutti i protocolli (`All`) o uno specifico numero IP (es. TCP=6, UDP=17, ICMP=1).
    * `Local Port` e `Remote Port`: Tutte (`All`) o porte L4 dedicate (`Specify`).
* **Scarto del Traffico (*Drop*):**
  * Qualsiasi pacchetto indirizzato all'interfaccia VPN che non corrisponda rigorosamente ai selettori definiti viene scartato dal firewall.

---

## 3. Phase 2 Proposal & Opzioni di Sicurezza Avanzate

La sezione **Phase 2 Proposal** stabilisce gli algoritmi usati per cifrare il carico utile (payload ESP) scambiato tra le sedi.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                              PHASE 2 PROPOSAL PARAMETERS                               │
├────────────────────────┬───────────────────────────────────────────────────────────────┤
│ Encryption             │ DES, 3DES, AES128, AES256, AES128-GCM, AES256-GCM,            │
│                        │ CHACHA20POLY1305.                                             │
├────────────────────────┼───────────────────────────────────────────────────────────────┤
│ Authentication (Hash)  │ MD5, SHA1, SHA256, SHA384, SHA512.                            │
├────────────────────────┼───────────────────────────────────────────────────────────────┤
│ Replay Detection       │ Abilita la verifica dei numeri di sequenza ESP per prevenire  │
│                        │ attacchi di tipo Man-in-the-Middle Replay (impostazione       │
│                        │ locale non negoziata con il peer).                            │
├────────────────────────┼───────────────────────────────────────────────────────────────┤
│ Perfect Forward        │ Impone un nuovo calcolo Diffie-Hellman a ogni rinnovo di      │
│ Secrecy (PFS)          │ chiave Phase 2 (selezionabile con specifici DH groups).       │
└────────────────────────┴───────────────────────────────────────────────────────────────┘
```

* **Proposte Multiple:** È possibile dichiarare più righe di proposta crittografica (es. AES256-SHA256 e in fallback AES128-SHA256). Il firewall le valuterà in ordine sequenziale.
* **Impatto Prestazionale:** La scelta dell'algoritmo impatta direttamente la velocità di trasmissione e la capacità di delegare il calcolo al chip hardware ASIC.

---

## 4. Rinegoziazione delle Chiavi (Key Lifetime & Keepalive)

Questi parametri governano la frequenza e le modalità con cui le associazioni di sicurezza IPsec SA vengono rinnovate per prevenire interruzioni di servizio.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                             GESTIONE TIMEOUT & RINEGOZIAZIONE                          │
├────────────────────────┬───────────────────────────────────────────────────────────────┤
│ Key Lifetime           │ Scadenza delle SA basata su:                                  │
│                        │ • Seconds: scadenza a tempo (default tipico: 43200 s = 12h)   │
│                        │ • Kilobytes: scadenza a volume di dati transitati             │
│                        │ • Both: rinnova in base alla condizione che si verifica prima │
├────────────────────────┼───────────────────────────────────────────────────────────────┤
│ Asimmetria Lifetime    │ Le soglie di lifetime NON devono necessariamente coincidere   │
│                        │ tra i due peer: il tunnel si attiva negoziando il valore più  │
│                        │ conservativo/basso tra i due.                                 │
├────────────────────────┼───────────────────────────────────────────────────────────────┤
│ Auto-negotiate         │ Se abilitato, FortiGate avvia automaticamente il rekey delle  │
│                        │ SA poco prima che scadano, prevenendo drop di traffico.       │
├────────────────────────┼───────────────────────────────────────────────────────────────┤
│ Autokey Keep Alive     │ Mantiene attivo il tunnel VPN rinegoziando continuamente le   │
│                        │ chiavi anche in totale assenza di traffico utente reale.      │
└────────────────────────┴───────────────────────────────────────────────────────────────┘
```

---

## 5. IPsec Hardware Offloading (Accelerazione ASIC NPU)

Nei firewall FortiGate fisici, la cifratura e decifratura di massa del traffico IPsec può essere scaricata dalla CPU principale e affidata a chip dedicati chiamati **NPU (Network Processing Unit)**.

```
Traffico Senza Offload (CPU Bound):
[ Pacchetto ESP ] ───► Kernel FortiOS (CPU Generale) ───► Cifratura Software ───► Latenza Alta

Traffico con NPU Offload (Hardware Accelerated):
[ Pacchetto ESP ] ───► ASIC NPU (NP6 / NP7) ────────────► Cifratura Hardware ───► Throughput Multi-Gigabit
```

* **Funzionamento:**
  * L'NPU gestisce l'elaborazione dei pacchetti ESP a livello circuitale, garantendo throughput multi-gigabit e latenza prossima allo zero.
  * Le capacità di scarico e gli algoritmi supportati variano in base alla generazione del processore (es. NP6, NP6lite, NP7).
  * Di default, l'accelerazione hardware è **abilitata automaticamente** per tutti gli algoritmi supportati dall'ASIC (es. AES-CBC, AES-GCM).
* **Disabilitazione Manuale (`npu-offload disable`):**
  * Utile in fase di troubleshooting avanzato (es. se si sospettano anomalie hardware su pacchetti frammentati o se si desidera analizzare il pacchetto crudo via packet sniffer della CPU prima dell'elaborazione).
  * Si disabilita a livello di Phase 1 CLI:
```fortios
config vpn ipsec phase1-interface
    edit "ToRemote"
        set npu-offload disable
    next
end
```

---

## 6. Sintassi di Riferimento CLI (FortiOS)

Configurazione manuale completa di una Phase 2 collegata a una Phase 1 da terminale:

```fortios
config vpn ipsec phase2-interface
    edit "ToRemote_P2"
        set phase1name "ToRemote"
        set proposal aes256-sha256 aes128-sha256
        set pfs enable
        set dhgrp 14 20
        set replay enable
        set auto-negotiate enable
        set keepalive enable
        set keylifeseconds 43200
        set src-subnet 10.0.11.0 255.255.255.0
        set dst-subnet 172.20.1.0 255.255.255.0
    next
end
```
















