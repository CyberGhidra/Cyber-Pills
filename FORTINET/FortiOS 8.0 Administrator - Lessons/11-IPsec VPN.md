# FortiGate IPsec VPN — Guida Tecnica e Architetturale Ufficiale (FortiOS)

---

## 1. Fondamenti di IPsec e Obiettivi di Sicurezza

### 1.1 Scopo Principale
Il protocollo **IPsec (Internet Protocol Security)** consente di unire host e reti geograficamente distribuite all'interno di una rete privata logica sicura (*virtual private network*) estesa attraverso un'infrastruttura di transito pubblica, non fidata e ostile (Internet).

I due macro-scenari operativi sono:
- **Site-to-Site (Gateway-to-Gateway):** Connessione permanente di intere sottoreti LAN aziendali (es. filiali, sedi remote, data center primari o ambienti cloud IaaS come AWS/Azure) verso l'Headquarters (HQ).
- **Client-to-Site / Remote Access (Dialup):** Accesso temporaneo o persistente di singoli utenti in mobilità (*telelavoro / smart working*) mediante software endpoint dedicato (**FortiClient**) o client IPsec nativo del sistema operativo.

### 1.2 I Tre Pilastri della Sicurezza IPsec
IPsec implementa le tre garanzie crittografiche fondamentali della sicurezza informatica:

```
┌────────────────────────────────────────────────────────────────────────┐
│                        I TRE PILASTRI DI IPSEC                         │
├────────────────────┬─────────────────────────────┬─────────────────────┤
│   Autenticazione   │      Integrità Dati         │    Riservatezza     │
│  (Authentication)  │      (Data Integrity)       │   (Confidentiality) │
├────────────────────┼─────────────────────────────┼─────────────────────┤
│ Verifica certa     │ Garanzia anti-manomissione  │ Cifratura simmetrica│
│ dell'identità dei  │ (tamper-proofing). Il dato  │ del payload. Rende  │
│ peer: Pre-Shared   │ non è stato alterato da     │ i dati illeggibili  │
│ Key (PSK), Certi-  │ intermediari. Basata su     │ a chi intercetta il │
│ ficati X.509, EAP  │ hash crittografico (SHA-2). │ transito (AES).     │
└────────────────────┴─────────────────────────────┴─────────────────────┘
```

---

## 2. Suite di Protocolli, Incapsulamento e Gestione del NAT

### 2.1 Architettura della Suite: IKE, AH ed ESP
IPsec è un'architettura modulare composta da più standard cooperanti:
- **IKE (Internet Key Exchange):** Protocollo di controllo Layer 7 trasportato su UDP, deputato alla mutua autenticazione, negoziazione delle associazioni di sicurezza (SA) e calcolo delle chiavi simmetriche di sessione.
- **AH (Authentication Header - IP protocol 51):** Fornisce integrità e autenticazione anche all'header IP esterno, ma **non supporta alcuna cifratura** (il payload viaggia in chiaro).
- **ESP (Encapsulating Security Payload - IP protocol 50):** Fornisce cifratura, integrità, autenticazione dell'origine e protezione anti-replay.

> ⚠️ **Comportamento di FortiOS:**  
> Fortinet ha scelto di **non supportare il protocollo AH** nei firewall FortiGate. Tutti i tunnel IPsec in FortiOS operano esclusivamente tramite **ESP (RFC 4303)**.

---

### 2.2 Modalità di Incapsulamento ESP: Tunnel Mode vs. Transport Mode

IPsec opera a **Livello 3 dello stack OSI (Network Layer)**, risultando totalmente trasparente per le applicazioni e i protocolli di livello superiore (TCP, UDP, ICMP, SCTP, HTTP/S).

#### A. Senza VPN (*No VPN — Traffico Standard in Chiaro*)
```text
┌───────────────────────┬────────────────────────────────────────┐
│  Original IP Header   │        Payload TCP / UDP / SCTP        │
│  (IP Sorgente/Dest.)  │                + Dati                  │
└───────────────────────┴────────────────────────────────────────┘
```

#### B. Tunnel Mode (*Default Assoluto in FortiOS*)
L'intero pacchetto IP originario (compreso il suo header con indirizzi privati interni) viene cifrato e incapsulato all'interno di una nuova intestazione IP pubblica.

