**Lezione 4**

Configurazione Sicura degli Switch, DHCP, ARP e Fondamenti di Routing

# 1. Gestione remota degli apparati: Telnet e SSH

Gli switch e i router di rete possono essere amministrati da remoto tramite protocolli di accesso alla CLI (Command-Line Interface). Il protocollo storicamente utilizzato e Telnet; oggi e considerato insicuro e deve essere sostituito da SSH.

## 1.1 Perche Telnet è insicuro

Telnet (RFC 854, 1983) trasmette in chiaro su TCP porta 23 la totalità della sessione: username, password e ogni comando impartito. Un attaccante in grado di intercettare il traffico di rete — tramite una cattura passiva sulla stessa rete, un attacco ARP poisoning, o un nodo intermedio compromesso — può leggere direttamente le credenziali amministrative e riprodurre o alterare i comandi.

I problemi principali di Telnet sono:

- Nessuna cifratura: i dati viaggiano in chiaro sul layer TCP.

- Nessuna autenticazione del server: il client non verifica l'identità del server, rendendo possibili attacchi man-in-the-middle.

- Nessuna integrità dei dati: il contenuto della sessione può essere alterato in transito.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Approfondimento — SSH: Secure Shell</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p>SSH (Secure Shell) e un protocollo crittografico definito in RFC 4251-4254 (SSHv2, 2006) che fornisce accesso remoto sicuro a dispositivi di rete e server. E’ composto da tre sotto-protocolli sovrapposti:</p>
<p><strong>1. SSH Transport Layer Protocol (RFC 4253):</strong> negozia l'algoritmo crittografico (key exchange), autentica il server tramite la propria host key, e stabilisce una connessione cifrata e autenticata con controllo di integrità. Algoritmi di key exchange tipici: Diffie-Hellman (DH) e sua variante ECDH (Elliptic Curve DH). Algoritmi di cifratura simmetrica: AES-128/256-CTR, AES-128/256-GCM, ChaCha20-Poly1305. MAC: HMAC-SHA2-256, HMAC-SHA2-512.</p>
<p><strong>2. SSH User Authentication Protocol (RFC 4252):</strong> autentica il client verso il server. Metodi supportati: password (sconsigliato per amministrazione di rete), publickey (RSA, ECDSA, Ed25519 — metodo preferito), keyboard-interactive.</p>
<p><strong>3. SSH Connection Protocol (RFC 4254):</strong> multiplexing di sessioni logiche (channel) su un'unica connessione SSH. Una singola connessione puo trasportare piu sessioni interattive, port forwarding, tunneling X11 ecc.</p>
<p>Funzionamento sintetico della connessione SSHv2:</p>
<ol type="1">
<li><p>Client apre connessione TCP verso porta 22 del server.</p></li>
<li><p>Scambio di versione: entrambi annunciano la versione SSH supportata.</p></li>
<li><p>Negoziazione algoritmi: client e server concordano cifratura, MAC, key exchange.</p></li>
<li><p>Key exchange (es. ECDH): entrambi derivano un segreto condiviso senza trasmetterlo.</p></li>
<li><p>Server autentica se stesso tramite la propria host key (firma digitale). Il client verifica la host key contro il proprio known_hosts.</p></li>
<li><p>Client autentica se stesso (password o chiave pubblica).</p></li>
<li><p>Sessione cifrata e integra stabilita.</p></li>
</ol>
<p>La verifica della host key al punto 5 e fondamentale: un client che accetta senza verificare e vulnerabile a MITM. Al primo collegamento, SSH chiede all'operatore di accettare e memorizzare la fingerprint (TOFU: Trust On First Use). Nei sistemi automatizzati, la host key deve essere distribuita in modo sicuro prima dell'uso.</p>
<p>Confronto SSHv1 vs SSHv2:</p>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<thead>
<tr>
<th><p>SSH v1 - singolo protocollo monolitico; vulnerabilità note (CRC-32 attack,</p>
<p>insertion attack); MAC address Table corrompibile; non usare.</p>
<p>SSH v2 - architettura a tre layer separati; algoritmi moderni; autenticazione</p>
<p>mutua; integrità per messaggio; standard corrente (RFC 4251-4254).</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table></td>
</tr>
</tbody>
</table>

## 1.2 Configurazione SSH su switch Cisco IOS

La configurazione di SSH su un apparato Cisco IOS segue una sequenza precisa. Il nome host e il dominio sono necessari perche Cisco IOS usa la stringa 'hostname.domain' come base per generare il materiale crittografico della chiave RSA.

```
! --- Passo 1: nome host e dominio ---
hostname SW1
ip domain-name lab.local
! --- Passo 2: utente locale con password cifrata ---
username admin privilege 15 secret PasswordSicura
! 'secret' usa MD5/scrypt per cifrare la password nella config.
! Non usare 'password' (testo in chiaro o cifratura debole tipo 7).
! --- Passo 3: generazione coppia di chiavi RSA ---
crypto key generate rsa modulus 2048
! Chiavi da 2048 bit minimo (NIST raccomanda 3072+ per nuove installazioni).
! La generazione abilita automaticamente SSH sul dispositivo.
! --- Passo 4: forzare SSHv2 ---
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
! --- Passo 5: linee VTY - solo SSH, autenticazione locale ---
line vty 0 15
transport input ssh
login local
exec-timeout 10 0
! --- Passo 6: disabilitare Telnet esplicitamente ---
line vty 0 15
transport input ssh
! 'transport input none' disabilita tutto; 'ssh' ammette solo SSH.
```

**Nota su 'privilege 15':** *Il livello di privilegio 15 su Cisco IOS equivale all'accesso completo (come root su Unix). In ambienti di produzione si raccomanda di creare utenti con privilege level inferiori per operazioni di sola lettura, e di richiedere l'autenticazione separata per accedere al Privileged EXEC Mode tramite 'enable secret'.*

## 1.3 Modalita operative Cisco IOS

Cisco IOS utilizza una struttura gerarchica di modalita operative, ciascuna con un livello di privilegio e un prompt caratteristico. La comprensione di questa gerarchia e fondamentale per l'amministrazione sicura degli apparati.

