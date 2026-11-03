# Fortinet FortiOS: Digital Certificates, SSL/SSH Inspection & VPN Cryptography
---

## Panoramica Esecutiva

Nel moderno panorama della sicurezza di rete, oltre l'80% del traffico web aziendale transita su protocolli cifrati (HTTPS/TLS). Se da un lato la crittografia protegge la privacy e l'integrità del dato contro intercettazioni esterne, dall'altro crea un pericoloso punto cieco (*Blind Spot*) per i Next-Generation Firewall (NGFW).

FortiOS integra una gestione avanzata dell'infrastruttura a chiave pubblica (PKI), motori di ispezione crittografica hardware-accelerati (SoC/CP9/CP10) e protocolli emergenti di cifratura quantistica e incapsulamento VPN.

---

## 1. Fondamenti dei Certificati Digitali in FortiOS

### 1.1 Definizione e Pilastri di Sicurezza
Un certificato digitale è una credenziale crittografica conforme allo standard **ITU-T X.509v3**, emessa e firmata da una **Certificate Authority (CA)** fidata, che lega l'identità di un soggetto a una specifica chiave pubblica.

I tre scopi primari nel FortiGate sono:
1. **Autenticazione (Authentication):** Verifica incontrovertibile dell'identità di server, client, interfacce o nodi VPN, proteggendo da attacchi *Man-in-the-Middle (MitM)* e *IP/DNS spoofing*.
2. **Cifratura e Scambio Chiavi (Encryption):** Permette di validare la chiave pubblica usata per negoziare chiavi simmetriche effimere di sessione (es. via ECDHE).
3. **Integrità (Integrity):** Garantisce che il payload e le intestazioni scambiate non siano stati alterati lungo il percorso.

```
       ┌────────────────────────────────────────────────────────┐
       │             AMBITI D'USO IN FORTIOS                    │
       ├─────────────────────────┬──────────────────────────────┤
       │ 1. INSPECTION           │ 2. PRIVACY & AUTHENTICATION  │
       ├─────────────────────────┼──────────────────────────────┤
       │ • SSL/SSH Deep Inspection│ • Portale GUI & SSH Admin    │
       │ • Outbound Forward Proxy│ • Portale Captive / ZTNA     │
       │ • Inbound Reverse Proxy │ • SSL-VPN & IPsec (IKEv2)    │
       │ • Decrypted Mirroring   │ • 2FA con Client Certificate │
       └─────────────────────────┴──────────────────────────────┘
```

---

### 1.2 Campi Critici dello Standard X.509v3
FortiOS analizza campi specifici per determinare la validità e l'associazione dell'identità:

* **Subject (DN - Distinguished Name):** Il nome univoco dell'entità (es. `CN=training.fortinet.com, O=Fortinet, C=US`).
* **Subject Alternative Name (SAN):** Estensione obbligatoria nei moderni stack TLS. FortiOS e i browser danno precedenza assoluta al SAN per validare la corrispondenza con l'FQDN richiesto (es. `DNS:training.fortinet.com`, `IP:10.1.1.1`).
* **Serial Number:** Identificativo esadecimale assegnato dalla CA; fondamentale per il tracciamento all'interno delle liste di revoca (**CRL**) e interrogazioni **OCSP**.
* **Validity Period (`Valid From` / `Valid To`):** Finestra temporale di validità.  
  > ⚠️ **Regola Operativa FortiOS:** Se il clock di sistema del FortiGate (sincronizzato via NTP) non ricade nell'intervallo temporale, la negoziazione SSL fallisce istantaneamente.

---

## 2. Meccanica della Verifica della Firma Digitale da parte di FortiGate

FortiGate esegue una procedura rigorosa in tre fasi per convalidare qualsiasi certificato ricevuto durante un handshake TLS:

```
[ PASSO 1: Calcolo Hash Locale ]
Certificato Ricevuto ─────────► [ Funzione Hash (es. SHA-256) ] ─────────► Hash A (Calcolato)

[ PASSO 2: Decifratura Firma ]
Firma Digitale CA    ─────────► [ Decifratura con Chiave Pubblica CA ] ──► Hash B (Originale)

[ PASSO 3: Verifica ]
Hash A == Hash B ?
  ├── SI ──► CERTIFICATO VALIDO (Autenticità della CA e Integrità del dato verificate)
  └── NO ──► CERTIFICATO NON VALIDO (Dati manomessi o chiave privata CA non corrispondente)
```