```text
┌──────────────┬────────────┬─────────────────────┬──────────────────────┬─────────────┬───────────────┐
│     New      │    ESP     │      Original       │      TCP / UDP       │     ESP     │ ESP Integrity │
│  IP Header   │   Header   │      IP Header      │        + Data        │   Trailer   │    (HMAC)     │
└──────────────┴────────────┴─────────────────────┴──────────────────────┴─────────────┴───────────────┘
                            │◄─────────────────────── CIFRATO ────────────────────────►│
               │◄─────────────────────────────── AUTENTICATO ──────────────────────────►│
```
- **New IP Header:** Contiene gli IP pubblici dei gateway VPN (es. IP WAN dei FortiGate).
- **Zona Cifrata:** Header IP originale privato + Dati L4/L7 + ESP Trailer.
- **Zona Autenticata:** Dall'ESP Header fino all'ESP Trailer (convalidata dall'HMAC ESP Integrity).
- **Uso Principale:** Collegamenti **Site-to-Site** e **Dialup FortiClient**.

#### C. Transport Mode (*Host-to-Host / Overlay*)
Protegge unicamente il carico di trasporto Layer 4. L'header IP originale non viene nascosto né sostituito.

```text
┌─────────────────────┬────────────┬──────────────────────┬─────────────┬───────────────┐
│      Original       │    ESP     │      TCP / UDP       │     ESP     │ ESP Integrity │
│     IP Header       │   Header   │        + Data        │   Trailer   │    (HMAC)     │
└─────────────────────┴────────────┴──────────────────────┴─────────────┴───────────────┘
                                   │◄────── CIFRATO ─────►│
                      │◄─────────────────── AUTENTICATO ────────────────►│
```
- **Original IP Header:** Rimane visibile all'esterno in chiaro e **non** è coperto dall'integrità crittografica ESP.
- **Uso Principale:** Connessioni dirette tra due host o all'interno di overlay di tunneling di livello superiore (**GRE over IPsec**, **L2TP over IPsec**).

#### Tabella Comparativa degli Incapsulamenti
| Parametro | Traffico in Chiaro (*No VPN*) | Tunnel Mode | Transport Mode |
| :--- | :--- | :--- | :--- |
| **New IP Header** | Assente | **Aggiunto** (IP pubblici gateway) | **Assente** |
| **Original IP Header** | Visibile in chiaro | **Cifrato** (occultato nel payload ESP)| **Visibile in chiaro** |
| **Integrità Header Orig.** | Assente | **Sì** (coperto da HMAC) | **No** (escluso da HMAC ESP) |
| **Overhead sul pacchetto**| Zero | Elevato (~40–56 Byte) | Minore (~20–36 Byte) |
| **Ambito di elezione** | Rete LAN / Internet | **Site-to-Site** & **Dialup** | **Host-to-Host** & **GRE/L2TP** |
| **Implementazione FortiOS**| Nativo | **Default su tutti i tunnel** | Esclusivo su richiesta CLI |

---

### 2.3 Gestione del NAT: NAT-Traversal (NAT-T, RFC 3948)

Il protocollo ESP nativo (**IP 50**) non possiede numeri di porta TCP o UDP a Layer 4. Qualsiasi router intermedio che esegue NAT o PAT (Port Address Translation) scarta i pacchetti ESP poiché non è in grado di tracciare la sessione di ritorno nella propria translation table.

Per risolvere questa criticità interviene il meccanismo **NAT-Traversal (NAT-T)**:

```text
Senza NAT (Direct Native ESP):
┌──────────────┬────────────────────────┬─────────────┐
│  IP Header   │ ESP Protocol (IP 50)   │ ESP Payload │
└──────────────┴────────────────────────┴─────────────┘

Con NAT-Traversal (UDP 4500 Encapsulation):
┌──────────────┬────────────────────────┬─────────────┬─────────────┐
│  IP Header   │ UDP (IP 17 - Port 4500)│ ESP Header  │ ESP Payload │
└──────────────┴────────────────────────┴─────────────┴─────────────┘
```

#### Mappatura Porte IKE ed ESP (Con NAT vs. Senza NAT)
| Flusso e Scopo | Senza NAT (*No NAT*) | Con NAT (*NAT-T Abilitato*) |
| :--- | :--- | :--- |
| **IKE Negotiation** (Phase 1 e Phase 2) | Protocollo IP 17: **UDP 500** | Inizia su **UDP 500** $\rightarrow$ commuta dinamicamente su **UDP 4500** appena viene riscontrato un NAT intermedio. |
| **ESP Data Traffic** (Flusso dati utenti) | Protocollo IP 50 nativo (**ESP**) | Protocollo IP 17: **UDP 4500** (il payload ESP viene incapsulato dentro segmenti UDP). |

#### Personalizzazione Porte da CLI FortiOS
Nel caso in cui un fornitore di connettività blocchi la porta standard UDP 500:
```fortios
config system settings
    set ike-port 10500     # Range consentito: da 1024 a 65535
end
```
*(Nota di servizio: come Responder per client NAT-T, FortiGate continua a restare sempre in ascolto sulla porta standard UDP 4500).*

---

## 3. Protocollo IKE: IKEv1 vs. IKEv2

IKE calcola dinamicamente le chiavi private di sessione senza trasmetterle sulla rete, sfruttando lo scambio matematico **Diffie-Hellman (DH)**.

```
┌────────────────────────────────────────────────────────────────────────┐
│                     MATRICE COMPARATIVA IKEv1 vs IKEv2                 │
├──────────────────────────┬──────────────────────┬──────────────────────┤
│ Parametro Architetturale │ IKEv1 (RFC 2409)     │ IKEv2 (RFC 7296)     │
├──────────────────────────┼──────────────────────┼──────────────────────┤
│ Exchange Modes           │ • Main: 9 messaggi   │ Scambio standard     │
│ (Numero pacchetti)       │ • Aggressive: 6 mess.│ unificato:           │
│                          │                      │ 4 messaggi totali    │
├──────────────────────────┼──────────────────────┼──────────────────────┤
│ Metodo di Autenticazione │ Simmetrico: entrambi │ Asimmetrico: i peer  │
│                          │ i lati usano lo      │ possono usare metodi │
│                          │ stesso metodo (PSK   │ differenti (es. uno  │
│                          │ o Certificati; XAuth)│ con Cert, uno EAP)   │
├──────────────────────────┼──────────────────────┼──────────────────────┤
│ Supporto NAT-T           │ Estensione opzionale │ Nativo nello standard│
├──────────────────────────┼──────────────────────┼──────────────────────┤
│ Affidabilità pacchetti   │ Unreliable (No ACK)  │ Reliable (ACK nativo)│
├──────────────────────────┼──────────────────────┼──────────────────────┤
│ Traffic Selector         │ Non supportato       │ Supportato           │
│ Narrowing                │ (I selettori devono  │ (Permette di restrin-│
│                          │ combaciare al bit)   │ gere la subnet pro-  │
│                          │                      │ posta dall'altro)    │
└──────────────────────────┴──────────────────────┴──────────────────────┘
```

---

## 4. Topologie VPN Site-to-Site

```
           TOPOLOGIE VPN SITE-TO-SITE A CONFRONTO

1. POINT-TO-POINT          2. HUB-AND-SPOKE             3. FULL MESH
    [Branch] <───> [HQ]           [Spoke1]               [S1] ─── [S2]
                                      │                    │  \   / │
                                  [HQ / Hub]               │   \ /  │
                                   /      \                │    X   │
                               [Spoke2]  [Spoke3]          │   / \  │
                                                         [S3] ─── [S4]
```

### 4.1 Caratteristiche delle Topologie
1. **Simple (Punto-Punto Diretto):** Collega esattamente due nodi. Configurazione rapida (1 Phase 1, 1 Phase 2), ma priva di scalabilità.
2. **Hub-and-Spoke (A Stella):** Tutti i rami remoti (*Spokes*) instaurano un solo tunnel verso il firewall centrale (*Hub*).
   - Numero di tunnel lineare: $N$.
   - Il traffico spoke-to-spoke deve attraversare l'Hub (*Hairpinning / Tromboning*).
   - Vantaggi: configurazione elementare per gli spoke, hardware leggero per le filiali.
   - Svantaggi: Single Point of Failure (SPOF) al centro, elevato consumo di banda per l'Hub.
3. **Full Mesh (Magliatura Completa):** Ogni sede ha un tunnel diretto con ciascuna delle altre sedi.
   - Formula di dimensionamento tunnel: $\frac{N(N-1)}{2}$. Esempio: 5 siti = 10 tunnel; 10 siti = 45 tunnel.
   - Vantaggi: latenza minima, massima tolleranza ai guasti (Fault Tolerant).
   - Svantaggi: configurazione complessa ed esponenziale, elevati requisiti hardware e di manutenzione su tutti i firewall.
4. **Partial Mesh (Magliatura Parziale):** Approccio ibrido in cui i siti nevralgici (Headquarters, Data Center, Disaster Recovery) sono completamente magliati tra loro, mentre le filiali periferiche mantengono collegamenti a stella verso i soli nodi primari.

### 4.2 Matrice Comparativa Ufficiale Fortinet
| Parametro / Metrica | Hub-and-Spoke | Partial Mesh | Full Mesh |
| :--- | :--- | :--- | :--- |
| **Complessità Configurazione** | **Easy** (Bassa) | **Moderate** (Media) | **Complex** (Elevata) |
| **Numero Complessivo Tunnel** | **Few** ($N$) | **Medium** | **Many** ($\frac{N(N-1)}{2}$) |
| **Banda Richiesta al Centro** | **High** | **Medium** (Ripartita) | **Low** (Distribuita) |
| **Tolleranza ai Guasti** | **Not fault tolerant** | **Some fault tolerance** | **Fault tolerant** |
| **Requisiti Hardware** | Low (Spoke) / High (Hub) | Medium su tutti i nodi | High su tutti i nodi |
| **Scalabilità** | **Scalable** | **Somewhat scalable** | **Difficult to scale** |
| **Comunicazione Diretta** | Solo via Hub centrale | Solo nodi scelti | Tra tutti i nodi |

---

## 5. Implementazione tramite FortiOS VPN Wizard

Il **VPN Wizard** (`VPN > VPN Wizard`) è la procedura guidata progettata per automatizzare il rilascio dei tunnel riducendo a zero le discrepanze di configurazione.

### 5.1 Modelli di Distribuzione (Template)
- **Site to Site:** Crea tunnel punto-punto permanenti verso altri FortiGate o router/firewall di terze parti (Cisco, Check Point, pfSense, Linux libreswan).
- **Hub and Spoke with ADVPN:** Configura l'Hub e gli spoke predisponendo l'infrastruttura **Auto Discovery VPN (ADVPN)**, consentendo alle sedi periferiche di istanziare tunnel dinamici temporanei spoke-to-spoke su richiesta.
- **Remote Access:** Configura l'infrastruttura di accesso dialup per client **FortiClient** con supporto per pool IP virtuali, assegnazione DNS e split-tunneling.
- **FortiClient Secure Internet Access (SIA):** Forza l'instradamento di tutto il traffico Internet dei telelavoratori all'interno del tunnel per ispezione centralizzata (Antivirus, Web Filter, IPS, Application Control).

### 5.2 Oggetti Generati Automaticamente nella Schermata `Review`
Al termine della procedura guidata, FortiOS crea istantaneamente:
1. **Addresses & Address Groups:** Oggetti subnet locali e remoti (`*_local`, `*_remote`).
2. **Interfaces & Zones:** Interfaccia logica virtuale di Phase 1 e associazione di Phase 2.
3. **Firewall Policies:** Regola Outbound (`Local_to_Remote`) e regola Inbound (`Remote_to_Local`) con NAT disabilitato.
4. **Static Routes:** Rotta di destinazione verso la subnet remota tramite l'interfaccia VPN.
5. **Blackhole Route:** Rotta con distanza amministrativa elevata (`254`) associata alla subnet remota con interfaccia blackhole.

> 🛡️ **Perché la Blackhole Route è Fondamentale?**  
> Se il tunnel IPsec cade (*down*), la rotta statica primaria verso l'interfaccia VPN viene rimossa dalla Forwarding Information Base (FIB). In assenza di una rotta blackhole, i pacchetti diretti alla subnet remota privata verrebbero instradati lungo la Default Route (`0.0.0.0/0`), finendo in chiaro su Internet verso l'ISP. La **Blackhole Route** intercetta e scarta silenziosamente questi pacchetti orfani, azzerando il rischio di *data leakage*.

---

## 6. Phase 1 — Architettura, Parametri e Negoziazione (IKE SA)

L'obiettivo della Phase 1 è autenticare i peer e generare una singola associazione di sicurezza **bidirezionale (IKE SA)** dedicata esclusivamente a proteggere il traffico di controllo.

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        FASE 1: CICLO IN 3 PASSI                        │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Autenticazione: Scambio di credenziali (PSK o Certificati PKI/EAP). │
│ 2. Scambio Diffie-Hellman: Calcolo congiunto della chiave segreta      │
│    condivisa (Shared Secret) senza mai trasmetterla in chiaro.         │
│ 3. Canale Cifrato IKE SA: Negoziazione completata. Si attiva il canale │
│    protetto per dare il via alla Phase 2.                              │
└────────────────────────────────────────────────────────────────────────┘
```

### 6.1 Parametri Network & Modalità di Remote Gateway
- **IP Version:** IPv4 o IPv6.
- **Interface:** Porta WAN fisica o logica del FortiGate su cui IKE resta in ascolto.
- **Local Gateway:** Permette di forzare l'IP sorgente locale (Specify) in presenza di IP secondari.
- **Remote Gateway Modes:**
  - `Static IP address`: Il peer remoto possiede un IP pubblico statico. Entrambi i firewall possono agire come **Initiator** o **Responder**.
  - `Dialup user`: L'indirizzo IP del peer è sconosciuto a priori (es. client FortiClient o branch con IP dinamico privo di DDNS). Il FortiGate locale può operare **esclusivamente come Responder**.
  - `Dynamic DNS`: Il peer è identificato tramite un Fully Qualified Domain Name (FQDN) risolto periodicamente dal DNS di sistema.

### 6.2 IKE Mode Config
Disponibile solo in modalità **Dialup user**. Agisce come un server DHCP integrato in IKE:
- **Assegnazione IP:** Assegna un IP privato al client attingendolo da un IP Range locale, da un server DHCP o da un Address Group.
- **DNS Server:** Invia al client i server DNS interni per consentire la risoluzione dei record dell'intranet.
- **Split Tunneling:**
  - *Enabled:* Invia nel tunnel solo i pacchetti destinati alle subnet protette aziendali; il traffico Internet generale dell'utente esce direttamente dal suo gateway locale.
  - *Disabled:* Invia tutto il traffico utente nel tunnel verso il FortiGate.

### 6.3 Dead Peer Detection (DPD) e Keepalive
Monitora la disponibilità del tunnel per evitare l'instradamento di pacchetti verso gateway non più raggiungibili:
- **On Demand (Default raccomandato):** Invia query probe DPD **solo** se c'è traffico locale in uscita ma non si riceve traffico di ritorno dal peer da un tempo definito. Risparmia CPU e banda.
- **On Idle:** Invia probe a intervalli regolari ogni volta che non transita alcun pacchetto.
- **Disabled:** Non invia probe attivi; si limita a rispondere a quelli inviati dal peer.
- **Soglie:** `Retry Count` (default: 3) e `Retry Interval` (default: 20s). Al superamento della soglia, l'IKE SA viene abbattuta e la rotta rimossa dalla FIB.

---

## 7. Phase 2 — Incapsulamento Dati, Selettori e NPU Offload (IPsec SA)

La **Phase 2** genera le associazioni di sicurezza che proteggono il traffico reale degli utenti tramite il protocollo **ESP**.

### 7.1 Regola d'Oro: Le IPsec SA sono UNIDIREZIONALI
Mentre la IKE SA di Phase 1 è bidirezionale, le associazioni di Phase 2 sono **unidirezionali**:
- **Outbound SA:** Cifra i pacchetti inviati al gateway remoto (associata a uno specifico SPI - *Security Parameter Index*).
- **Inbound SA:** Decifra i pacchetti ricevuti dal gateway remoto (associata a un diverso valore di SPI).
- **Rapporto:** Per una comunicazione bidirezionale servono sempre **due SA distinte**. Sotto una singola Phase 1 è possibile istanziare **molteplici Phase 2** per applicare cifrari differenti a sottoreti diverse.

```text
[ Tuo FortiGate ] ─── Outbound IPsec SA (Cifra traffico di andata) ───► [ Firewall Remoto ]
[ Tuo FortiGate ] ◄─── Inbound IPsec SA  (Decifra traffico di ritorno) ── [ Firewall Remoto ]
```

### 7.2 Phase 2 Selectors (Traffic Selectors / Proxy IDs)
Definiscono quali pacchetti IP possono transitare all'interno del tunnel.
- **Criterio Speculare Obbligatorio:** Nelle connessioni statiche punto-punto:
  - `Local Address (Locale)` = `Remote Address (Remoto)`
  - `Remote Address (Locale)` = `Local Address (Remoto)`
- **Discrepanza Selettori:** In IKEv1, se i selettori non combaciano esattamente, la Phase 2 fallisce. In IKEv2 interviene la funzione di **Traffic Selector Narrowing** che tenta di negoziare la subnet comune più restrittiva.

### 7.3 Perfect Forward Secrecy (PFS)
Se abilitato, PFS impone al motore crittografico di eseguire un nuovo scambio Diffie-Hellman a ogni rinnovo della chiave di Phase 2 (`Key Lifetime`). In questo modo, l'ipotetica compromissione passata o futura della chiave di Phase 1 o di una SA non consente di decifrare i dati delle sessioni successive.

---

### 7.4 IPsec Hardware Acceleration (NPU Offloading)

Nei firewall FortiGate dotati di chip ASIC proprietari dedicati alla rete (**NPU — Network Processing Unit**, serie NP6, NP6lite, NP7), la cifratura e decifratura dei pacchetti ESP viene interamente delegata all'hardware.

```
Elaborazione Software (CPU Bound):
[ Frame ESP ] ───► Kernel FortiOS (CPU Generale) ───► Calcolo Cripto ──► Latenza Alta

Elaborazione Hardware (NPU FastPath):
[ Frame ESP ] ───► ASIC NPU (NP6 / NP7) ────────────► Cripto Wire-Speed ─► Throughput Multi-Gbps
```
- **Stato Predefinito:** L'accelerazione hardware (`npu-offload`) è abilitata automaticamente per tutti i cifrari supportati dall'ASIC (es. AES-CBC, AES-GCM).
- **Disabilitazione in Troubleshooting:** Si disattiva da CLI per diagnosticare problemi di frammentazione anomala o per catturare i frame completi decifrati tramite il packet sniffer della CPU prima dell'offload.

---

## 8. Routing e Firewall Policy per Tunnel IPsec

FortiOS adotta un modello VPN **Route-Based**: ogni tunnel IPsec genera un'interfaccia logica virtuale visibile all'interno della tabella di routing e selezionabile come porta di ingresso/uscita nelle policy.

### 8.1 Requisito 1: Routing Lookup
Affinché il traffico raggiunga la sede remota, deve esistere una rotta statica nella FIB:
- **Destination:** Subnet remota (es. `172.20.1.0/24`).
- **Interface:** L'interfaccia virtuale del tunnel IPsec creato.
- **Gateway:** Lasciato a `0.0.0.0` (il tunnel è un'interfaccia punto-punto logica; il FortiGate non richiede un next-hop IP).

### 8.2 Requisito 2: Firewall Policy
Il traffico deve essere esplicitamente autorizzato. Se entrambe le sedi devono poter originare connessioni in modo indipendente, servono **due regole distinte**:

#### Policy Outbound (Dalla LAN verso il Tunnel Remoto)
- **Incoming Interface:** Porta LAN interna (es. `internal`).
- **Outgoing Interface:** Interfaccia tunnel VPN.
- **Source:** Subnet LAN locale (es. `10.0.11.0/24`).
- **Destination:** Subnet remota (es. `172.20.1.0/24`).
- **Action:** `ACCEPT`.
- **NAT:** **DISABILITATO** *(Disattivare il flag NAT è mandatorio: i pacchetti devono preservare gli indirizzi privati RFC 1918 originali. Se si abilita il NAT, la sorgente viene mascherata e la comunicazione diretta fallisce).*

#### Policy Inbound (Dal Tunnel Remoto verso la LAN)
- **Incoming Interface:** Interfaccia tunnel VPN.
- **Outgoing Interface:** Porta LAN interna.
- **Source:** Subnet remota.
- **Destination:** Subnet LAN locale.
- **Action:** `ACCEPT`.
- **NAT:** **DISABILITATO**.

---

## 9. Tecnologie Avanzate ed Emergenti in FortiOS

### 9.1 TLS 1.3 VPN Key Negotiation Over TCP (IKE over TCP)
- **Contesto:** In presenza di reti geografiche o captive portal che bloccano totalmente il traffico UDP (UDP 500/4500 ed ESP nativo IP 50), FortiOS permette di incapsulare l'intera sessione VPN all'interno di una connessione TCP protetta da **TLS 1.3**.
- **Payload:** Sia il flusso di controllo IKEv2 che i pacchetti dati ESP vengono incapsulati dentro il socket TLS.
- **Vantaggi TLS 1.3:** Eliminazione di suite di cifratura deboli e handshake rapido a 1-RTT.

```fortios
config system settings
    set ike-tcp-service enable
end
```

### 9.2 Post-Quantum Cryptography (PQC) per IPsec IKEv2
- **Scopo:** Proteggere la fase di autenticazione e scambio chiavi dalla futura minaccia dei computer quantistici su larga scala (in grado di decifrare gli schemi RSA ed ECC tramite l'algoritmo di Shor).
- **RFC 7427 Framework:** Consente la negoziazione di firme digitali *algorithm-agnostic* in IKEv2.
- **Algoritmi NIST Integrati in FortiOS:**
  - **ML-DSA (FIPS 204):** Basato su reticoli algebrici (*Module-Lattice-Based Digital Signature Algorithm*).
  - **SLH-DSA (FIPS 205):** Basato su alberi di hash (*Stateless Hash-Based Digital Signature Algorithm*).

```fortios
config vpn ipsec phase1-interface
    edit "PQC_Tunnel"
        set authmethod signature
        set digital-signature-auth enable
        set signature-hash-alg identity
    next
end
```

---

## 10. Configurazione CLI Completa di Riferimento

Configurazione completa manuale di un tunnel IPsec Site-to-Site tra due apparati FortiGate:

```fortios
# -------------------------------------------------------------
# 1. Configurazione Phase 1 Interface (IKE SA)
# -------------------------------------------------------------
config vpn ipsec phase1-interface
    edit "ToRemote_HQ"
        set interface "port1"
        set peertype any
        set net-device disable
        set ike-version 2
        set proposal aes256-sha256 aes128-sha256
        set dhgrp 14 20
        set remote-gw 198.51.100.1
        set psksecret "F0rt1n3tC3rt1f13d!2026"
        set dpd-mode on-demand
        set dpd-retrycount 3
        set dpd-retryinterval 20
        set npu-offload enable
    next
end

# -------------------------------------------------------------
# 2. Configurazione Phase 2 Interface (IPsec SA)
# -------------------------------------------------------------
config vpn ipsec phase2-interface
    edit "ToRemote_HQ_P2"
        set phase1name "ToRemote_HQ"
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

# -------------------------------------------------------------
# 3. Routing: Rotta Primaria e Blackhole di Sicurezza
# -------------------------------------------------------------
config router static
    edit 101
        set dst 172.20.1.0 255.255.255.0
        set device "ToRemote_HQ"
    next
    edit 102
        set dst 172.20.1.0 255.255.255.0
        set blackhole enable
        set distance 254
    next
end

# -------------------------------------------------------------
# 4. Firewall Policies (NAT Rigorosamente Disabilitato)
# -------------------------------------------------------------
config firewall policy
    edit 10
        set name "LAN_to_Remote_IPsec"
        set srcintf "port2"
        set dstintf "ToRemote_HQ"
        set srcaddr "LAN_Locale_10.0.11.0"
        set dstaddr "LAN_Remota_172.20.1.0"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat disable
    next
    edit 11
        set name "Remote_to_LAN_IPsec"
        set srcintf "ToRemote_HQ"
        set dstintf "port2"
        set srcaddr "LAN_Remota_172.20.1.0"
        set dstaddr "LAN_Locale_10.0.11.0"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat disable
    next
end
```

---

## 11. Comandi CLI Essenziali per Diagnostica e Troubleshooting

Quando un tunnel IPsec presenta disservizi o non si attiva, i comandi CLI di diagnostica consentono di isolare l'anomalia in tempo reale:

```fortios
# 1. Verifica dello stato dei tunnel attivi e dei gateway negoziati
diagnose vpn ike gateway list

# 2. Controllo delle Security Association (SA) di Phase 2 e contatori traffico/SPI
diagnose vpn tunnel list

# 3. Debug in tempo reale del motore IKE (analizza negoziazioni, errori PSK o selettori errati)
diagnose vpn ike log-filter dst-addr4 <IP_Pubblico_Remoto>
diagnose debug application ike -1
diagnose debug enable

# 4. Per interrompere la sessione di debug
diagnose debug reset
diagnose debug disable

# 5. Flush manuale per forzare la rinegoziazione immediata del tunnel
diagnose vpn ike gateway clear name <Nome_Phase1>
diagnose vpn tunnel flush <Nome_Phase1>
```