| **Modalita**     | **Prompt**          | **Accesso**                         | **Operazioni permesse**                                                                                          |
| ---------------- | ------------------- | ----------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| User EXEC        | SW1\>               | Accesso iniziale (login)            | Comandi diagnostici limitati: ping, traceroute, show version. Non visualizza la configurazione completa.         |
| Privileged EXEC  | SW1#                | Comando enable (+ password/secret)  | Accesso completo in lettura: show running-config, show ip route, debug. Accesso alle modalita di configurazione. |
| Global Config    | SW1(config)#        | configure terminal                  | Configurazione parametri globali: hostname, routing, VLAN, utenti, servizi.                                      |
| Interface Config | SW1(config-if)#     | interface \<tipo\> \<num\>          | Configurazione di una specifica interfaccia: IP address, duplex, speed, shutdown/no shutdown.                    |
| VLAN Config      | SW1(config-vlan)#   | vlan \<id\>                         | Definizione e naming delle VLAN.                                                                                 |
| Line Config      | SW1(config-line)#   | line vty \<range\> / line console 0 | Configurazione linee di accesso: baud rate, timeout, transport input.                                            |
| Router Config    | SW1(config-router)# | router \<protocollo\>               | Configurazione protocolli di routing (RIP, OSPF, EIGRP, BGP).                                                    |

Per uscire da una modalità si usa 'exit' (torna al livello precedente) o 'end' / Ctrl+Z (torna direttamente al Privileged EXEC Mode). Il comando 'do' permette di eseguire comandi Privileged EXEC dall'interno della modalità di configurazione senza uscire (es. 'do show ip interface brief').

## 1.4 Switch Layer 2 e Layer 3

La distinzione tra switch Layer 2 e Layer 3 definisce le capacita fondamentali dell'apparato e il suo posizionamento nell'architettura di rete.

| **Caratteristica** | **Switch Layer 2** | **Switch Layer 3** |
|----|----|----|
| Livello OSI | Livello 2 (Data Link) | Livello 2 + Livello 3 (Network) |
| Forwarding basato su | MAC Address (CAM table) | MAC Address + IP Address (routing table) |
| Funzione primaria | Switching Ethernet intra-VLAN | Switching Ethernet + routing inter-VLAN e IP |
| Instradamento IP | No (richiede router esterno) | Si (hardware-assisted routing, ASIC) |
| Inter-VLAN routing | Non nativo (richiede router-on-a-stick) | Si, tramite Switched Virtual Interface (SVI) |
| Posizione tipica in rete | Accesso (access layer) | Distribuzione e core |
| Esempi Cisco | Catalyst 2960, 2960X | Catalyst 3750, 9300, 9400 |

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Approfondimento — Switched Virtual Interface (SVI) e inter-VLAN routing</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p>Un Layer 3 switch può eseguire il routing tra VLAN diverse senza un router esterno, tramite interfacce logiche chiamate SVI (Switched Virtual Interface).</p>
<p>Ogni SVI è un'interfaccia IP virtuale associata a una VLAN. Il traffico tra VLAN viene inoltrato a livello 3 dall'ASIC (Application-Specific Integrated Circuit) dello switch, con prestazioni molto superiori al 'router-on-a-stick' (dove tutto il traffico inter-VLAN passa su un unico link fisico verso un router).</p>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<thead>
<tr>
<th><p>! Creazione SVI per VLAN 10 (rete utenti)</p>
<p>interface Vlan10</p>
<p>ip address 10.10.10.1 255.255.255.0</p>
<p>no shutdown</p>
<p>! Creazione SVI per VLAN 20 (rete server)</p>
<p>interface Vlan20</p>
<p>ip address 10.10.20.1 255.255.255.0</p>
<p>no shutdown</p>
<p>! Abilitare il routing IP globalmente</p>
<p>ip routing</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
<p>Con questa configurazione, un host in VLAN 10 (es. 10.10.10.100) puo raggiungere un host in VLAN 20 (es. 10.10.20.50) tramite il Layer 3 switch, che funge da default gateway per entrambe le VLAN.</p></td>
</tr>
</tbody>
</table>

## 1.5 Architettura modulare degli switch enterprise

Gli switch enterprise di fascia alta (es. Cisco Catalyst 9400, 9500 o i moduli Nexus) adottano un'architettura modulare che offre alta disponibilità, flessibilità di configurazione e capacità di upgrade senza interruzione del servizio.

Componenti tipici:

- Supervisor / Control Engine: la CPU principale che gestisce il piano di controllo (routing, STP, gestione VLAN). Spesso ridondato (active/standby).

- Line card / blade: moduli rimovibili a caldo (hot-swappable) che contengono gruppi di porte fisiche. Possono essere sostituiti o aggiunti senza spegnere lo chassis.

- Fabric / backplane: il bus ad alta velocità che interconnette supervisor e line card. Velocità tipiche da centinaia di Gbps a Tbps.

- Alimentatori ridondati (N+1 o N+N): più alimentatori permettono la continuità anche in caso di guasto parziale.

- Ventole ridondate: spesso organizzate con flusso d'aria front-to-back o back-to-front standardizzato per la gestione termica del data center.

La nomenclatura delle interfacce negli switch modulari segue il formato 'slot/modulo/porta' (es. GigabitEthernet 1/0/24): il primo numero indica lo slot del chassis, il secondo il blade/modulo in quello slot, il terzo la porta sul modulo. Questo schema permette di identificare univocamente qualsiasi porta in uno chassis complesso.

# 2. Dynamic Host Configuration Protocol (DHCP)

DHCP (Dynamic Host Configuration Protocol) è un protocollo applicativo definito in RFC 2131 (1997), basato su UDP, che automatizza l'assegnazione dei parametri di configurazione IP ai dispositivi di rete. Sostituisce il predecessore BOOTP (Bootstrap Protocol, RFC 951).

Parametri distribuiti da un server DHCP:

- Indirizzo IP e subnet mask: parametri fondamentali per l'indirizzamento Layer 3.

- Default gateway: l'indirizzo del router a cui il client invia il traffico destinato a subnet diverse.

- DNS server (opzione 6): uno o più indirizzi di server DNS per la risoluzione dei nomi.

- Lease time (opzione 51): la durata in secondi di validità dell'indirizzo IP.

- Domain name (opzione 15): il suffisso DNS del dominio locale.

- NTP server (opzione 42): server di sincronizzazione dell'ora.

- TFTP server (opzione 66): usato per il provisioning zero-touch di dispositivi di rete.

