**Lezione 6**

*VLAN, Trunking e Routing Inter-VLAN*

# 1. Virtual Local Area Network (VLAN)

Le Virtual Local Area Network (VLAN) rappresentano uno dei meccanismi fondamentali per la segmentazione logica delle reti Ethernet moderne. Il loro scopo principale consiste nel separare gruppi di dispositivi all'interno della stessa infrastruttura fisica, creando domini di broadcast distinti e migliorando sia l'efficienza sia la sicurezza della rete.

In una rete Ethernet tradizionale non segmentata, tutti i dispositivi collegati agli stessi switch condividono il medesimo dominio di broadcast: ogni frame broadcast inviato da un host viene ricevuto da tutti gli altri host nel dominio. In reti di grandi dimensioni, questo comportamento produce congestione, riduzione delle prestazioni e una maggiore superficie di attacco per tecniche quali ARP spoofing e DHCP starvation.

Le VLAN permettono di segmentare logicamente la rete in più domini di broadcast indipendenti, anche all'interno dello stesso switch fisico, senza richiedere hardware aggiuntivo.

## 1.1 Definizione e Caratteristiche Fondamentali

Una VLAN può essere definita come un dominio di broadcast logico costruito sopra un'infrastruttura fisica condivisa. La sua appartenenza è determinata dalla configurazione dello switch, non dalla posizione fisica del dispositivo.

Ogni VLAN:

- costituisce un dominio di broadcast separato e indipendente;

- è normalmente associata a una specifica subnet IP (uno spazio di indirizzamento Layer 3 distinto);

- richiede un meccanismo di routing Layer 3 per comunicare con altre VLAN;

- è identificata da un VLAN ID (VID) compreso tra 1 e 4094 (12 bit), secondo lo standard IEEE 802.1Q.

Esempio di mapping VLAN–Subnet:

| **VLAN**              | **Subnet assegnata** |
|-----------------------|----------------------|
| VLAN 10 (UFFICI)      | 192.168.10.0/24      |
| VLAN 20 (LABORATORIO) | 192.168.20.0/24      |
| VLAN 30 (GESTIONE)    | 192.168.30.0/24      |

I dispositivi appartenenti alla VLAN 10 non possono comunicare direttamente con quelli della VLAN 20: il traffico tra VLAN distinte deve sempre attraversare un dispositivo di routing (router o switch Layer 3).

## 1.2 Vantaggi delle VLAN

### Riduzione del traffico broadcast

Limitando il dominio di broadcast a un insieme ristretto di host, le VLAN riducono drasticamente il traffico broadcast sulla rete. In ambienti con centinaia o migliaia di dispositivi, la riduzione del dominio di broadcast migliora sensibilmente le prestazioni.

### Maggiore sicurezza

La separazione logica impone che il traffico inter-VLAN transiti attraverso un dispositivo di Layer 3, il quale può applicare:

- Access Control List (ACL) per filtrare il traffico tra segmenti;

- policy di sicurezza per limitare l'accesso a risorse critiche (server, database);

- ispezione del traffico tramite firewall o sistemi IDS/IPS.

### Maggiore flessibilità gestionale

Con switch managed è possibile spostare un dispositivo da una VLAN a un'altra semplicemente modificando la configurazione della porta, senza alcun intervento fisico sul cablaggio. Questo semplifica l'organizzazione logica della rete e l'applicazione di Quality of Service (QoS).

|  |
|----|
| **ℹ Nota: Switch Managed vs Unmanaged** |
| Uno switch unmanaged opera in modo completamente automatico (plug-and-play) e non consente configurazioni avanzate: tutte le porte appartengono a un unico dominio di broadcast. |
| Uno switch managed espone invece un'interfaccia di amministrazione (CLI, SNMP, interfaccia web) che consente: configurazione di VLAN e trunk, monitoraggio del traffico, aggregazione di link (Link Aggregation – IEEE 802.3ad/LACP), applicazione di QoS e policy di sicurezza, gestione remota via SSH o Telnet. |
| In ambienti professionali e universitari, gli switch managed costituiscono la soluzione standard. |

# 2. Porte Access e Porte Trunk

## 2.1 Porte Access