1. **Fase 1 (Fresh Hash):** FortiGate applica al corpo del certificato la funzione crittografica di hashing indicata nell'algoritmo di firma (es. SHA-256), ottenendo un'impronta calcolata in tempo reale.
2. **Fase 2 (Decifratura Signature):** FortiOS recupera dal proprio archivio (*Remote CA / Intermediate Store*) il certificato pubblico della CA che ha firmato il certificato. Usa la chiave pubblica della CA per decifrare il blocco di firma, estraendo l'hash originale generato dalla CA emittente.
3. **Fase 3 (Matching & Policy Action):** I due hash vengono confrontati. Se coincidono perfettamente, integrità e autenticità sono confermate. In caso di discrepanza, la connessione viene terminata o gestita in base alle impostazioni di errore SSL del profilo.

---

## 3. Il Dilemma del Traffico Cifrato: Architetture a Confronto

Senza ispezione SSL, il firewall opera come un semplice router di Livello 3/4 o L7 limitato ai metadati:

```
+------------------------------------------------------------------------------------+
| 1. NO SSL INSPECTION (BLIND SPOT ASSOLUTO)                                         |
|                                                                                    |
|  [ Client ] <============== Sessione TLS Cifrata End-to-End =============> [ Server]
|                             ▲                                                      |
|                             │                                                      |
|                    [ FortiGate Cieco ]                                             |
|                    • I Security Profiles (AV, IPS, DLP, File Filter) NON vedono     |
|                      il payload. Ransomware e malware transitano indisturbati.     |
+------------------------------------------------------------------------------------+
| 2. SSL CERTIFICATE INSPECTION (CONTROLLO SUPERFICIALE)                             |
|                                                                                    |
|  [ Client ] <============== Sessione TLS Cifrata End-to-End =============> [ Server]
|                             ▲                                                      |
|                             │ Legge SNI e Certificato Server                       |
|                    [ FortiGate Sni-Filter ]                                        |
|                    • Valuta solo FQDN e Categoria Web.                             |
|                    • Payload rimane cifrato: NESSUNA scansione file/antivirus.     |
+------------------------------------------------------------------------------------+
| 3. FULL SSL INSPECTION (DEEP PACKET INSPECTION - PROXY MITM AUTORIZZATO)          |
|                                                                                    |
|  [ Client ] <--- Sessione TLS 1 ---> [ FortiGate ] <--- Sessione TLS 2 ---> [ Server]
|                                      │ Decifra  │                                  |
|                                      │ Scansiona│ (AV, IPS, File Filter, DLP)     |
|                                      │ Ri-cifra │ con certificato CA al volo      |
+------------------------------------------------------------------------------------+
```

### Tabella Comparativa delle Modalità

| Caratteristica | No Inspection | SSL Certificate Inspection | Full SSL Inspection (Deep Packet) |
| :--- | :--- | :--- | :--- |
| **Punto di Terminazione TLS** | Client-Server | Client-Server | Spezzato: Client-FG e FG-Server |
| **Visibilità Payload** | Nulla ($0\%$) | Nulla ($0\%$) | Completa ($100\%$) |
| **Web Filtering** | Solo per IP | Per Dominio/SNI/Host | URL path completo (`/path/file.exe`) |
| **Protezione AntiVirus / DLP** | ❌ Disattivata | ❌ Disattivata | ✅ Funzionante in real-time |
| **Impatto su CPU Firewall** | Minimo | Basso | Medio/Alto (offload su chip CP9/CP10) |
| **Certificato CA su Client** | Non Richiesto | Non Richiesto | **Obbligatorio** (per evitare warning) |
| **Protocolli Supportati** | Qualsiasi | HTTPS | HTTPS, SMTPS, POP3S, IMAPS, FTPS |

---

## 4. Configurazione e Tuning del Profilo SSL/SSH Inspection

Il profilo viene definito in **Security Profiles > SSL/SSH Inspection** e applicato direttamente alle regole in **Firewall Policy**.