- Altri parametri vendor-specific: tramite opzioni DHCP numerate (1-254).

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Formato del messaggio DHCP (RFC 2131)</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p>Un messaggio DHCP è un singolo datagramma UDP. Il formato del payload e derivato direttamente da BOOTP:</p>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<thead>
<tr>
<th><p>0 1 2 3</p>
<p>0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| op (1) | htype (1) | hlen (1) | hops (1) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| xid (4 byte - transaction ID) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| secs (2) | flags (2) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| ciaddr (4) - Client IP address (se gia assegnato) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| yiaddr (4) - 'Your' IP address (offerto dal server) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| siaddr (4) - Server IP address |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| giaddr (4) - Relay agent IP address (gateway) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| chaddr (16) - Client hardware address (MAC + padding) |</p>
<p>| |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| sname (64) - Server host name (opzionale) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| file (128) - Boot file name (opzionale) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| options (variabile) - Opzioni DHCP (min 312 byte per RFC 2131)|</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
<p>Campi principali:</p>
<ul>
<li><p>op: 1 = BOOTREQUEST (client-&gt;server), 2 = BOOTREPLY (server-&gt;client).</p></li>
<li><p>htype / hlen: tipo e lunghezza dell'hardware address (1 / 6 per Ethernet/MAC).</p></li>
<li><p>hops: numero di relay agent attraversati. Il relay incrementa questo campo ad ogni hop.</p></li>
<li><p>xid: Transaction ID a 32 bit, generato casualmente dal client per correlare request e reply.</p></li>
<li><p>flags: bit 0 (Broadcast flag) = 1 indica al server di rispondere in broadcast.</p></li>
<li><p>ciaddr: IP corrente del client (0.0.0.0 in Discover se non ha ancora un IP).</p></li>
<li><p>yiaddr: IP offerto al client dal server (popolato in Offer e ACK).</p></li>
<li><p>giaddr: IP del relay agent (0.0.0.0 in assenza di relay). Il server usa questo campo per selezionare il pool corretto.</p></li>
<li><p>chaddr: MAC address del client (usato dal server per identificare il client e creare la lease).</p></li>
<li><p>options: campo a lunghezza variabile. Inizia con il magic cookie 0x63825363. Le opzioni sono in formato TLV (Type-Length-Value). L'opzione 53 indica il tipo di messaggio DHCP (1=Discover, 2=Offer, 3=Request, 5=ACK, 6=NAK, 7=Release, 8=Inform).</p></li>
</ul></td>
</tr>
</tbody>
</table>

## 2.1 Il processo DHCP

L'assegnazione di un indirizzo IP tramite DHCP avviene attraverso uno scambio di quattro messaggi, acronimo DORA (Discover, Offer, Request, Acknowledge). Ogni messaggio è un datagramma UDP.

```
Client Server
| |
|--- DHCP Discover (broadcast UDP) -----------> |
| src IP: 0.0.0.0:68 |
| dst IP: 255.255.255.255:67 |
| src MAC: <client MAC> |
| dst MAC: FF:FF:FF:FF:FF:FF |
| chaddr: <client MAC> |
| xid: 0x3903F326 (random) |
| |
| <-- DHCP Offer (unicast o broadcast) -------- |
| src IP: 192.168.0.254:67 |
| dst IP: 255.255.255.255:68 (se bcast flag) |
| yiaddr: 192.168.0.10 (IP offerto) |
| options: subnet mask, gateway, DNS, lease |
| xid: 0x3903F326 (stesso del Discover) |
| |
|--- DHCP Request (broadcast UDP) -----------> |
| Accetta l'offerta; broadcast per avvisare |
| eventuali altri server che non e' scelti. |
| option 50: Requested IP = 192.168.0.10 |
| option 54: Server Identifier = 192.168.0.254|
| |
| <-- DHCP ACK (unicast o broadcast) ---------- |
| yiaddr: 192.168.0.10 confermato |
| options: tutti i parametri finali |
| option 51: lease time = 86400 sec (24h) |
| |
[Il client verifica l'IP con Gratuitous ARP] |
[Se nessuno risponde, il client usa l'indirizzo]|
```

Dettagli operativi:

- Il DHCP Discover è inviato in broadcast a livello IP (255.255.255.255) e a livello Ethernet (FF:FF:FF:FF:FF:FF) poiché il client non ha ancora un IP e non conosce l'indirizzo del server DHCP.

- Il DHCP Request è ancora in broadcast (non unicast) per avvisare tutti i server DHCP presenti nella subnet che solo uno è stato selezionato, permettendo agli altri di liberare l'offerta proposta.

- Il campo xid (Transaction ID) permette al client di correlare le risposte del server alle proprie richieste, soprattutto in presenza di piu server.

- Lease renewal: a T/2 (meta del lease time) il client invia un DHCP Request unicast direttamente al server per rinnovare il lease. Se non riceve risposta, a T\*7/8 riprova in broadcast. Se il lease scade senza rinnovo, il client perde l'IP e ricomincia da Discover.

- DHCP Release: quando il client si disconnette ordinatamente, puo inviare un DHCP Release per liberare anticipatamente l'indirizzo. Non è obbligatorio.

- DHCP Inform: usato da un host che ha già un IP (configurato manualmente) per richiedere solo i parametri aggiuntivi (DNS, dominio) senza richiedere un indirizzo.

## 2.2 Configurazione DHCP server su router/switch Cisco

```
! --- Escludere gli indirizzi riservati (router, server, stampanti) ---
ip dhcp excluded-address 192.168.0.1 192.168.0.127
! --- Definire il pool DHCP ---
ip dhcp pool LAN_UTENTI
network 192.168.0.0 255.255.255.0
default-router 192.168.0.1
dns-server 8.8.8.8 8.8.4.4
domain-name lab.local
lease 1 0 0 ! 1 giorno, 0 ore, 0 minuti
! --- Verifica operativa ---
show ip dhcp binding ! lease attive: MAC, IP, scadenza
show ip dhcp pool ! statistiche pool: totale/usati/disponibili
show ip dhcp conflict ! IP per cui e stato rilevato un conflitto
show ip dhcp server statistics ! contatori messaggi D/O/R/A
debug ip dhcp server events ! debug real-time (usare con cautela)
```

## 2.3 DHCP Relay Agent

I messaggi DHCP Discover sono broadcast Layer 3 (IP 255.255.255.255) e non vengono inoltrati dai router: per definizione, un router non propaga i broadcast IP tra subnet diverse. In ambienti enterprise con piu VLAN o subnet, installare un server DHCP per ogni subnet sarebbe impraticabile. Il DHCP Relay Agent (o IP Helper) risolve questo problema.

Funzionamento del relay:

8.  Il client invia un DHCP Discover in broadcast nella propria subnet.

