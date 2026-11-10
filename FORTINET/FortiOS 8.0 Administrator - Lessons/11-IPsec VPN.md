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


