### 4.1 Tipologia di Profilo
* **Multiple Clients Connecting to Multiple Servers (Outbound / Forward Proxy):** Ispezione del traffico generato dai client interni verso Internet. Richiede la rigenerazione dinamica dei certificati mediante CA.
* **Protecting SSL Server (Inbound / Reverse Proxy):** Protezione di web farm interne esposte verso l'esterno. Il FortiGate detiene il certificato reale del server e la corrispondente chiave privata; non genera certificati dinamici, ma esegue l'offloading SSL per scansionare il traffico in ingresso (WAF, IPS, AntiVirus).

### 4.2 Gestione Multi-Server tramite SNI (Inbound)
In scenari multi-tenant o con un singolo IP pubblico che pubblica più siti:
* FortiOS legge il campo **Server Name Indication (SNI)** nel pacchetto `Client Hello`.
* Associa l'SNI ai certificati locali configurati nel profilo.
* Se non trova corrispondenza, applica la prima chiave della lista come opzione di fallback.

### 4.3 Gestione dei Certificati Remoti Non Validi
FortiOS permette di personalizzare la risposta verso certificati remoti non affidabili (*Expired, Revoked, Untrusted CA, Validation Failed*):
* **Block:** Chiude la sessione TCP (invio di TCP RST) o presenta una pagina di blocco FortiOS. È la scelta consigliata in ottica Zero Trust.
* **Keep Untrusted & Allow:** FortiGate rincapsula il pacchetto verso il client firmandolo con una CA non attendibile interna, costringendo il browser dell'utente a mostrare il warning nativo.
* **Ignore:** Il firewall firma comunque il certificato fraudolento con la propria CA fidata, mascherando l'errore all'utente. **Pratica sconsigliata** perché espone la rete a frodi crittografiche.

### 4.4 Esenzioni SSL (SSL Exemptions)
Indispensabili per:
1. **Certificate Pinning:** Applicazioni client rigide (es. WhatsApp, Apple Updates, Dropbox, banking apps) che rifiutano CA intermedie.
2. **Privacy & Normative (GDPR / HIPAA):** Divieto di intercettare credenziali bancarie, sanitarie o dati sensibili del personale.
3. **Risparmio Risorse:** Escludere flussi ad alto volume e basso rischio.

Metodi configurabili:
* **Reputable Websites:** Database cloud FortiGuard costantemente aggiornato.
* **Web Categories:** Esclusione per intere macro-categorie (es. *Finance and Banking*, *Health and Wellness*).
* **Addresses / FQDN:** Oggetti di rete dedicati (es. endpoint Microsoft 365, Google Workspace).

---

## 5. Gestione del FortiGate Certificate Store

Amministrabile tramite **System > Certificates**, la struttura del database certificati di FortiOS è organizzata come segue:

```
┌────────────────────────────────────────────────────────────────────────┐
│                      FORTIGATE CERTIFICATE STORE                       │
├────────────────────────────────────────────────────────────────────────┤
│ 1. CRL (Certificate Revocation Lists)                                  │
│    Verifica di revoca locale; aggiornamento HTTP, LDAP o SCEP.         │
├────────────────────────────────────────────────────────────────────────┤
│ 2. Local CA Certificate                                                │
│    Certificati con chiave privata autorizzati a emettere certificati.  │
│    • Requisiti X.509: Basic Constraints: cA=True                       │
│                       Key Usage: keyCertSign                           │
│    • Default: Fortinet_CA_SSL (Self-signed)                            │
│    • Enterprise: Subordinate CA emessa da Microsoft AD CS / PKI interna│
├────────────────────────────────────────────────────────────────────────┤
│ 3. Local Certificate                                                   │
│    Certificati con chiave privata residenti a bordo del FortiGate.     │
│    • Certificato Portale GUI Admin (Fortinet_GUI_Server)               │
│    • Terminazione SSL-VPN e IPsec Gateway                              │
│    • Reverse Proxy per Server Interni protetti                         │
│    • CSR generate in attesa di firma (Pending CSR)                     │
├────────────────────────────────────────────────────────────────────────┤
│ 4. Remote CA Certificate                                               │
│    Bundle di CA Root/Intermediate attendibili per validare server      │
│    esterni durante l'handshake in uscita.                              │
└────────────────────────────────────────────────────────────────────────┘
```

