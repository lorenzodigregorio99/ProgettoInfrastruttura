# Enterprise Infrastructure Lab: Multi-Router Routing, PKI Auto-Enrollment & Secure VPN L2TP

## 📌 Descrizione del Progetto
Questo progetto documenta l'implementazione pratica di un'infrastruttura di rete aziendale interamente virtualizzata su **VMware**. Il laboratorio simula l'interconnessione di sicurezza tra due sedi tramite una **VPN Site-to-Site** configurata tra due router perimetrali, e permette a un client remoto situato in una subnet differente di connettersi in **VPN L2TP/IPsec** direttamente al Domain Controller aziendale per fruire di servizi web critici protetti da mutua autenticazione SSL/TLS.

---

## 🗺️ Architettura e Flussi di Rete
L'infrastruttura è strutturata su tre macro-aree logiche:
1. **Sede Centrale (HQ) - Subnet 192.168.1.0/24:** Ospita il Domain Controller principale (`DC1`) e il server web `IIS`. Questa rete è attestata sull'interfaccia LAN del **Router 1**.
2. **Rete di Transito WAN (Internet):** Rete geografica simulata che unisce le interfacce pubbliche di **Router 1** e **Router 2**, sui quali è stabilita una **VPN Site-to-Site** permanente con credenziali speculari su entrambi gli apparati.
3. **Sede Remota (Branch) - Subnet 10.0.0.0/24:** Ospita il **Client 7** (Windows 7). Questa subnet è attestata sul **Router 2**, il quale è configurato come **DHCP Relay Agent** per inoltrare le richieste broadcast dei client verso il server DHCP situato su `DC1` nella Sede Centrale.

---

## 🛠️ Dettagli Implementativi e Servizi Configurati

### 1. Servizi di Directory e Network Core (DC1)
*   **Active Directory Domain Services (AD DS):** Configurazione della foresta e del dominio aziendale. Creazione del gruppo di sicurezza personalizzato `"Utenti Certificati"` e popolamento con gli utenti dedicati ai test di acesso.
*   **DHCP Server:** Configurazione dell'ambito (scope) e dei range di indirizzamento IP dedicati alla subnet remota (`10.0.0.0/24`), abilitando il rilascio centralizzato degli indirizzi IP per il perimetro di rete gestito dal Relay Agent.
*   **Routing e Accesso Remoto (RRAS):** Configurazione del servizio di routing e attivazione dell'endpoint VPN direttamente sul Domain Controller per accettare le connessioni inbound dei client abilitati.

### 2. Public Key Infrastructure (PKI) Avanzata e Auto-Enrollment
*   **Enterprise Certification Authority (Root CA):** Configurazione della CA integrata in Active Directory per l'emissione del *Certificato 0* (Root aziendale).
*   **Auto-Enrollment di Gruppo (GPO):** Implementazione di Group Policy Objects specifiche per l'autodistribuzione e il rilascio automatico dei certificati digitali sia alle macchine fisiche (Computer) sia ai profili utente del dominio.
*   **Certificati Extended Validation (EV):** Configurazione della CA per il rilascio di certificati a convalida estesa (EV) per la protezione dei server web, distribuiti e validati come "Trusted" su tutti gli endpoint tramite GPO.

### 3. Server Web IIS con Mutua Autenticazione (HTTPS)
*   **Internet Information Services (IIS):** Pubblicazione di due siti web aziendali indipendenti.
*   **Hardening SSL/TLS:** Configurazione dei binding di rete per imporre l'utilizzo esclusivo del protocollo sicuro **HTTPS** e associazione dei certificati EV generati dalla CA.
*   **Filtro Accessi basato su Certificato Client:** Configurazione delle regole di IIS per richiedere obbligatoriamente un certificato client valido (*Require Client Certificates*). L'accesso ai siti è consentito esclusivamente agli utenti appartenenti al gruppo `"Utenti Certificati"` in possesso della propria chiave privata.

### 4. Configurazione dei Router e Architettura VPN
*   **Router 1 & Router 2 (Remote Access):** Configurazione del routing statico e dei servizi di accesso remoto su entrambi i nodi perimetrali.
*   **Tunnel VPN Site-to-Site:** Configurazione di una connessione VPN permanente tra i due router per garantire la connettività di base inter-subnet e il transito del traffico di rete.
*   **DHCP Relay Agent (Router 2):** Configurazione del demone di relay sull'interfaccia LAN del Router 2. Il servizio intercetta le richieste DHCP Discover (broadcast) del Client 7 e le instrada via unicast attraverso il tunnel verso l'IP del server DHCP (`DC1`).
*   **VPN Remote Access L2TP (Client 7):** Configurazione sul Client 7 di una connessione VPN di tipo **L2TP/IPsec** protetta da certificati. Il tunnel termina direttamente su `DC1`, consentendo al client di superare il perimetro di rete, autenticarsi tramite **RADIUS**, ottenere un IP della rete e verificare la corretta raggiungibilità dei siti IIS protetti in HTTPS.

---

## 📈 Competenze Tecniche Dimostrate (Hard Skills)
*   **Sistemi Operativi:** Windows Server 2012, Windows 7 (Client).
*   **Networking e Routing:** Routing Statico, DHCP Relay Agent, VPN Site-to-Site, Subnetting multi-classe.
*   **Infrastruttura VPN:** L2TP/IPsec basata su certificati, RADIUS Authentication, Servizi RRAS.
*   **PKI & Sicurezza IT:** Enterprise CA, PKI Auto-Enrollment (User & Computer), Certificati Extended Validation (EV), GPO Root Trust.
*   **Web Server Management:** Internet Information Services (IIS), Certificati Client (Mutua Autenticazione), Binding HTTPS.
*   **Virtualizzazione:** Gestione avanzata di reti isolate su VMware Workstation / ESXi.