9.  Il relay agent (tipicamente il router o il Layer 3 switch che ha l'interfaccia nella subnet del client) intercetta il broadcast.

10. Il relay agent converte il broadcast in un messaggio unicast verso l'IP del server DHCP centralizzato, inserendo il proprio IP nell'indirizzo IP sorgente e nel campo giaddr del messaggio DHCP.

11. Il server DHCP usa il campo giaddr per selezionare il pool DHCP corretto (quello configurato per quella subnet).

12. Il server risponde con un DHCP Offer unicast verso l'IP del relay agent.

13. Il relay agent ritrasmette l'Offer al client (in broadcast se il broadcast flag è impostato, altrimenti unicast all'IP del client se gia noto).

```
! Configurazione del relay agent sull'interfaccia rivolta verso i client:
interface GigabitEthernet 0/1
ip address 192.168.10.1 255.255.255.0
ip helper-address 10.0.0.254 ! IP del server DHCP centralizzato
no shutdown
! Il comando 'ip helper-address' per default invia in forward anche:
! - TFTP (UDP 69), DNS (UDP 53), Time (UDP 37), NetBIOS (UDP 137/138)
! Per limitare al solo DHCP:
! no ip forward-protocol udp 69
! no ip forward-protocol udp 53
! (e altri servizi non necessari)
```

## 2.4 Sicurezza DHCP: attacchi e contromisure

DHCP (RFC 2131) non prevede meccanismi di autenticazione nativa. Qualsiasi dispositivo nella rete può rispondere a una richiesta DHCP o inviare richieste massicce. Questo introduce due categorie principali di attacco.

### DHCP Starvation Attack

Un attaccante invia un numero elevato di messaggi DHCP Discover con MAC address sorgente casuali (generati via software, bypassando il MAC reale della scheda). Ogni Discover provoca l'allocazione di un IP nel pool del server. Quando il pool è esaurito, i client legittimi che inviano un Discover non ricevono risposta e rimangono senza IP.

Strumenti usati in penetration testing: Yersinia, dhcpstarv (disponibili in Kali Linux). *Queste attività sono lecite solo in ambienti autorizzati.*

### Rogue DHCP Server

Un attaccante introduce nella rete un server DHCP non autorizzato. I client che ricevono prima la DHCP Offer del server malevolo accettano la configurazione compromessa, che può includere:

- Default gateway malevolo: tutto il traffico del client viene inoltrato attraverso un host controllato dall'attaccante (man-in-the-middle totale).

- DNS server malevolo: le risoluzioni DNS vengono manipolate per reindirizzare il client verso siti di phishing.

- Lease time brevissimo: il server malevolo assegna lease da pochi secondi per mantenere il controllo anche dopo un rinnovo.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Contromisura — DHCP Snooping (IEEE 802.1D estensione / RFC vendor)</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p>DHCP Snooping è una funzionalità di sicurezza implementata sullo switch che classifica le porte in due categorie:</p>
<ul>
<li><p>Trusted (fidate): le porte verso cui è collegato un server DHCP legittimo o un altro switch trusted (uplink). Su queste porte lo switch accetta e invia tutti i messaggi DHCP (incluse Offer e ACK dal server).</p></li>
<li><p>Untrusted (non fidate): le porte verso cui sono collegati i client. Su queste porte lo switch scarta i messaggi DHCP che provengono dal server (Offer, ACK, NAK): se un client sulla porta untrusted tenta di agire da server DHCP (rogue DHCP), il messaggio viene bloccato.</p></li>
</ul>
<p>Configurazione Cisco:</p>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<thead>
<tr>
<th><p>! Abilitare DHCP Snooping globalmente</p>
<p>ip dhcp snooping</p>
<p>ip dhcp snooping vlan 10,20,30</p>
<p>! Configurare le porte trusted (uplink verso server o switch core)</p>
<p>interface GigabitEthernet 0/24</p>
<p>ip dhcp snooping trust</p>
<p>! Le porte di accesso sono untrusted per default</p>
<p>! Opzionale: limitare la frequenza di richieste per porta</p>
<p>interface range GigabitEthernet 0/1 - 23</p>
<p>ip dhcp snooping limit rate 15</p>
<p>! max 15 pacchetti DHCP/secondo; oltre -&gt; port goes to err-disabled</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
<p>DHCP Snooping costruisce e mantiene una tabella di binding (DHCP Snooping Binding Table): per ogni lease attiva memorizza MAC address, IP assegnato, VLAN, porta e lease time. Questa tabella viene usata da Dynamic ARP Inspection (DAI) e IP Source Guard come database di riferimento per validare il traffico ARP e IP.</p>
<p>Altre contromisure complementari:</p>
<ul>
<li><p>Port Security: limita il numero di MAC address validi su una porta; blocca l'attacco starvation se il MAC viene verificato (ma un attaccante puo usare spoofing MAC).</p></li>
<li><p>802.1X (Network Access Control): richiede autenticazione del dispositivo prima di ammettere traffico; soluzione piu robusta per reti enterprise.</p></li>
<li><p>Segmentazione VLAN: riduce il dominio di broadcast e l'impatto di un attacco DHCP.</p></li>
<li><p>IP Source Guard: blocca il traffico IP da client che non corrispondono a una entry nella DHCP Snooping Binding Table.</p></li>
<li><p>Dynamic ARP Inspection (DAI): verifica le richieste ARP contro la binding table; blocca ARP spoofing.</p></li>
</ul></td>
</tr>
</tbody>
</table>

# 3. ARP — Address Resolution Protocol

ARP (Address Resolution Protocol, RFC 826, 1982) e il protocollo che risolve il mapping tra indirizzi Layer 3 (IP) e indirizzi Layer 2 (MAC) all'interno di un segmento di rete locale. Opera esclusivamente all'interno dello stesso dominio di broadcast (stessa subnet / stessa VLAN).