### 5.1 Strategia di Distribuzione della CA per Deep Inspection
* **Approccio Self-Signed (`Fortinet_CA_SSL`):** Richiede il download del file `.cer` da FortiGate e l'installazione su ciascun dispositivo client nell'archivio *Autorità di certificazione radice attendibili*. Adatto a laboratori o reti di piccole dimensioni.
* **Approccio Enterprise (Subordinate CA):** Generare una CSR da FortiGate come CA intermedia e farla firmare dalla Root CA aziendale Active Directory. Tutti i dispositivi del dominio riconosceranno la catena senza necessità di distribuire certificati dedicati sugli endpoint.

---

## 6. Crittografia Avanzata per VPN in FortiOS

### 6.1 IKE/ESP Over TCP con TLS 1.3
Nativamente, IPsec sfrutta porte e protocolli UDP:
* **UDP 500:** IKEv2 Phase 1/2
* **UDP 4500:** NAT-Traversal
* **IP 50:** ESP (Encapsulating Security Payload)

In ambienti ostili (es. Wi-Fi pubblici con captive portal restrittivi o carrier che bloccano pacchetti UDP/ESP), FortiOS supporta l'incapsulamento dell'intera sessione IKE/ESP in un socket TCP cifrato con **TLS 1.3**.

```
[ Pacchetto IPsec Reale: IKEv2 + ESP ]
                 │
                 ▼
 ┌──────────────────────────────────────────────┐
 │   Infrastruttura TLS 1.3 su Porta TCP (443)  │  <-- Bypassa i blocchi UDP
 └──────────────────────────────────────────────┘
```

#### Vantaggi di TLS 1.3:
* **Latenza Ridotta:** Handshake in $1\text{-RTT}$ (rispetto ai $2\text{-RTT}$ di TLS 1.2), velocizzando la fase iniziale di negoziazione del tunnel.
* **Security Hardening:** Rimozione totale di cipher obsoleti o deboli (RC4, 3DES, CBC mode, SHA-1).

#### Configurazione CLI FortiOS:
```fortios
config system settings
    set ike-tcp-service enable
end
```

---

### 6.2 Crittografia Post-Quantistica (PQC) per IPsec (RFC 7427)
I futuri computer quantistici renderanno obsoleti gli algoritmi asimmetrici tradizionali basati su fattorizzazione o logaritmo discreto (RSA, Diffie-Hellman, ECDSA).

FortiOS introduce l'autenticazione post-quantistica per tunnel IPsec IKEv2 sfruttando lo standard **RFC 7427** (*IKEv2 Digital Signature Framework*), che disaccoppia la negoziazione delle chiavi dallo schema di firma.

#### Algoritmi NIST PQC Supportati:
1. **ML-DSA (FIPS 204):** *Module-Lattice-Based Digital Signature Algorithm* (derivato da CRYSTALS-Dilithium). Basato su problemi algebrici complessi su reticoli vettoriali.
2. **SLH-DSA (FIPS 205):** *Stateless Hash-Based Digital Signature Algorithm* (derivato da SPHINCS+). Basato esclusivamente sulla sicurezza delle funzioni di hash crittografiche, garantendo resilienza crittografica anche in caso di compromissione dei reticoli.

#### Configurazione Phase-1 CLI FortiOS:
```fortios
config vpn ipsec phase1-interface
    edit "PQC-Tunnel"
        set interface "wan1"
        set ike-version 2
        set authmethod signature
        set digital-signature-auth enable
        set signature-hash-alg identity
        set certificate "PQC_Cert_Local"
    next
end
```

---

## 7. Sintesi dei Comandi CLI per Troubleshooting

```fortios
# Visualizzare lo stato e i certificati installati a bordo
get vpn certificate local detail
get vpn certificate ca detail

# Verificare le sessioni SSL Deep Inspection attive gestite dal demone WAD/IPS
diagnose wad ssl-session list
diagnose wad stats

# Debug real-time dell'handshake SSL e dell'ispezione
diagnose debug reset
diagnose debug application wad 255
diagnose debug enable

# Debug per VPN IKE over TCP
diagnose vpn ike log-filter dst-port 443
diagnose debug application ike -1
diagnose debug enable
```
