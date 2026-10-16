# Fortinet FortiOS: Fondamenti di Rete e Sicurezza

## 1. Il Contesto Moderno della Sicurezza di Rete

### 1.1 Evoluzione del Firewall
Il firewall moderno non agisce più come un semplice "filtro di frontiera" (packet filter / portiere perimetrale). A causa dell'iper-distribuzione degli ambienti aziendali, il perimetro di rete tradizionale è dissolto. Un Next-Generation Firewall (NGFW) deve proteggere scenari eterogenei:
- **Forza lavoro remota e ibrida** (utenti mobili, telelavoro, VPN/ZTNA).
- **Fornitori e partner terzi** con accessi transitori e granulari.
- **Infrastrutture Cloud Ibride** (Private Cloud, AWS, Azure, GCP).
- **Dispositivi IoT/OT** (spesso privi di agenti di sicurezza a bordo).
- **BYOD (Bring Your Own Device)** non gestiti centralmente.

### 1.2 Casi d'Uso e Funzionalità

#### Scenari di Deployment (Per Uso)
* **Enterprise Distributed Firewall:** Dislocato in sedi distaccate e filiali collegate via SD-WAN.
* **Next-Generation Firewall (NGFW):** Posizionato al perimetro Internet per ispezione applicativa profonda e content security.
* **Internal Segmentation Firewall (ISFW):** Installato all'interno del core network per prevenire il movimento laterale di malware tra VLAN diverse.
* **Data Center Firewall:** Apparati ad altissimo throughput e latenza minima per carichi server intensivi.

#### Funzionalità di Rete e Sicurezza (Per Funzione)
* **Servizi Core di Rete:** DNS Server/Relay, DHCP Server, Dynamic Routing.
* **Ispezione di Sicurezza:** Deep Packet Inspection (DPI) SSL/TLS, IPS (Intrusion Prevention System), Antivirus/Antimalware, Web & Video Filtering, Application Control.

---

### 1.3 Architettura Hardware e Software
I sistemi FortiGate combinano hardware accelerato proprietario con un sistema operativo unificato:

* **Hardware - SPU (Security Processing Unit):**
  * **NP (Network Processor):** Esegue l'offload delle funzioni di rete e routing L3/L4 ad altissima velocità.
  * **CP (Content Processor):** Accelera la crittografia/decrittografia SSL/IPsec e la scansione euristica/antivirus.
  * **SoC (System-on-a-Chip):** Integra core CPU, NP e CP su un unico chip per apparati entry/mid-range (modelli serie 40F–90F).
* **Software - FortiOS:** Sistema operativo proprietario condiviso da tutti i modelli fisici, virtuali (VM) e cloud.

#### Moduli e Licenze FortiGuard
| Modulo | Modalità di Licenza | Descrizione |
| :--- | :--- | :--- |
| **Application Control** | Incluso con supporto base | Riconoscimento e categorizzazione del traffico applicativo L7 |
| **Antivirus / Sandboxing** | Sottoscrizione FortiGuard | Analisi di file, stream e mitigazione malware zero-day |
| **Web Filtering** | Sottoscrizione FortiGuard | Categorizzazione URL dinamica, ispezione DNS/HTTP/HTTPS |
| **IPS (Intrusion Prevention)** | Sottoscrizione FortiGuard | Rilevamento e blocco exploit di vulnerabilità e attacchi di rete |
| **Industrial / OT Security** | Sottoscrizione add-on | Firme per protocolli SCADA e ambienti industriali |

---

### 1.4 L'Ecosistema Fortinet Fabric
FortiGate è il nodo centrale del **Fortinet Security Fabric**, integrandosi in modo nativo con soluzioni dedicate:
* **FortiWeb:** Web Application Firewall (WAF) specializzato per applicazioni web e API.
* **FortiMail:** Secure Email Gateway contro phishing, spam e minacce via posta elettronica.
* **FortiSandbox:** Ispezione dinamica in ambiente isolato per file e URL sospetti.
* **FortiClient EMS:** Gestione centralizzata degli endpoint (ZTNA client, vulnerabilità, telemetria).