Problema che ARP risolve: quando un host vuole inviare un pacchetto IP a un altro host nella stessa subnet, conosce l'IP di destinazione ma non il MAC address necessario per costruire il frame Ethernet. ARP fornisce il meccanismo per scoprire il MAC a partire dall'IP.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Formato del messaggio ARP (RFC 826)</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p>Il messaggio ARP è incapsulato direttamente in un frame Ethernet con EtherType 0x0806. Non usa IP come trasporto.</p>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<thead>
<tr>
<th><p>0 1 2 3</p>
<p>0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Hardware Type (HTYPE, 2 byte) | Protocol Type (PTYPE, 2 byte) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| HLEN (1) | PLEN (1) | Operation (OPER, 2 byte) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Sender Hardware Address (SHA) - 6 byte per Ethernet |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Sender Protocol Address (SPA) - 4 byte per IPv4 |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Target Hardware Address (THA) - 6 byte (00:00:00:00:00:00 |</p>
<p>| in ARP Request) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Target Protocol Address (TPA) - 4 byte per IPv4 |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
<p>Campi:</p>
<ul>
<li><p>HTYPE: tipo di hardware. 1 = Ethernet.</p></li>
<li><p>PTYPE: tipo di protocollo. 0x0800 = IPv4.</p></li>
<li><p>HLEN / PLEN: lunghezza hardware/protocol address. 6 / 4 per Ethernet/IPv4.</p></li>
<li><p>OPER: 1 = ARP Request, 2 = ARP Reply. (RARP usava 3 e 4, ormai obsoleto.)</p></li>
<li><p>SHA: MAC del mittente. Sempre popolato.</p></li>
<li><p>SPA: IP del mittente. Sempre popolato.</p></li>
<li><p>THA: MAC del destinatario cercato. 00:00:00:00:00:00 nell'ARP Request (sconosciuto); popolato nell'ARP Reply.</p></li>
<li><p>TPA: IP del destinatario cercato. Sempre popolato.</p></li>
</ul></td>
</tr>
</tbody>
</table>

## 3.1 Funzionamento di ARP

Il processo ARP per risolvere l'IP 192.168.1.20 da un host con IP 192.168.1.10:

14. L'host controlla la propria ARP cache: se esiste già una entry per 192.168.1.20 valida (non scaduta), usa direttamente il MAC associato.

15. Se la entry non esiste o e scaduta, invia un ARP Request in broadcast Ethernet (dst MAC FF:FF:FF:FF:FF:FF) con TPA = 192.168.1.20 e THA = 00:00:00:00:00:00.

16. Tutti i dispositivi nella stessa subnet ricevono l'ARP Request. Solo quello con IP = 192.168.1.20 risponde con un ARP Reply unicast (dst MAC = SHA del richiedente) contenente il proprio MAC nel campo THA e SHA.

17. Il richiedente aggiorna la propria ARP cache con la nuova entry IP-\>MAC e usa il MAC per costruire il frame Ethernet.

La ARP cache ha un timeout (tipicamente 4 minuti su Windows, 20 minuti su Cisco IOS, variabile su Linux). Dopo la scadenza, la entry viene rimossa e un nuovo ARP Request viene inviato alla successiva necessita.

## 3.2 Gratuitous ARP e conflitti IP

Un Gratuitous ARP (GARP) e un ARP Request speciale in cui il mittente imposta sia SPA sia TPA al proprio indirizzo IP, e THA a FF:FF:FF:FF:FF:FF. Non e una domanda ma un annuncio.

Usi del Gratuitous ARP:

- Rilevamento conflitti IP: se un altro host nella rete ha già quell'IP, risponderà al GARP con il proprio MAC. Il mittente rileva il conflitto.

- Aggiornamento cache: notifica tutti gli host nella subnet del nuovo MAC associato a quell'IP (utile dopo un failover in cluster attivo-passivo, o dopo una sostituzione hardware).

- Verifica DHCP: come descritto, il client DHCP invia un GARP dopo aver ricevuto un indirizzo per verificare che non sia già in uso prima di accettarlo (RFC 2131).

## 3.3 ARP Cache vs MAC Address Table

| **Elemento** | **Dove risiede** | **Contenuto** | **Funzione** | **Timeout tipico** | **Comando Cisco** |
|----|----|----|----|----|----|
| ARP Cache | Host, router, dispositivi L3 | IP address \<-\> MAC address | Risolve IP-\>MAC per costruire frame Ethernet. Consultata prima di inviare ogni pacchetto IP su LAN. | 4-20 minuti (OS-dipendente) | show ip arp (Cisco IOS) arp -a (Windows/Linux) |
| MAC Address Table (CAM Table) | Switch (Layer 2 e L3) | MAC address \<-\> Porta fisica | Permette allo switch di inoltrare frame Ethernet solo sulla porta corretta (unicast forwarding). | 300 sec (5 minuti, default Cisco) | show mac address-table |

**Comportamento di flooding:** *Quando uno switch riceve un frame unicast con MAC destinazione non presente nella MAC address table, lo inonda (flood) su tutte le porte della VLAN tranne quella di ingresso. Questo comportamento è normale per i MAC non ancora appresi, ma viene sfruttato nell'attacco MAC flooding (tramite strumenti come macof) per saturare la CAM table e forzare lo switch a comportarsi come un hub, permettendo la cattura passiva del traffico unicast altrui.*

# 4. Router e routing IP

Un router è un dispositivo di livello 3 che interconnette subnet IP diverse, instradando i pacchetti dalla rete sorgente alla rete destinataria. Ogni interfaccia fisica o logica del router appartiene a una subnet diversa e possiede un indirizzo IP in quella subnet.

## 4.1 Processo di forwarding del router

Il processo con cui un router elabora e inoltra un pacchetto IP:

18. Ricezione del frame Ethernet sull'interfaccia di ingresso.

19. Verifica del CRC del frame: se errato, il frame viene scartato silenziosamente.

20. Verifica del MAC address destinazione: deve corrispondere al MAC dell'interfaccia del router (o essere broadcast). Se non corrisponde e non e broadcast, il frame viene scartato.

21. Decapsulamento Layer 2: il router rimuove l'header e il trailer Ethernet, estraendo il pacchetto IP.

22. Lettura dell'IP destinazione nell'header IP.

23. Decremento del TTL (Time To Live): se TTL = 0 dopo il decremento, il pacchetto viene scartato e viene inviato un messaggio ICMP Time Exceeded al mittente originale.

24. Consultazione della routing table con Longest Prefix Match (LPM): il router cerca la rotta con il prefisso piu lungo che corrisponde all'IP destinazione.

25. Determinazione del next-hop e dell'interfaccia di uscita.

26. Risoluzione ARP del next-hop (se non gia in ARP cache): il router invia un ARP Request per ottenere il MAC del next-hop.

27. Riencapsulamento in un nuovo frame Ethernet con: MAC src = MAC dell'interfaccia di uscita del router, MAC dst = MAC del next-hop (o del destinatario finale se nella stessa subnet).

28. Trasmissione del nuovo frame sull'interfaccia di uscita.

**IP address invariato:** *Gli indirizzi IP sorgente e destinazione del pacchetto rimangono invariati per tutta la durata del percorso end-to-end (eccetto in caso di NAT). I MAC address cambiano a ogni hop Layer 3 perche ogni router sostituisce il frame Ethernet con uno nuovo per il link successivo.*

