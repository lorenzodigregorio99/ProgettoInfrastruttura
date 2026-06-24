# Enterprise Infrastructure Lab: Multi-Router Routing, PKI Auto-Enrollment & Secure VPN L2TP

## 📌 Descrizione del Progetto
Questo progetto documenta l'implementazione pratica di un'infrastruttura di rete aziendale interamente virtualizzata su **VMware**. Il laboratorio simula l'interconnessione di sicurezza tra due sedi tramite una **VPN Site-to-Site** configurata tra due router perimetrali, e permette a un client remoto situato in una subnet differente di connettersi in **VPN L2TP/IPsec** direttamente al Domain Controller aziendale per fruire di servizi web critici protetti da mutua autenticazione SSL/TLS.

---

## 🗺️ Architettura e Schema degli Indirizzi IP
L'infrastruttura è strutturata su tre macro-aree logiche con la seguente mappatura di rete:

1. **Sede Centrale (HQ) - Subnet LAN: `192.168.1.0/24`**
   *   **DC1 (Domain Controller & IIS):** Configurato con doppio indirizzo IP:
       *   `192.168.1.2` (Interfaccia LAN principale per i servizi di dominio, DNS, DHCP, IIS)
       *   `192.168.254.100` (Interfaccia logica/dedicata per l'endpoint di terminazione VPN)
   *   **Router 1 (Gateway HQ):** `192.168.1.1`

2. **Rete di Transito WAN (Internet Pubblico) - Subnet: `200.0.0.0/30`**
   *   **Router 1 (Interfaccia WAN):** `200.0.0.1`
   *   **Router 2 (Interfaccia WAN):** `200.0.0.2`
   *   *Nota:* Su questa tratta è stabilita la **VPN Site-to-Site** permanente con credenziali speculari su entrambi gli apparati.

3. **Sede Remota - Subnet LAN: `10.0.0.0/24`**
   *   **Router 2 :** `10.0.0.1` (Configurato anche come DHCP Relay Agent)
   *   **Client 7 (Windows 7):** Ottiene l'IP dinamicamente tramite DHCP all'interno della rete `10.0.0.0/24`.

---

## 🛠️ Dettagli Implementativi e Servizi Configurati

### 1. Servizi di Directory e Network Core (DC1)
*   **Active Directory Domain Services (AD DS):** Configurazione della foresta e del dominio aziendale. Creazione del gruppo di sicurezza personalizzato `"Utenti Certificati"` e popolamento con gli utenti dedicati ai test di accesso.
*   *   <img width="572" height="614" alt="image" src="https://github.com/user-attachments/assets/498e34c4-4f16-480c-9842-1630f5188e99" />
*   **DHCP Server:** Configurazione dell'ambito (scope) e dei range di indirizzamento IP dedicati alla subnet remota (`10.0.0.0/24`), abilitando il rilascio centralizzato degli indirizzi IP per il perimetro di rete gestito dal Relay Agent.
*   <img width="485" height="255" alt="image" src="https://github.com/user-attachments/assets/b6f36d64-b85a-4974-be03-bba43ab225c2" />


*   **Routing e Accesso Remoto (RRAS):** Configurazione del servizio di routing e attivazione dell'endpoint VPN sull'IP dedicato `192.168.254.100` per accettare le connessioni inbound dei client abilitati.
        <img width="994" height="444" alt="image" src="https://github.com/user-attachments/assets/180f97e7-5576-4807-9b0f-826cedd00744" />
    
### 2. Public Key Infrastructure (PKI) Avanzata e Auto-Enrollment
*   **Enterprise Certification Authority (Root CA):** Configurazione della CA integrata in Active Directory per l'emissione del *Certificato 0* (Root aziendale).
*   <img width="1423" height="868" alt="image" src="https://github.com/user-attachments/assets/1f3ac4d6-a7b2-4f8f-bd8d-48ab30f79a37" />

*   **Auto-Enrollment di Gruppo (GPO):** Implementazione di Group Policy Objects specifiche per l'autodistribuzione e il rilascio automatico dei certificati digitali sia alle macchine fisiche (Computer) sia ai profili utente del dominio.
*   <img width="1625" height="849" alt="image" src="https://github.com/user-attachments/assets/82d80cee-b0f1-430e-be04-a68e0b0ebb89" />

*   **Certificati Extended Validation (EV):** Configurazione della CA per il rilascio di certificati a convalida estesa (EV) per la protezione dei server web, distribuiti e validati come "Trusted" su tutti gli endpoint tramite GPO.

### 3. Server Web IIS con Mutua Autenticazione (HTTPS)
*   **Internet Information Services (IIS):** Pubblicazione di due siti web aziendali indipendenti sull'IP `192.168.1.2`.
*   **Hardening SSL/TLS:** Configurazione dei binding di rete per imporre l'utilizzo esclusivo del protocollo sicuro **HTTPS** e associazione dei certificati EV generati dalla CA.
*   **Filtro Accessi basato su Certificato Client:** Configurazione delle regole di IIS per richiedere obbligatoriamente un certificato client valido (*Require Client Certificates*). L'accesso ai siti è consentito esclusivamente agli utenti appartenenti al gruppo `"Utenti Certificati"` in possesso della propria chiave privata.
*   <img width="1675" height="810" alt="image" src="https://github.com/user-attachments/assets/8ebb6ee8-b387-4d17-be7e-f96695d40232" />


### 4. Configurazione dei Router e Architettura VPN
*   **Router 1 & Router 2 (Remote Access):** Configurazione del routing statico e dei servizi di accesso remoto su entrambi i nodi perimetrali.
*   **Tunnel VPN Site-to-Site:** Configurazione di una connessione VPN permanente certificata tra Router 1 (`200.0.0.1`) e Router 2 (`200.0.0.2`) per garantire la connectività di base inter-subnet e il transito del traffico di rete.
*   <img width="1019" height="493" alt="image" src="https://github.com/user-attachments/assets/f7f746fa-36aa-4ac7-9e8b-77673fd05a6b" />

     
*   **DHCP Relay Agent (Router 2):** Configurazione del demone di relay sull'interfaccia LAN `10.0.0.1`. Il servizio intercetta le richieste DHCP Discover (broadcast) del Client 7 e le instrada via unicast attraverso il tunnel verso l'IP del server DHCP (`192.168.1.2`).
*   <img width="1108" height="344" alt="image" src="https://github.com/user-attachments/assets/407a9efd-3add-4973-bee5-11ad194065d5" />

*   **VPN Remote Access L2TP (Client 7):** Configurazione sul Client 7 di una connessione VPN di tipo **L2TP/IPsec** protetta da certificati. Il tunnel termina sull'interfaccia `192.168.254.100` di `DC1`, consentendo al client di superare il perimetro di rete, ottenere un IP della rete e verificare la corretta raggiungibilità dei siti IIS protetti in HTTPS.
*   <img width="553" height="224" alt="image" src="https://github.com/user-attachments/assets/da3ff8d6-3d17-42da-bd80-4ebaced5f9ec" />
<img width="497" height="207" alt="image" src="https://github.com/user-attachments/assets/1e196f74-735f-4106-947b-c34e0b369456" />



---

## 📈 Competenze Tecniche Dimostrate (Hard Skills)
*   **Sistemi Operativi:** Windows Server 2012, Windows 7 (Client).
*   **Networking e Routing:** Routing Statico, DHCP Relay Agent, VPN Site-to-Site, Subnetting multi-classe, Gestione interfacce multi-IP.
*   **Infrastruttura VPN:** L2TP/IPsec basata su certificati, RADIUS Authentication, Servizi RRAS.
*   **PKI & Sicurezza IT:** Enterprise CA, PKI Auto-Enrollment (User & Computer), Certificati Extended Validation (EV), GPO Root Trust.
*   **Web Server Management:** Internet Information Services (IIS), Certificati Client (Mutua Autenticazione), Binding HTTPS.
*   **Virtualizzazione:** Gestione avanzata di reti isolate su VMware Workstation / ESXi.