---

## 2. Accesso Iniziale e Modalità Operative (Factory Default)

### 2.1 Accesso Out-of-the-Box per Categoria Hardware

#### Dispositivi Entry-Level (es. FortiGate 40F, 60F, 70F)
* **Porte LAN fisiche (`internal` / switch virtuale):** Di fabbrica le porte 1–4 condividono un'interfaccia switch con IP predefinito `192.168.1.99/24` e **DHCP Server attivo**.
* **Accesso:** È sufficiente collegare un computer via cavo su una porta LAN: la scheda di rete riceverà un IP nel range `192.168.1.110 - 192.168.1.210` e la GUI sarà raggiungibile su `https://192.168.1.99`.

#### Dispositivi Mid/High-End (es. FortiGate 100F, 200F e superiori)
* **Porta MGMT dedicata:** Dispongono di una porta fisica denominata `mgmt` o `mgmt1`.
* **Configurazione predefinita:** IP statico `192.168.1.99/24`, ma **senza DHCP server**. Il PC deve essere configurato manualmente (es. `192.168.1.100/24`).

#### Credenziali di Primo Accesso
* **Username:** `admin`
* **Password:** *(vuota)*
* **Protocolli attivi:** HTTPS, HTTP (reindirizzato su HTTPS), Ping, SSH.
* Al primo login la GUI richiede obbligatoriamente la creazione di una nuova password robusta.

---

### 2.2 Modalità Operative: NAT Mode vs. Transparent Mode

```
+------------------------------------------------------------------------+
| NAT / Route Mode (Default - Layer 3)                                  |
| [LAN: 192.168.10.0/24] <---> [ FortiGate (Gateway/Router) ] <---> WAN  |
+------------------------------------------------------------------------+

+------------------------------------------------------------------------+
| Transparent Mode (Layer 2 Bridge)                                     |
| [LAN] <---> [ Core Switch ] <---> [ FortiGate ] <---> [ Edge Router ]  |
|                               (Stessa subnet L2)                       |
+------------------------------------------------------------------------+
```

1. **NAT / Route Mode (Predefinita):**
   * Lavora a **Livello 3 (Network)**.
   * Il FortiGate agisce da router e default gateway tra diverse subnet.
   * Gestisce traduzione degli indirizzi (SNAT/DNAT), routing statico/dinamico, VPN IPSec e policy di sicurezza.
2. **Transparent Mode:**
   * Lavora a **Livello 2 (Data Link)** come un bridge invisibile.
   * Non esegue routing e non modifica gli header IP dei pacchetti.
   * È ideale per inserire funzioni di sicurezza (IPS, Web Filter, Antivirus) in infrastrutture esistenti senza modificare piani di indirizzamento o gateway.

---

## 3. Configurazione delle Interfacce di Rete

Navigazione da GUI: **Network > Interfaces**

### 3.1 Modalità di Assegnazione IP
* **Manual:** Assegnazione di un indirizzo IP statico e relativa subnet mask (es. `192.168.1.1/24`).
* **DHCP:** L'interfaccia richiede i parametri di rete a un server DHCP esterno (tipico per WAN domestiche o backup LTE).
* **PPPoE:** Autenticazione punto a punto per linee broadband/FTTC/FTTH che richiedono credenziali ISP.
* **IPAM (IP Address Management):** Assegnazione centralizzata gestita tramite Fabric/IPAM.

### 3.2 Ruoli (Role) e Alias
* **Role (Ruolo):** Modifica contestualmente le opzioni visibili nella GUI per semplificare la configurazione:
  * `LAN`: Mostra le configurazioni DHCP Server, Device Detection e LLDP.
  * `WAN`: Nasconde le opzioni server locali e attiva parametri specifici di connettività Internet (stima banda, IP dinamici).
  * `DMZ`: Ottimizzata per subnet esposte a servizi pubblici.
  * `Undefined`: Mantiene disponibili tutte le opzioni avanzate senza filtri visivi.
* **Alias:** Descrizione personalizzata e intuitiva associata alla porta fisica (es. `port1` $\rightarrow$ `Alias: LAN_Uffici_Piano1`).