## 4.2 La routing table e il Longest Prefix Match

La routing table è la struttura dati centrale del router che associa prefissi di rete a informazioni di forwarding (next-hop, interfaccia di uscita, metrica). Quando un pacchetto arriva, il router cerca il prefisso piu specifico (longest prefix match, LPM) che corrisponde all'IP destinazione.

```
Router# show ip route
Codes: C - connected, S - static, R - RIP, O - OSPF, B - BGP, D - EIGRP
* - candidate default
Gateway of last resort is 10.0.0.1 to network 0.0.0.0
C 192.168.0.0/24 is directly connected, GigabitEthernet0/0
C 192.168.1.0/24 is directly connected, GigabitEthernet0/1
S 10.0.0.0/8 [1/0] via 192.168.0.254
O 172.16.0.0/16 [110/20] via 192.168.1.254, 00:01:05, GigabitEthernet0/1
S* 0.0.0.0/0 [1/0] via 10.0.0.1 ! default route
```

Esempio di LPM: un pacchetto destinato a 10.10.5.1 corrisponde sia a '10.0.0.0/8' (via 192.168.0.254) sia a '0.0.0.0/0' (default). Il router usa 10.0.0.0/8 perché è il prefisso piu lungo (8 bit \> 0 bit). Un pacchetto destinato a 172.20.1.1 corrisponde solo a 0.0.0.0/0 e viene inviato al gateway di default.

La notazione '\[1/0\]' indica \[administrative distance / metric\]: AD 1 corrisponde alle rotte statiche (massima fiducia dopo le rotte connected); la metrica 0 indica costo zero. Per OSPF '\[110/20\]': AD 110 (valore default OSPF), metrica 20 (somma dei costi OSPF dei link sul percorso).

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Approfondimento — Administrative Distance (AD)</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p>Quando un router apprende la stessa rete da piu fonti (es. una rotta statica e una rotta OSPF per lo stesso prefisso), usa la Administrative Distance per scegliere la rotta preferita. AD piu bassa = maggiore fiducia.</p>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<thead>
<tr>
<th><p>Sorgente della rotta AD (Cisco IOS default)</p>
<p>-----------------------------------------------------</p>
<p>Connected (direttamente conn.) 0 (non modificabile)</p>
<p>Static route 1</p>
<p>EIGRP (summary) 5</p>
<p>BGP (eBGP) 20</p>
<p>EIGRP (internal) 90</p>
<p>OSPF 110</p>
<p>IS-IS 115</p>
<p>RIP 120</p>
<p>EIGRP (external) 170</p>
<p>BGP (iBGP) 200</p>
<p>Unknown / non fidato 255 (mai usato per forwarding)</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
<p>Una rotta con AD 255 non viene mai installata nella routing table attiva: e un meccanismo di sicurezza per disabilitare rotte non affidabili. I valori di AD sono locali al router e non vengono annunciati ai vicini nei protocolli di routing.</p></td>
</tr>
</tbody>
</table>

## 4.3 Routing statico

Nel routing statico l'amministratore configura manualmente ogni rotta nella routing table. Il router non apprende dinamicamente cambiamenti di topologia.

```
! Rotta statica verso la rete 10.0.0.0/8 via il next-hop 192.168.0.254
ip route 10.0.0.0 255.0.0.0 192.168.0.254
! Rotta statica usando l'interfaccia di uscita (invece del next-hop IP)
ip route 10.0.0.0 255.0.0.0 GigabitEthernet0/0
! Default route (0.0.0.0/0): usata per tutto il traffico senza rotta specifica
ip route 0.0.0.0 0.0.0.0 10.0.0.1
! Rotta statica floating (AD=200): backup da usare solo se la rotta OSPF cade
ip route 10.0.0.0 255.0.0.0 192.168.1.254 200
```

| **Aspetto** | **Routing Statico** | **Routing Dinamico** |
|----|----|----|
| Configurazione | Manuale su ogni router | Automatica tramite protocollo |
| Adattamento ai guasti | Non si adatta (rotta rimane in tabella) | Converge automaticamente su percorso alternativo |
| Scalabilità | Bassa: O(n^2) configurazioni per n reti | Alta: il protocollo propaga le informazioni |
| Overhead CPU/banda | Nessuno | Calcolo algoritmo + traffico di controllo |
| Prevedibilità | Totale | Dipende dall'algoritmo e dai parametri |
| Sicurezza | Nessun rischio di route injection | Rischio di route injection se non autenticato |
| Uso tipico | Reti piccole, rotte di default, lab | Reti enterprise, ISP, Internet (BGP) |

## 4.4 Protocolli di routing dinamico

I protocolli di routing dinamico permettono ai router di scambiarsi informazioni sulla topologia di rete e di aggiornare automaticamente le routing table in risposta ai cambiamenti. Si classificano in due famiglie principali.

### Distance Vector

I protocolli Distance Vector (Bellman-Ford algorithm) distribuiscono la conoscenza della rete annunciando periodicamente a ogni vicino diretto la propria routing table completa ('routing by rumor'). Ogni router conosce la distanza (metrica) e la direzione (next-hop) verso ogni destinazione, ma non la topologia completa.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Approfondimento — RIP: Routing Information Protocol</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p>RIP (RFC 1058 per RIPv1; RFC 2453 per RIPv2) e il protocollo Distance Vector piu semplice e storico. Opera a livello applicativo su UDP porta 520.</p>
<p>Caratteristiche:</p>
<ul>
<li><p>Metrica: hop count (numero di router attraversati). Massimo 15 hop; 16 = infinito (rete irraggiungibile). Questo limita RIP a reti piccole.</p></li>
<li><p>Update periodici: ogni 30 secondi ogni router invia l'intera routing table ai propri vicini diretti.</p></li>
<li><p>RIPv1: annunci in broadcast (255.255.255.255); classful (non include la subnet mask negli annunci). Incompatibile con VLSM.</p></li>
<li><p>RIPv2: annunci in multicast (224.0.0.9); classless (include la subnet mask negli annunci); supporta autenticazione MD5.</p></li>
<li><p>Timers: Update (30s), Invalid (180s), Holddown (180s), Flush (240s).</p></li>
<li><p>Problemi: convergenza lenta (counting to infinity); inefficiente per reti grandi; non considera la larghezza di banda dei link.</p></li>
</ul>
<p>Formato del messaggio RIP (payload UDP):</p>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<thead>
<tr>
<th><p>0 1 2 3</p>
<p>0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Command (1) | Version (1) | Zero (2, must be zero) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Route Entry 1: Address Family ID (2) | Route Tag (2) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| IP Address (4 byte) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Subnet Mask (4 byte) [solo RIPv2] |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Next Hop (4 byte) [solo RIPv2, 0.0.0.0 = usa il router vicino]|</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Metric (4 byte, valore 1-16) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| ... fino a 25 route entry per messaggio (max UDP 512 byte) ...|</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
<p>Oggi RIP e quasi completamente sostituito da OSPF e EIGRP nelle reti enterprise, ma rimane rilevante a scopo didattico per la sua semplicita concettuale.</p></td>
</tr>
</tbody>
</table>