Una porta access è associata a una singola VLAN. Il dispositivo collegato a tale porta non è consapevole dell'esistenza delle VLAN: riceve e trasmette frame Ethernet normali, senza alcun tag. È lo switch che associa internamente il traffico alla VLAN configurata sulla porta.

Le porte access vengono tipicamente utilizzate per collegare dispositivi terminali: workstation, stampanti, server, telefoni IP, access point di fascia base.

Configurazione Cisco IOS (esempio):

> interface FastEthernet0/2
>
> switchport mode access
>
> switchport access vlan 20

## 2.2 Porte Trunk

Una porta trunk è in grado di trasportare contemporaneamente il traffico di più VLAN sullo stesso collegamento fisico. Per distinguere i frame appartenenti a VLAN differenti, viene utilizzato il meccanismo di tagging definito dallo standard IEEE 802.1Q (descritto nella sezione seguente).

Le porte trunk vengono utilizzate per interconnettere:

- switch tra loro (uplink inter-switch);

- switch e router (per il routing inter-VLAN);

- switch e firewall;

- switch e access point avanzati (che gestiscono più SSID su VLAN distinte).

Configurazione Cisco IOS (esempio):

> interface GigabitEthernet0/1
>
> switchport mode trunk

# 3. IEEE 802.1Q e VLAN Tagging

Lo standard IEEE 802.1Q, pubblicato originariamente nel 1998 e successivamente revisionato, definisce il meccanismo di VLAN tagging per le reti Ethernet. Quando un frame deve attraversare una porta trunk, lo switch inserisce un campo aggiuntivo di 4 byte (il tag 802.1Q) all'interno del frame Ethernet, immediatamente dopo i campi MAC di destinazione e sorgente, per poter individuare a quale VLAN appartiene il frame.

## 3.1 Struttura del Frame Ethernet con Tag 802.1Q

Il frame Ethernet originale ha la seguente struttura (semplificata):

| **Campo**                  | **Dimensione** |
|----------------------------|----------------|
| Preambolo + SFD            | 8 byte         |
| MAC Destinazione           | 6 byte         |
| MAC Sorgente               | 6 byte         |
| EtherType / Lunghezza      | 2 byte         |
| Payload (dati)             | 46–1500 byte   |
| FCS (Frame Check Sequence) | 4 byte         |

Con l'aggiunta del tag 802.1Q, il frame diventa:

| **Campo**                               | **Dimensione** |
|-----------------------------------------|----------------|
| Preambolo + SFD                         | 8 byte         |
| MAC Destinazione                        | 6 byte         |
| MAC Sorgente                            | 6 byte         |
| Tag Protocol Identifier (TPID) = 0x8100 | 2 byte         |
| Tag Control Information (TCI)           | 2 byte         |
| EtherType / Lunghezza originale         | 2 byte         |
| Payload (dati)                          | 46–1500 byte   |
| FCS (ricalcolato)                       | 4 byte         |

## 3.2 Struttura del Tag Control Information (TCI)

Il campo TCI di 2 byte (16 bit) è così strutturato:

| **Sotto-campo** | **Dimensione** | **Descrizione** |
|----|----|----|
| PCP (Priority Code Point) | 3 bit | Priorità QoS (valori 0–7, usato da IEEE 802.1p) |
| DEI (Drop Eligible Indicator) | 1 bit | Indica se il frame può essere scartato in caso di congestione |
| VID (VLAN Identifier) | 12 bit | Identificatore VLAN (valori validi: 1–4094) |

Il campo VID a 12 bit consente di definire teoricamente 4096 VLAN (0–4095). I valori 0 e 4095 sono riservati; i valori utilizzabili vanno da 1 a 4094.

Nota importante: l'aggiunta del tag 802.1Q aumenta la dimensione massima del frame Ethernet da 1518 byte a 1522 byte (escludendo preambolo e SFD). Questo richiede che i dispositivi di rete supportino i cosiddetti "baby giant frames" per gestire correttamente i frame taggati senza errori di dimensione.

## 3.3 Operazioni di Tagging e Untagging

Il processo di tagging/untagging avviene come segue:

1.  Un host invia un frame normale (senza tag) sulla porta access.