### 3.3 Accesso Amministrativo e Protocolli di Gestione

Nel riquadro **Administrative Access** dell'interfaccia è possibile definire selettivamente i protocolli consentiti:
* `HTTPS` / `HTTP`: Accesso all'interfaccia grafica web (porta 443/80 di default).
* `SSH`: Accesso alla shell testuale (porta 22).
* `PING`: Risposta alle richieste ICMP Echo per test di raggiungibilità.
* `SNMP`: Monitoraggio tramite protocolli di rete (v2c / v3).
* `Security Fabric Connection`: Abilita le porte dedicate (`TCP 8013`) per collegare l'apparato all'ecosistema Fabric.
* `FMG-Access`: Apre il servizio FGFM (`TCP 541`) necessario per la gestione centralizzata da FortiManager.

#### Supporto LLDP (Link Layer Discovery Protocol)
* **LLDP (Layer 2):** Permette agli apparati adiacenti di scambiarsi informazioni su identità, porte e funzionalità.
* **Receive LLDP / Transmit LLDP:** Abilita l'invio e la ricezione di frame LLDP. In ambienti Fortinet, permette al FortiGate di rilevare switch FortiSwitch e access point FortiAP a valle o altri firewall a monte.

---

## 4. Configurazione del Server DHCP

Per abilitare FortiGate come server DHCP per la rete locale:
1. Andare su **Network > Interfaces** e modificare l'interfaccia di destinazione (es. `lan`).
2. Impostare il **Role** su `LAN`.
3. Inserire l'indirizzo IP statico del firewall (che farà da gateway per i client, es. `192.168.1.1/24`).
4. Attivare il toggle **DHCP Server**.