### Link State

I protocolli Link State (Dijkstra / Shortest Path First) utilizzano un approccio radicalmente diverso: ogni router raccoglie informazioni sullo stato di ogni link nella rete e costruisce una mappa topologica completa (LSDB - Link State Database). Su questa mappa esegue l'algoritmo SPF (Shortest Path First) per calcolare i percorsi piu brevi verso ogni destinazione.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Approfondimento — OSPF: Open Shortest Path First</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p>OSPF (RFC 2328 per OSPFv2/IPv4; RFC 5340 per OSPFv3/IPv6) e il protocollo Link State piu diffuso nelle reti enterprise. Opera direttamente su IP con protocollo 89 (non usa TCP o UDP).</p>
<p>Concetti fondamentali:</p>
<ul>
<li><p>LSA (Link State Advertisement): il messaggio con cui ogni router annuncia i propri link, vicini, costi e reti connesse. Esistono diversi tipi di LSA (Type 1: Router LSA, Type 2: Network LSA, Type 3: Summary LSA, Type 5: External LSA ecc.).</p></li>
<li><p>LSDB (Link State Database): il database sincrono che ogni router mantiene, contenente tutti gli LSA ricevuti. Tutti i router nella stessa area OSPF hanno la stessa LSDB.</p></li>
<li><p>SPF (Shortest Path First / Dijkstra): algoritmo che ogni router esegue sulla propria LSDB per costruire l'albero dei percorsi piu brevi verso ogni destinazione.</p></li>
<li><p>Costo OSPF: metrica basata sulla larghezza di banda del link. Costo = 10^8 / bandwidth (bps). Un link da 100 Mbps ha costo 1; un link da 10 Mbps ha costo 10. Piu basso = migliore.</p></li>
<li><p>Aree OSPF: OSPF scala tramite la suddivisione in aree. L'area 0 e il backbone; tutte le altre aree devono connettersi all'area 0. Questo limita la diffusione dei LSA e riduce il carico computazionale dell'SPF.</p></li>
<li><p>DR e BDR: su segmenti multi-access (Ethernet), OSPF elegge un Designated Router (DR) e un Backup DR (BDR) per ridurre il numero di adjacency e di LSA. Tutti i router formano adjacency solo con DR e BDR.</p></li>
</ul>
<p>Tipi di pacchetti OSPF (tutti trasportati su IP/89):</p>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<thead>
<tr>
<th><p>Tipo 1 - Hello: scoperta vicini, mantenimento delle adjacency.</p>
<p>Multicast 224.0.0.5 (AllSPFRouters).</p>
<p>Tipo 2 - DBD (Database scambio degli header LSA durante la sincronizzazione</p>
<p>Description): iniziale del LSDB tra due router.</p>
<p>Tipo 3 - LSR (Link State richiesta di LSA specifici al vicino.</p>
<p>Request):</p>
<p>Tipo 4 - LSU (Link State trasmissione degli LSA richiesti o aggiornamenti.</p>
<p>Update):</p>
<p>Tipo 5 - LSAck: acknowledgement di LSA ricevuti.</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
<p>Formato dell'header OSPF comune (24 byte):</p>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<thead>
<tr>
<th><p>0 1 2 3</p>
<p>0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Version (1) | Type (1) | Packet Length (2) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Router ID (4 byte - scelto come IP piu alto o manuale) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Area ID (4 byte - 0.0.0.0 per backbone) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Checksum (2) | Auth Type (2) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Authentication Data (8 byte) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| [corpo specifico del tipo di pacchetto segue] |</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
<p>Configurazione OSPF su Cisco IOS:</p>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<thead>
<tr>
<th><p>! Abilitare OSPF con process ID locale 1</p>
<p>router ospf 1</p>
<p>router-id 1.1.1.1</p>
<p>! Annuncia la rete 192.168.0.0/24 nell'area 0</p>
<p>network 192.168.0.0 0.0.0.255 area 0</p>
<p>! Annuncia la rete 192.168.1.0/24 nell'area 0</p>
<p>network 192.168.1.0 0.0.0.255 area 0</p>
<p>! Interfaccia passiva: non invia Hello su questa interfaccia</p>
<p>passive-interface GigabitEthernet0/0</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table></td>
</tr>
</tbody>
</table>

| **Caratteristica** | **RIP (Distance Vector)** | **OSPF (Link State)** |
|----|----|----|
| Algoritmo | Bellman-Ford distribuito | Dijkstra SPF (locale su ogni router) |
| Metrica | Hop count (max 15) | Costo basato su bandwidth (Reference: 10^8 bps) |
| Convergenza | Lenta (30s update + counting to infinity) | Rapida (LSA flooding + SPF immediato) |
| Scalabilità | Bassa (max 15 hop) | Alta (aree OSPF, milioni di rotte con OSPF) |
| Aggiornamenti | Periodici (ogni 30s, full table) | Event-driven (solo al cambio topologia), parziali |
| Protocollo trasporto | UDP 520 | IP protocol 89 (direttamente su IP) |
| Autenticazione | MD5 (RIPv2) | MD5 / SHA (OSPFv2); IPsec (OSPFv3) |
| Topologia conosciuta | Parziale (solo distanza e direzione) | Completa (LSDB = mappa intera area) |
| RFC | RFC 2453 (RIPv2) | RFC 2328 (OSPFv2), RFC 5340 (OSPFv3) |

# 5. Comunicazione tra subnet: il ruolo di MAC e IP

La comprensione di come cambiano i MAC address a ogni hop mentre gli IP rimangono invariati è fondamentale per diagnosticare i problemi di rete e capire il funzionamento dei firewall, degli IPS e degli strumenti di analisi del traffico.