2.  Lo switch riceve il frame sulla porta access e determina la VLAN di appartenenza dalla configurazione della porta.

3.  Lo switch aggiunge il tag 802.1Q con il VID corrispondente e instrada il frame verso la porta trunk.

4.  Lo switch di destinazione riceve il frame taggato sulla porta trunk, legge il VID e rimuove il tag.

5.  Lo switch consegna il frame (senza tag) all'host di destinazione tramite la porta access della VLAN corretta.

# 4. VLAN Native

La VLAN nativa è la VLAN per la quale i frame vengono trasmessi su un trunk senza aggiunta del tag 802.1Q. Per ragioni di compatibilità storica con dispositivi che non supportano il tagging, lo standard 802.1Q prevede che una VLAN (e una sola) possa essere configurata come "nativa, *native* in inglese" su ciascun trunk.

Per impostazione predefinita su switch Cisco, la VLAN native è la VLAN 1. Questo comportamento introduce tuttavia rischi di sicurezza, sfruttati dalla tecnica di VLAN hopping descritta nella sezione seguente.

Best practice: è fortemente raccomandato modificare la VLAN native rispetto al valore di default (VLAN 1) e utilizzare una VLAN dedicata, non associata ad alcun host, per ridurre la superficie di attacco.

> interface GigabitEthernet0/1
>
> switchport trunk native vlan 99

|  |
|----|
| **ℹ Nota: VLAN 1 – Rischi e Best Practice** |
| La VLAN 1 è la VLAN di default su tutti gli switch Cisco e su molti altri vendor. Su di essa transitano, per default, protocolli di controllo come CDP (Cisco Discovery Protocol), VTP (VLAN Trunking Protocol), STP (Spanning Tree Protocol) e PAgP. |
| Per ridurre il rischio, è buona pratica: (1) non assegnare host alla VLAN 1; (2) cambiare la VLAN native dei trunk; (3) disabilitare le porte inutilizzate e assegnarle a una VLAN di quarantena. |

# 5. VLAN Hopping

Il VLAN hopping è una categoria di attacchi che mira a far transitare traffico tra VLAN differenti bypassando i controlli di sicurezza, senza passare attraverso un router o firewall.

Le tecniche principali sono due:

## 5.1 Switch Spoofing

Un attaccante configura la propria scheda di rete per negoziare un trunk con lo switch (sfruttando DTP – Dynamic Trunking Protocol, un protocollo Cisco proprietario). Se la porta dello switch è configurata in modalità "auto" o "desirable", accetterà la negoziazione del trunk, consentendo all'attaccante di ricevere traffico da tutte le VLAN.

Contromisura: disabilitare DTP sulle porte access con il comando:

> interface FastEthernet0/X
>
> switchport mode access
>
> switchport nonegotiate

## 5.2 Double Tagging (Doppio Tagging 802.1Q)

L'attaccante costruisce un frame con due tag 802.1Q sovrapposti. Il tag esterno corrisponde alla VLAN native del trunk (tipicamente VLAN 1, non taggata); il tag interno corrisponde alla VLAN bersaglio.

Il primo switch rimuove il tag esterno (VLAN native, non taggata) e invia il frame al secondo switch, che lo instrada nella VLAN indicata dal tag interno. L'attacco è unidirezionale: il traffico di ritorno non può seguire lo stesso percorso.

Contromisure:

- modificare la VLAN native dei trunk (usare una VLAN dedicata non assegnata ad alcun host);

- limitare le VLAN consentite sui trunk con il comando:

> switchport trunk allowed vlan 10,20,30

- disabilitare le porte inutilizzate e assegnarle a una VLAN di quarantena.

# 6. VLAN Speciali: Management e Voice

## 6.1 Management VLAN

Le Management VLAN sono VLAN dedicate all'amministrazione degli apparati di rete. Su di esse transitano protocolli e servizi di gestione quali SSH, SNMP (Simple Network Management Protocol), syslog e interfacce web di amministrazione.

Poiché l'accesso a una Management VLAN consente potenzialmente il controllo dell'intera infrastruttura di rete, essa rappresenta un obiettivo critico dal punto di vista della sicurezza. In ambienti avanzati è comune adottare:

- microsegmentazione con ACL dedicate per limitare l'accesso alla Management VLAN;

- Private VLAN (PVLAN) per isolare ulteriormente il traffico di gestione;

- reti di management fisicamente separate (out-of-band management);

- autenticazione a due fattori per l'accesso ai dispositivi.

|  |
|----|
| **Approfondimento: SNMP (Simple Network Management Protocol)** |
| SNMP è un protocollo applicativo (UDP, porta 161) per il monitoraggio e la gestione di dispositivi di rete. Utilizza un modello manager-agent: il manager interroga gli agent (residenti sui dispositivi) tramite operazioni GET/SET; gli agent inviano notifiche autonome (trap) al manager su eventi significativi. |
| SNMPv1 e SNMPv2c trasmettono le community string (password) in chiaro: non vanno usati su reti di produzione. SNMPv3 introduce autenticazione (HMAC-MD5 o HMAC-SHA) e cifratura (DES, AES): è la versione raccomandata per ambienti sicuri. |

## 6.2 Voice VLAN

Le Voice VLAN sono VLAN dedicate al traffico VoIP (Voice over IP). Hanno lo scopo di ridurre latenza e jitter, garantire priorità al traffico voce rispetto al traffico dati e migliorare la qualità delle comunicazioni audio.

I telefoni IP supportano due VLAN simultaneamente su una singola porta dello switch: una per il traffico dati (dati del PC collegato al passante del telefono) e una per il traffico voce. Il telefono IP taggherà autonomamente il traffico voce con il VLAN ID configurato, mentre il traffico dati rimane non taggato (VLAN access).

Configurazione Cisco IOS:

> interface FastEthernet0/18
>
> switchport mode access
>
> switchport access vlan 20
>
> switchport voice vlan 150
>
> mls qos trust cos

In questo esempio: VLAN 20 trasporta il traffico dati del PC collegato al telefono; VLAN 150 trasporta il traffico voce VoIP; il comando mls qos trust cos abilita la fiducia nel campo CoS (Class of Service) del tag 802.1Q inserito dal telefono, garantendo la corretta applicazione del QoS.

# 7. Configurazione delle VLAN e dei Trunk

## 7.1 Creazione e Verifica VLAN

> ! Creazione VLAN
>
> vlan 10
>
> name UFFICI
>
> vlan 20
>
> name LABORATORIO
>
> ! Verifica
>
> show vlan
>
> show vlan brief

## 7.2 Configurazione Trunk Completa

> interface GigabitEthernet0/1
>
> switchport mode trunk
>
> switchport trunk encapsulation dot1q
>
> switchport trunk native vlan 99
>
> switchport trunk allowed vlan 10,20,30
>
> switchport nonegotiate
>
> ! Verifica
>
> show interfaces trunk
>
> show interfaces GigabitEthernet0/1 switchport

|  |
|----|
| **Nota: VTP (VLAN Trunking Protocol)** |
| VTP è un protocollo Cisco proprietario che propaga automaticamente le informazioni sulle VLAN tra switch dello stesso dominio VTP. Semplifica la gestione ma introduce rischi: uno switch con revisione numero più alto può sovrascrivere la configurazione VLAN dell'intera rete. |
| In ambienti di produzione è comune disabilitare VTP (modalità "transparent" o "off") e gestire le VLAN manualmente su ogni switch, per evitare propagazioni indesiderate. |
| Versioni: VTPv1 e v2 supportano solo VLAN 1–1005 (normal range). VTPv3 aggiunge supporto per VLAN extended range (1006–4094) e Private VLAN. |

# 8. Routing Inter-VLAN

Poiché ogni VLAN costituisce un dominio di broadcast distinto e normalmente corrisponde a una subnet IP differente, la comunicazione tra host appartenenti a VLAN diverse richiede un'operazione di routing al Layer 3. Le tre architetture principali sono:

| **Architettura** | **Caratteristiche principali** |
|----|----|
| Router tradizionale (un'interfaccia per VLAN) | Un'interfaccia fisica per ogni VLAN. Semplice ma costoso in termini di porte. |
| Router-on-a-Stick | Una sola interfaccia fisica con più sottointerfacce logiche su un trunk. |
| Switch Layer 3 (Multilayer Switch) | Routing eseguito internamente allo switch tramite hardware dedicato (ASIC). Soluzione ad alte prestazioni. |

## 8.1 Router-on-a-Stick

Il Router-on-a-Stick è una soluzione che utilizza un singolo collegamento trunk tra switch e router. Sul router vengono create più sottointerfacce logiche (subinterface), ognuna associata a una VLAN tramite incapsulamento 802.1Q e configurata con l'indirizzo IP del gateway predefinito di quella VLAN.

Configurazione del router:

> interface GigabitEthernet0/0
>
> no ip address
>
> no shutdown
>
> interface GigabitEthernet0/0.10
>
> encapsulation dot1Q 10
>
> ip address 192.168.10.1 255.255.255.0
>
> interface GigabitEthernet0/0.20
>
> encapsulation dot1Q 20
>
> ip address 192.168.20.1 255.255.255.0
>
> interface GigabitEthernet0/0.99
>
> encapsulation dot1Q 99 native
>
> ip address 192.168.99.1 255.255.255.0

| **Elemento** | **Funzione** |
|----|----|
| GigabitEthernet0/0 | Interfaccia fisica: abilitata ma senza IP |
| GigabitEthernet0/0.10 | Sottointerfaccia logica per VLAN 10 |
| encapsulation dot1Q 10 | Associa la sottointerfaccia alla VLAN 10 |
| ip address 192.168.10.1 | Gateway predefinito per i dispositivi della VLAN 10 |
| encapsulation dot1Q 99 native | Indica che questa sottointerfaccia gestisce la VLAN native |

## 8.2 Switch Layer 3 (Multilayer Switch)

Negli ambienti di produzione, il routing inter-VLAN è quasi sempre eseguito da switch Layer 3 (Multilayer Switch, MLS), che integrano capacità di routing hardware tramite circuiti ASIC dedicati. Le prestazioni sono nettamente superiori al Router-on-a-Stick, che introduce un collo di bottiglia sulla singola interfaccia trunk.

Su switch Layer 3 Cisco, il routing viene abilitato tramite interfacce VLAN (SVI – Switched Virtual Interface):

> ip routing
>
> interface Vlan10
>
> ip address 192.168.10.1 255.255.255.0
>
> no shutdown
>
> interface Vlan20
>
> ip address 192.168.20.1 255.255.255.0
>
> no shutdown

# 9. Processo di Routing e Inoltro dei Pacchetti

Quando un host deve comunicare con un dispositivo su una subnet diversa, il processo si svolge come segue:

6.  L'host confronta l'indirizzo IP di destinazione con la propria subnet mask e determina che la destinazione è in una rete remota.

7.  L'host incapsula il pacchetto IP in un frame Ethernet indirizzato al MAC address del default gateway (ottenuto tramite ARP).

8.  Il frame raggiunge il router (o lo switch L3). Il router decapsula il frame, estrae il pacchetto IP e consulta la propria routing table.

9.  Il router trova la rotta per la rete di destinazione, decrementa il TTL di 1 e ricalcola il checksum IP.

10. Il router incapsula il pacchetto in un nuovo frame Ethernet con il MAC di destinazione del next-hop e lo instrada sull'interfaccia corretta.

|  |
|----|
| **Approfondimento: TTL (Time To Live)** |
| Il campo TTL (Time To Live) nel header IPv4 è un campo a 8 bit inizializzato dal mittente (valori tipici: 64 su Linux/macOS, 128 su Windows, 255 su Cisco IOS). Ogni router che instrada il pacchetto decrementa il TTL di 1. Quando il TTL raggiunge 0, il router scarta il pacchetto e invia un messaggio ICMP 'Time Exceeded' al mittente. |
| Funzione principale: prevenire loop di routing infiniti. Funzione diagnostica: il comando traceroute (tracert su Windows) sfrutta il TTL per identificare i router intermedi lungo il percorso. |
| Nota: IPv6 non utilizza TTL ma il campo equivalente Hop Limit, con la stessa semantica operativa. |

# 10. Subnetting e VLSM

Il Variable Length Subnet Masking (VLSM) consente di suddividere uno spazio di indirizzamento IP utilizzando subnet mask di lunghezza variabile, ottimizzando l'utilizzo degli indirizzi disponibili. Questa tecnica è standardizzata in RFC 1009 e RFC 1519 (CIDR – Classless Inter-Domain Routing).

## 10.1 Esempio di Suddivisione con VLSM

Rete iniziale: 192.168.0.0/24 (254 host utilizzabili).

Suddivisione in due subnet da 62 host:

| **Subnet**      | **Range indirizzi**          | **Host utilizzabili** |
|-----------------|------------------------------|-----------------------|
| 192.168.0.0/26  | 192.168.0.1 – 192.168.0.62   | 62                    |
| 192.168.0.64/26 | 192.168.0.65 – 192.168.0.126 | 62                    |

Maschera /26: 255.255.255.192 (11111111.11111111.11111111.11000000 in binario).

## 10.2 Subnet /30 per Collegamento Punto-Punto

Per i collegamenti punto-punto tra router è prassi consolidata utilizzare subnet /30, che fornisce esattamente 2 indirizzi host utilizzabili, riducendo al minimo lo spreco di indirizzi.

| **Indirizzo** | **Funzione**                |
|---------------|-----------------------------|
| 192.168.0.128 | Indirizzo di rete (Network) |
| 192.168.0.129 | Host (interfaccia router A) |
| 192.168.0.130 | Host (interfaccia router B) |
| 192.168.0.131 | Broadcast                   |

In ambienti con IPv6 o con indirizzi non scarsi, i collegamenti punto-punto possono utilizzare subnet /31 (RFC 3021) o addirittura indirizzi /127 in IPv6, consentendo di risparmiare ulteriori indirizzi.

# 11. Encapsulation e De-Encapsulation

Il modello a strati (OSI o TCP/IP) prevede che ogni livello aggiunga la propria intestazione (header) ai dati ricevuti dal livello superiore, in un processo detto incapsulamento (encapsulation). Il processo inverso, effettuato dal destinatario, è detto de-encapsulamento.

| **Livello** | **PDU (Protocol Data Unit)** | **Header aggiunto** |
|----|----|----|
| Applicazione/Trasporto | Segmento (TCP) / Datagramma (UDP) | Porte sorgente/dest., seq. number, ecc. |
| Rete (Layer 3) | Pacchetto IP | IP sorgente/dest., TTL, protocollo |
| Data Link (Layer 2) | Frame Ethernet / PPP / HDLC | MAC sorgente/dest., FCS |
| Fisico (Layer 1) | Bit / Segnale elettrico o ottico | Preambolo, codifica di linea |

Nota: su collegamenti seriali (WAN), il frame Ethernet viene rimosso e il pacchetto IP viene incapsulato in un protocollo di Layer 2 adatto al mezzo fisico, come PPP (Point-to-Point Protocol) o HDLC (High-Level Data Link Control). Il contenuto IP rimane invariato lungo tutto il percorso.

# 12. Routing Statico e Verifica della Configurazione

## 12.1 Routing Statico

Il routing statico prevede la configurazione manuale delle rotte nella routing table del router. È adatto a reti piccole, stabili e con percorsi predeterminati.

> ip route 192.168.0.0 255.255.255.0 192.168.100.1
>
> ip route 0.0.0.0 0.0.0.0 10.0.0.1 ! Default route

Il primo comando specifica: per raggiungere la rete 192.168.0.0/24, inviare i pacchetti al next-hop 192.168.100.1. Il secondo comando definisce la default route (rotta di default): tutti i pacchetti verso reti non elencate esplicitamente vengono inviati al gateway 10.0.0.1.

## 12.2 Comandi di Verifica

| **Comando**             | **Funzione**                          |
|-------------------------|---------------------------------------|
| show ip route           | Visualizza la routing table completa  |
| show ip interface brief | Stato delle interfacce e indirizzi IP |
| show interfaces trunk   | Stato e VLAN dei trunk                |
| show vlan brief         | Riepilogo VLAN e porte associate      |
| ping 192.168.x.x        | Test di raggiungibilità Layer 3       |
| traceroute 192.168.x.x  | Percorso dei pacchetti hop-by-hop     |

## 12.3 Problemi Comuni (Troubleshooting)

Durante la configurazione di reti VLAN, i problemi più frequenti riguardano:

- gateway predefinito errato o assente sull'host;

- interfacce nello stato administratively down (comando no shutdown dimenticato);

- VLAN non creata sullo switch o porta assegnata alla VLAN errata;

- trunk non configurato o VLAN non inclusa nel trunk;

- subnet mask errata che porta a una classificazione incorretta del traffico;

- routing mancante per una subnet (rotta assente nella routing table).

# 13. Transmission Control Protocol (TCP)

Il Transmission Control Protocol (TCP), definito in RFC 793 (1981) e successivamente aggiornato da RFC 7323, RFC 9293 e altri, è un protocollo di Layer 4 (Trasporto) orientato alla connessione. Fornisce un servizio di consegna affidabile, ordinato e controllato dei dati tra applicazioni su host remoti.

## 13.1 Formato dell'Header TCP

L'header TCP ha una dimensione minima di 20 byte (senza opzioni) e la seguente struttura:

| **Campo** | **Dimensione** | **Funzione** |
|----|----|----|
| Source Port | 16 bit | Porta applicazione sorgente (0–65535) |
| Destination Port | 16 bit | Porta applicazione destinazione |
| Sequence Number | 32 bit | Numero di sequenza del primo byte del segmento |
| Acknowledgment Number | 32 bit | Prossimo byte atteso (valido se ACK=1) |
| Data Offset (HLEN) | 4 bit | Lunghezza header in parole da 32 bit (min 5, max 15) |
| Reserved | 3 bit | Riservato, deve essere 0 |
| Control Flags | 9 bit | Flag di controllo (NS, CWR, ECE, URG, ACK, PSH, RST, SYN, FIN) |
| Window Size | 16 bit | Dimensione finestra di ricezione (byte) |
| Checksum | 16 bit | Controllo di integrità (header + dati + pseudo-header IP) |
| Urgent Pointer | 16 bit | Offset dati urgenti (valido se URG=1) |
| Options | variabile | Opzioni facoltative (MSS, SACK, Timestamps, Window Scale…) |
| Padding | variabile | Allineamento header a multiplo di 32 bit |

## 13.2 Flag TCP

| **Flag** | **Nome** | **Funzione** |
|----|----|----|
| SYN | Synchronize | Avvio connessione; sincronizza i Sequence Number |
| ACK | Acknowledge | Conferma ricezione dati; valida il campo Acknowledgment Number |
| FIN | Finish | Richiesta di chiusura ordinata della connessione (half-close) |
| RST | Reset | Reset immediato e chiusura forzata della connessione |
| PSH | Push | Richiede consegna immediata dei dati al layer applicativo |
| URG | Urgent | Indica dati urgenti; il campo Urgent Pointer è valido |
| ECE | ECN-Echo | Segnalazione congestione (Explicit Congestion Notification) |
| CWR | Congestion Window Reduced | Conferma riduzione finestra a seguito di ECN |
| NS | Nonce Sum | Protezione contro occultamento accidentale di flag ECN |

## 13.3 Three-Way Handshake TCP

L'apertura di una connessione TCP avviene attraverso un processo in tre fasi (three-way handshake):

11. Client → Server: SYN (Sequence Number = x)

12. Server → Client: SYN-ACK (Sequence Number = y, Acknowledgment Number = x+1)

13. Client → Server: ACK (Acknowledgment Number = y+1)

Al termine del three-way handshake, la connessione è stabilita in entrambe le direzioni e lo scambio di dati può avere inizio.

## 13.4 Controllo di Flusso e Finestra TCP

TCP implementa il controllo di flusso tramite il meccanismo della sliding window. Il campo Window Size nell'header TCP indica quanti byte il ricevitore è in grado di accettare nel proprio buffer di ricezione. Il mittente non può inviare più byte di quanti indicati dalla finestra corrente.

Se il buffer del ricevitore si riempie, questo può inviare un ACK con Window Size = 0 per fermare temporaneamente il mittente (zero window). Quando il buffer si svuota, invia un Window Update per riprendere la trasmissione.

L'estensione Window Scale (RFC 7323) consente di superare il limite teorico di 65535 byte della finestra TCP, fondamentale per le reti ad alta velocità e alta latenza (Long Fat Networks – LFN).

## 13.5 Porte TCP e Socket

TCP utilizza porte logiche a 16 bit (0–65535) per identificare i processi applicativi:

| **Range** | **Categoria** | **Esempi** |
|----|----|----|
| 0–1023 | Well-Known Ports (IANA registrate, privilegi root richiesti) | 80 HTTP, 443 HTTPS, 22 SSH, 25 SMTP, 53 DNS (anche UDP) |
| 1024–49151 | Registered Ports | 3306 MySQL, 5432 PostgreSQL, 8080 HTTP alternativo |
| 49152–65535 | Ephemeral/Dynamic Ports (lato client) | Assegnate dinamicamente dal SO per le connessioni in uscita |

Una socket è identificata dalla coppia (indirizzo IP, porta). Una connessione TCP è identificata univocamente dalla 4-tupla: (IP sorgente, porta sorgente, IP destinazione, porta destinazione). Questo consente a un server di gestire migliaia di connessioni simultanee sulla stessa porta di destinazione.

# 14. Considerazioni Finali

La comprensione approfondita di VLAN, trunking, subnetting, routing, encapsulation e TCP costituisce la base indispensabile per affrontare tematiche avanzate di sicurezza di rete e progettazione di architetture moderne.

Senza una comprensione concreta del funzionamento reale dei protocolli e dei dispositivi di rete, risulta impossibile interpretare correttamente:

- attacchi di rete (VLAN hopping, ARP spoofing, TCP session hijacking);

- anomalie nel traffico rilevate da sistemi IDS/IPS;

- log di firewall e router;

- catture di traffico (Wireshark, tcpdump);

- problemi di configurazione in ambienti complessi;

- meccanismi di difesa e hardening della rete.

# Bibliografia

**\[1\]** IEEE Std 802.1Q-2022, "IEEE Standard for Local and Metropolitan Area Networks—Bridges and Bridged Networks," IEEE, 2022.

**\[2\]** Postel, J. (1981). Transmission Control Protocol (RFC 793). Internet Engineering Task Force (IETF). https://www.rfc-editor.org/rfc/rfc793

**\[3\]** Borman, D., Braden, B., Jacobson, V., & Scheffenegger, R. (2014). TCP Extensions for High Performance (RFC 7323). IETF. https://www.rfc-editor.org/rfc/rfc7323

**\[4\]** Eddy, W. (2022). Transmission Control Protocol (STD 7, RFC 9293). IETF. https://www.rfc-editor.org/rfc/rfc9293

**\[5\]** Fuller, V., & Li, T. (2006). Classless Inter-Domain Routing (CIDR): The Internet Address Assignment and Aggregation Plan (RFC 4632). IETF. https://www.rfc-editor.org/rfc/rfc4632

**\[6\]** Lammle, T. (2020). CCNA Certification Study Guide, Volume 2: Exam 200-301. Sybex/Wiley.

**\[7\]** Odom, W. (2019). CCNA 200-301 Official Cert Guide (Volumes 1 & 2). Cisco Press.

**\[8\]** Stevens, W. R. (1994). TCP/IP Illustrated, Volume 1: The Protocols. Addison-Wesley.

**\[9\]** Tanenbaum, A. S., & Wetherall, D. J. (2011). Computer Networks (5th ed.). Pearson/Prentice Hall.

**\[10\]** Forouzan, B. A. (2021). Data Communications and Networking (5th ed.). McGraw-Hill Education.

**\[11\]** Cisco Systems. (2023). Catalyst Switches – VLAN Configuration Guide. https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst/

**\[12\]** Hassell, J. (2004). RADIUS: Securing Public Access to Private Resources. O'Reilly Media.

**\[13\]** Case, J., Fedor, M., Schoffstall, M., & Davin, J. (1990). A Simple Network Management Protocol (SNMP) (RFC 1157). IETF. https://www.rfc-editor.org/rfc/rfc1157

**\[14\]** Mauro, D. R., & Schmidt, K. J. (2005). Essential SNMP (2nd ed.). O'Reilly Media.