### Parametri di Configurazione DHCP
* **Address Range:** L'intervallo di indirizzi IP rilasciabili (es. `192.168.1.100` - `192.168.1.200`).
* **Netmask:** Subnet mask associata (derivata automaticamente dall'IP dell'interfaccia, es. `255.255.255.0`).
* **Default Gateway:**
  * *Same as Interface IP* (consigliato: i client useranno l'IP del FortiGate come gateway).
  * *Specify* (per instradare verso un gateway alternativo).
* **DNS Server:**
  * *Same as System DNS* (utilizza i server DNS configurati sul FortiGate).
  * *Specify* (consente di distribuire DNS locali Active Directory o pubblici es. `1.1.1.1`).
* **Lease Time:** Durata della validità dell'assegnazione IP prima del rinnovo (default: `7 giorni`, riducibile in reti guest o ad alto turnover).

---

## 5. Routing Statico: Default Route per Accesso a Internet

Senza una rotta di default, il FortiGate non è in grado di inoltrare il traffico generato dai client locali verso l'esterno.

Navigazione da GUI: **Network > Static Routes > Create New**

### Campi Obbligatori
* **Destination:** `0.0.0.0/0.0.0.0` (rappresenta qualsiasi destinazione esterna / default gateway).
* **Gateway:** Indirizzo IP del router/modem a monte (fornito dal provider internet, es. `192.168.100.254` o IP pubblico punto-punto).
* **Interface:** L'interfaccia fisica o logica connessa a Internet (es. `wan1`).
* **Administrative Distance (AD):** Valore di default `10` su FortiOS. Definisce la priorità della rotta; un valore più basso indica maggiore affidabilità rispetto ad altre rotte con lo stesso prefisso.

---

## 6. Canali di Gestione e Automazione

### 6.1 Interfacce di Controllo
* **Web GUI:** Console grafica basata su widget, log viewer e visualizzazione interattiva dei flussi di traffico.
* **CLI (Console / SSH):** Riga di comando completa per configurazioni avanzate, debug e automazioni batch.
* **REST API:** Endpoint HTTPS per interrogare lo stato del dispositivo ed eseguire configurazioni programmatiche (tramite token API in `System > Administrators`).
* **FortiManager:** Piattaforma centralizzata multi-tenant per il deployment di policy, template di configurazione e aggiornamenti firmware su flotte estese di FortiGate.

### 6.2 Integrazione FortiAI e GenAI
FortiOS integra assistenti basati su intelligenza artificiale per semplificare troubleshooting, generazione di comandi CLI e indagini di sicurezza:
* **Prerequisiti:** Servizi cloud attivi, licenza FortiCare/FortiGuard valida e account con permessi di amministratore.
* **Provider:** Supporta l'assistente nativo FortiAI o l'integrazione tramite chiavi API esterne (es. OpenAI).
* **Best Practice di Sicurezza:** Non inoltrare mai credenziali, chiavi private, certificati o dati confidenziali nei prompt dell'assistente.

---

## 7. Amministrazione e Hardening di Sicurezza

### 7.1 Profili Amministrativi e RBAC
Navigazione: **System > Admin Profiles**

I profili definiscono i privilegi granulari applicati ai singoli amministratori su ogni modulo di sistema (`Security Profile`, `Firewall Policy`, `Router`, `System`, `Log & Report`):
* **Livelli di Permesso:**
  * `None`: Nessuna visibilità.
  * `Read`: Sola lettura (consultazione).
  * `Read/Write`: Controllo completo.
  * `Custom`: Regolazione specifica per sottomodulo.
* **Permessi CLI:** Possibilità di abilitare, disabilitare o filtrare comandi operativi (es. inibire `config` e consentire solo `get` o `show`).
* **Override Idle Timeout:** Impostazione di un timeout di disconnessione automatica per inattività specifico per il profilo (es. 10 minuti).

#### Modello Gerarchico Consigliato (Principle of Least Privilege)
1. **Super Admin (`super_admin`):** Accesso illimitato globale su tutti i moduli e VDOM.
2. **Security & Network Admin:** Modifica di policy firewall, oggetti e routing, senza accesso ai parametri globali di sistema.
3. **Helpdesk / Operatori L1:** Visualizzazione in sola lettura di sessioni attive, policy e diagnostica.
4. **Auditor / Compliance:** Accesso esclusivo alla consultazione e all'esportazione dei log di sistema.

---

### 7.2 Restrizione degli Accessi (Trusted Hosts e 2FA)
Navigazione: **System > Administrators**

* **Trusted Hosts:** Obbliga l'amministratore a collegarsi esclusivamente da subnet o indirizzi IP specifici (es. `10.0.1.50/32` per la workstation di gestione), bloccando i tentativi provenienti da altri host anche se provvisti di credenziali corrette.
* **Autenticazione a Due Fattori (2FA):** Supporto integrato a **FortiToken** (fisico o mobile) o server RADIUS/SAML esterni per garantire l'accesso sicuro.

---

### 7.3 Hardening Password e Porte di Sistema
Navigazione: **System > Settings**

* **Porte di Amministrazione:** Modificare le porte standard di ascolto per ridurre il rischio di scansioni automatiche:
  * HTTPS: Spostare dalla porta predefinita `443` a una porta dedicata (es. `8443`).
  * SSH: Spostare dalla porta predefinita `22` (es. `2222`).
* **Criteri Complessità Password (Password Policy):**
  * `Password Scope`: Applicazione della policy agli amministratori locali e/o utenti di rete.
  * Lunghezza minima caratteri (consigliato $\ge 12$).
  * Requisiti di complessità: lettere maiuscole, minuscole, numeri e caratteri speciali.
  * `Password History`: Impedisce il riutilizzo delle ultime password inserite (es. blocco delle ultime 5 password).

---

### 7.4 Limitazione Metodi di Login via CLI
È possibile inibire specifici vettori di accesso per singoli account amministrativi tramite la CLI di FortiOS:

#### Sintassi Base
```fortios
config system admin
    edit "<nome_amministratore>"
        set disallowed-login-methods {console | gui | ssh | telnet}
    next
end
```

#### Esempio Operativo
Per impedire all'utente `Mario` l'accesso da Console fisica, interfaccia Web GUI e sessioni SSH:
```fortios
config system admin
    edit "Mario"
        set disallowed-login-methods console gui ssh
    next
end
```