```
Scenario: PC-A (192.168.1.10) invia un pacchetto a Server (10.0.0.50)
attraverso Router R1 (GE0/0: 192.168.1.1 / GE0/1: 10.0.0.1)
=== Segmento 1: PC-A -> R1 ===
Frame Ethernet:
src MAC: MAC-PC-A dst MAC: MAC-GE0/0-R1 (gateway)
Pacchetto IP:
src IP: 192.168.1.10 dst IP: 10.0.0.50 (invariato)
R1 riceve il frame, decapsula, consulta la routing table:
10.0.0.0/24 is directly connected, GigabitEthernet0/1
R1 fa ARP per 10.0.0.50 se non e in ARP cache.
=== Segmento 2: R1 -> Server ===
Frame Ethernet (NUOVO frame, costruito da R1):
src MAC: MAC-GE0/1-R1 dst MAC: MAC-Server (nuovo MAC!)
Pacchetto IP:
src IP: 192.168.1.10 dst IP: 10.0.0.50 (INVARIATO)
Conclusione: ad ogni hop L3, il frame Ethernet viene completamente
ricostruito con nuovi MAC address. L'header IP rimane
intatto per tutta la durata del percorso end-to-end.
```

# 6. Importanza della pratica nel networking

Il networking e una disciplina profondamente pratica. La comprensione teorica dei protocolli è necessaria ma non sufficiente per sviluppare competenze operative reali. L'analisi dei pacchetti con strumenti come Wireshark, la configurazione di apparati reali o simulati (Cisco Packet Tracer, GNS3, EVE-NG) e il troubleshooting di problemi concreti sono componenti irrinunciabili della formazione.

In particolare, la cattura e l'analisi del traffico DHCP, ARP e dei protocolli di routing con Wireshark consente di:

- Verificare empiricamente il processo DORA e osservare il formato reale dei messaggi DHCP.

- Osservare il comportamento del gratuitous ARP e rilevare ARP spoofing.

- Analizzare gli hello OSPF e le LSA flooding per comprendere la convergenza.

- Diagnosticare problemi di duplex mismatch, loop di switching, o conflitti IP.

- Familiarizzare con i filtri di cattura e di display di Wireshark per analisi mirate.

# Bibliografia

Le opere sono organizzate per argomento e citate in formato APA 7a edizione.

## Reti informatiche — testi di riferimento

Tanenbaum, A. S., & Wetherall, D. J. (2011). *Computer networks* (5th ed.). Prentice Hall.

Kurose, J. F., & Ross, K. W. (2021). *Computer networking: A top-down approach* (8th ed.). Pearson.

Forouzan, B. A. (2013). *Data communications and networking* (5th ed.). McGraw-Hill.

## SSH

Ylonen, T., & Lonvick, C. (2006). *The Secure Shell (SSH) Protocol Architecture* (RFC 4251). IETF. https://www.rfc-editor.org/rfc/rfc4251

Ylonen, T., & Lonvick, C. (2006). *The Secure Shell (SSH) Transport Layer Protocol* (RFC 4253). IETF. https://www.rfc-editor.org/rfc/rfc4253

Ylonen, T., & Lonvick, C. (2006). *The Secure Shell (SSH) Authentication Protocol* (RFC 4252). IETF. https://www.rfc-editor.org/rfc/rfc4252

Barrett, D. J., Silverman, R. E., & Byrnes, R. G. (2005). *SSH, the Secure Shell: The definitive guide* (2nd ed.). O'Reilly Media.

## DHCP

Droms, R. (1997). *Dynamic Host Configuration Protocol* (RFC 2131). IETF. https://www.rfc-editor.org/rfc/rfc2131

Alexander, S., & Droms, R. (1997). *DHCP options and BOOTP vendor extensions* (RFC 2132). IETF. https://www.rfc-editor.org/rfc/rfc2132

Patrick, M. (2001). *DHCP relay agent information option* (RFC 3046). IETF. https://www.rfc-editor.org/rfc/rfc3046

## ARP

Plummer, D. C. (1982). *An Ethernet address resolution protocol (ARP)* (RFC 826). IETF. https://www.rfc-editor.org/rfc/rfc826

Cheswick, W. R., Bellovin, S. M., & Rubin, A. D. (2003). *Firewalls and Internet security: Repelling the wily hacker* (2nd ed.). Addison-Wesley.

## Routing e protocolli di routing

Hedrick, C. (1988). *Routing Information Protocol* (RFC 1058). IETF. https://www.rfc-editor.org/rfc/rfc1058

Malkin, G. (1998). *RIP version 2* (RFC 2453). IETF. https://www.rfc-editor.org/rfc/rfc2453

Moy, J. (1998). *OSPF version 2* (RFC 2328). IETF. https://www.rfc-editor.org/rfc/rfc2328

Coltun, R., Ferguson, D., Moy, J., & Lindem, A. (2008). *OSPF for IPv6* (RFC 5340). IETF. https://www.rfc-editor.org/rfc/rfc5340

Doyle, J., & Carroll, J. D. (2005). *Routing TCP/IP* (Vol. 1, 2nd ed.). Cisco Press.

Dijkstra, E. W. (1959). A note on two problems in connexion with graphs. *Numerische Mathematik, 1*(1), 269-271. https://doi.org/10.1007/BF01386390

## Sicurezza di rete

Lammle, T. (2019). *CCNA Cisco Certified Network Associate study guide* (8th ed.). Sybex.

Bejtlich, R. (2004). *The Tao of network security monitoring: Beyond intrusion detection*. Addison-Wesley.

Cert, C. (2004). *DHCP server attack — Rogue DHCP* \[Technical note\]. Carnegie Mellon University Software Engineering Institute.

## Configurazione Cisco IOS

Cisco Systems. (2024). *Cisco IOS security configuration guide: Securing user services*. https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_usr_ssh/configuration/xe-17/sec-usr-ssh-xe-17-book.html

Cisco Systems. (2024). *Cisco IOS IP addressing services configuration guide: DHCP*. https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/ipaddr_dhcp/configuration/xe-17/dhcp-xe-17-book.html

Cisco Systems. (2024). *Catalyst switches: DHCP snooping configuration example*. https://www.cisco.com/c/en/us/support/docs/switches/catalyst-6500-series-switches/72846-98.html

**Nota:** *Tutte le RFC citate sono gratuitamente accessibili su https://www.rfc-editor.org e costituiscono la fonte normativa primaria per i protocolli descritti in questa lezione. I riferimenti alle guide di configurazione Cisco rimandano alla documentazione ufficiale, soggetta ad aggiornamenti periodici.*
