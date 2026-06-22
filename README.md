# ProgettoInfrastruttura
# Enterprise Infrastructure Lab: Multi-Site Routing, PKI & VPN Architecture

## 📌 Descrizione del Progetto
Questo progetto consiste nella progettazione, implementazione e documentazione di un'infrastruttura di rete aziendale complessa e interamente virtualizzata all'interno di un ambiente **VMware**. 
L'obiettivo principale è simulare l'interconnessione sicura tra una **Sede Centrale (HQ)** e una **Sede Remota (Branch Office)**, implementando una gestione centralizzata delle identità, una Public Key Infrastructure (PKI) interna e una doppia architettura VPN (Site-to-Site e Remote Access).

---

## 🗺️ Topologia di Rete (Network Diagram)

```
<!-- ![Network Diagram](diagramma.png) -->

---

## 🛠️ Dettagli Implementativi e Servizi Configurati

### 1. Identity & Network Services (Sede Centrale)
*   **Active Directory Domain Services (AD DS):** Creazione del dominio aziendale, configurazione della foresta e annessione centralizzata dei client. Gestione granulare di utenti, gruppi e permessi di sicurezza.
*   **Group Policy Management (GPO):** Implementazione di policy oggettive per la distribuzione automatica delle configurazioni di sicurezza e la mappatura delle risorse.
*   **DHCP Server:** Configurazione dell'ambito (scope) di rete con esclusioni dedicate, prenotazioni IP statiche e implementazione di **DHCP Relay Agent** per consentire il corretto instradamento delle richieste broadcast tra subnet differenti.
*   **DNS Server:** Configurazione di zone di ricerca diretta e inversa per la corretta risoluzione dei nomi dei servizi interni ed esterni.

### 2. Public Key Infrastructure (PKI) & Web Security
*   **Certification Authority (CA):** Configurazione e deployment di una CA interna per la gestione del ciclo di vita dei certificati digitali (rilascio, rinnovo e revoca).
*   **Automazione Trust Client:** Distribuzione della chiave pubblica della Root CA tramite GPO, garantendo che tutti i client appartenenti al dominio riconoscano automaticamente come **Trusted** le connessioni e i servizi interni.
*   **Internet Information Services (IIS):** Configurazione del server web aziendale protetto tramite binding HTTPS e associazione di certificati avanzati (simulazione Extended Validation - EV).

### 3. Storage & Business Continuity
*   **iSCSI Storage Services:** Configurazione dello storage centralizzato target/initiator per l'allocazione logica dello spazio disco su rete.
*   **RAID Configuration:** Ottimizzazione della tolleranza ai guasti dell'hardware di archiviazione attraverso la strutturazione logica dei volumi in array RAID.
*   **File Sharing:** Configurazione di cartelle condivise di rete protette da permessi NTFS e partizionamento delle risorse per dipartimento aziendale.

### 4. Networking, Routing & Doppia Architettura VPN
*   **Routing and Remote Access (RRAS):** Configurazione dei servizi di routing e di accesso remoto per l'instradamento inter-network del traffico dati.
*   **VPN Site-to-Site:** Implementazione di un tunnel crittografato permanente a livello infrastrutturale tra **Router 1** e **Router 2**, unendo in sicurezza la rete locale della sede centrale (`192.168.1.0/24`) e della sede remota (`10.0.0.0/24`).
*   **VPN Remote Access:** Configurazione di accessi esterni multi-protocollo flessibili per gli utenti in mobilità, testando e validando:
    *   *L2TP/IPsec certificata* (Standard aziendale sicuro basato su certificati digitali).
    *   *IKEv2 certificata* (Alta stabilità e riconnessione automatica).
    *   *PPTP* (Utilizzato in ambiente di test per analisi legacy).
*   **Server RADIUS (NPS):** Centralizzazione dei processi di autenticazione, autorizzazione e accounting (AAA) per tutti gli utenti che tentano l'accesso tramite i tunnel VPN.
*   **Firewall Security:** Configurazione restrittiva delle regole in entrata ed uscita (**Inbound/Outbound rules**) sui sistemi host e perimetrali, applicando rigorosamente il principio del minimo privilegio.

---

## 📈 Competenze Tecniche Dimostrate (Hard Skills)
*   **Sistemi Operativi:** Windows Server 2012, Linux (Ubuntu, AlmaLinux, Kali Linux).
*   **Networking:** Routing, Switching, Subnetting, VPN (Site-to-Site, Remote Access), L2TP, IKEv2, PPTP, RADIUS.
*   **Sicurezza e Infrastruttura:** Active Directory, PKI, Certificate Authority, Windows Firewall, GPO, DNS, DHCP.
*   **Storage & Web:** iSCSI, RAID, IIS Web Server.
*   **Virtualizzazione:** VMware Workstation / ESXi.

