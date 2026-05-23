# Lezione 8

*NAT, ICMP e Wireless Networking*

------------------------------------------------------------------------

# 1. Network Address Translation (NAT)

## 1.1 Introduzione e Motivazione

Il Network Address Translation (NAT) è un meccanismo che consente a dispositivi configurati con indirizzi IP privati di comunicare con reti esterne (tipicamente Internet) utilizzando uno o più indirizzi IP pubblici. È definito originariamente in RFC 1631 (1994) e successivamente aggiornato da RFC 3022 (2001).

La necessità del NAT nasce dall'esaurimento dello spazio di indirizzamento IPv4: con soli 2^32 ≈ 4,3 miliardi di indirizzi disponibili e una crescita esponenziale dei dispositivi connessi, il NAT ha permesso di riutilizzare gli stessi indirizzi privati in milioni di reti distinte, rimandando di decenni la transizione a IPv6.

## 1.2 Indirizzi IP Privati — RFC 1918

Gli indirizzi IP privati sono definiti da RFC 1918 (1996). I router su Internet non instradano pacchetti con questi indirizzi come sorgente o destinazione: sono visibili solo all'interno delle reti locali.

| **Range** | **Notazione CIDR** | **Numero di indirizzi** | **Utilizzo tipico** |
|----|----|----|----|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | 16.777.216 | Grandi reti aziendali |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | 1.048.576 | Reti medie |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | 65.536 | Reti domestiche e small office |

|  |
|----|
| **Approfondimento — Altri range speciali** |
| Oltre agli indirizzi RFC 1918, esistono altri range non instradabili su Internet: |
| \- 169.254.0.0/16 (RFC 3927): indirizzi link-local APIPA, assegnati automaticamente |
| quando un host non riesce a ottenere un indirizzo tramite DHCP. |
| \- 100.64.0.0/10 (RFC 6598): Shared Address Space, riservato ai provider per NAT |
| di livello carrier (CGN — Carrier-Grade NAT). |
| \- 127.0.0.0/8: loopback (127.0.0.1 è il localhost). |

## 1.3 Funzionamento del NAT — Traduzione degli Indirizzi

Quando un host interno genera un pacchetto verso Internet, il router NAT intercetta il pacchetto e sostituisce l'indirizzo IP sorgente privato con il proprio indirizzo pubblico. L'operazione inversa viene applicata ai pacchetti di risposta in arrivo.

Esempio: un host 192.168.0.10 contatta il server DNS 1.1.1.1 sulla porta UDP 53.

**Pacchetto generato dall'host (prima della traduzione):**

| **Campo**          | **Valore**                            |
|--------------------|---------------------------------------|
| IP sorgente        | 192.168.0.10                          |
| Porta sorgente     | 45000 (porta effimera scelta dall'OS) |
| IP destinazione    | 1.1.1.1                               |
| Porta destinazione | 53 (DNS)                              |

**Pacchetto modificato dal router NAT (dopo la traduzione):**

| **Campo**          | **Valore**                                |
|--------------------|-------------------------------------------|
| IP sorgente        | 3.3.3.3 (indirizzo pubblico del router)   |
| Porta sorgente     | 45000 (invariata in assenza di conflitti) |
| IP destinazione    | 1.1.1.1                                   |
| Porta destinazione | 53                                        |

Il server 1.1.1.1 risponde a 3.3.3.3:45000. Il router NAT consulta la propria tabella di traduzione e reinoltra la risposta a 192.168.0.10:45000.

## 1.4 La NAT Table (Tabella di Traduzione)

Per gestire correttamente il traffico di ritorno, il router mantiene una NAT Table (anche detta NAT translation table), che mappa ogni sessione attiva. Ogni voce contiene tipicamente:

- Indirizzo IP interno (inside local) e porta sorgente

- Indirizzo IP pubblico (inside global) e porta NAT assegnata

- Indirizzo IP destinazione (outside global) e porta destinazione

- Protocollo (TCP, UDP, ICMP)

- Timestamp / timer di scadenza

Esempio di voce nella NAT Table:

| **Inside Local**   | **Inside Global** | **Outside Global** | **Protocollo** |
|--------------------|-------------------|--------------------|----------------|
| 192.168.0.10:45000 | 3.3.3.3:45000     | 1.1.1.1:53         | UDP            |
| 192.168.0.11:45000 | 3.3.3.3:45001     | 8.8.8.8:53         | UDP            |

|  |
|----|
| **📘 Terminologia Cisco — Inside Local, Inside Global, Outside** |
| Cisco utilizza una nomenclatura precisa per descrivere gli indirizzi nel NAT: |
| \- Inside Local: indirizzo IP privato dell'host interno (es. 192.168.0.10) |
| \- Inside Global: indirizzo pubblico con cui l'host interno è visibile su Internet (es. 3.3.3.3:45000) |
| \- Outside Global: indirizzo IP del server remoto come visto dall'esterno (es. 1.1.1.1) |
| \- Outside Local: come il router NAT interno vede l'indirizzo del server remoto. |
| In NAT semplice coincide con Outside Global; in scenari con NAT doppio può differire. |

## 1.5 NAT e Port Address Translation (PAT)

Il termine generico NAT copre in realtà diverse modalità operative. La distinzione principale è tra NAT "puro" e PAT (Port Address Translation).

| **Modalità** | **Indirizzi pubblici** | **Gestione porte** | **Utilizzo tipico** |
|----|----|----|----|
| Static NAT | Uno-a-uno (fisso) | Nessuna modifica | Pubblicare server interni |
| Dynamic NAT | Pool di IP pubblici | Generalmente invariate | Reti medie con pool di IP |
| PAT / NAT Overload | Un solo IP pubblico | Modificate per distinguere le sessioni | Router domestici, hotspot |

Il PAT — spesso denominato NAT Overload nella terminologia Cisco — è il meccanismo più diffuso: un singolo indirizzo IP pubblico viene condiviso da molti host interni, differenziati tramite il numero di porta.

## 1.6 Collisione di Porte e Riassegnazione

Se due host interni utilizzano la stessa porta sorgente e devono essere tradotti verso lo stesso IP pubblico, il router non può mantenere due voci identiche nella NAT Table. La soluzione è rinumerare la porta di uno dei due:

| **Host interno** | **Porta originale** | **IP pubblico** | **Porta assegnata dal NAT** |
|----|----|----|----|
| 192.168.0.10 | 45000 | 3.3.3.3 | 45000 |
| 192.168.0.11 | 45000 | 3.3.3.3 | 45001 (rinumerata) |

Il range di porte disponibile per il PAT è 1–65535 (16 bit). Le porte 0–1023 sono dette well-known e riservate per servizi standard; le porte 1024–49151 sono registered; le porte 49152–65535 sono ephemeral (effimere). In pratica il NAT utilizza principalmente le porte effimere, ma può scendere nelle registered in caso di alta occupazione.

## 1.7 Limiti del NAT Overload

Il numero massimo teorico di traduzioni simultanee per un singolo IP pubblico è limitato dal numero di porte disponibili (65535). In scenari reali il limite effettivo è inferiore a causa di:

- Porte riservate (0–1023 non utilizzabili dal NAT per nuove sessioni)

- Porte già occupate da sessioni attive

- Timer di scadenza non ancora scaduti per sessioni chiuse

Il problema di esaurimento delle porte è reale in:

- Hotspot Wi-Fi pubblici con molti utenti simultanei

- Carrier-Grade NAT (CGN) dei provider, dove migliaia di utenti condividono pochi IP pubblici

- Reti aziendali molto grandi

|  |
|----|
| **📘 Approfondimento — Carrier-Grade NAT (CGN)** |
| Il CGN (RFC 6888) è un NAT operato direttamente dal provider Internet: il router |
| domestico del cliente riceve già un indirizzo RFC 6598 (100.64.0.0/10) invece di un |
| IP pubblico, e il provider applica un secondo livello di NAT. Questo schema è detto |
| Double NAT o NAT444. I problemi di NAT Traversal (VPN, VoIP, gaming) si aggravano |
| significativamente in presenza di CGN. |

## 1.8 NAT Pool

Per aumentare la capacità complessiva di traduzione, si può assegnare al router un pool di indirizzi IP pubblici. In questo caso il router distribuisce le sessioni su più IP, moltiplicando il numero totale di porte disponibili. Con un pool di N indirizzi pubblici, la capacità teorica sale a N × 65535 sessioni.

## 1.9 NAT Dinamico — Configurazione Cisco

Scenario: rete interna 192.168.0.0/24, pool pubblico 1.1.1.1–1.1.1.6.

**1) ACL per identificare il traffico interno da tradurre:**

> access-list 1 permit 192.168.0.0 0.0.0.255

**2) Definizione del NAT Pool:**

> ip nat pool PUBLIC_POOL 1.1.1.1 1.1.1.6 netmask 255.255.255.248

La subnet 255.255.255.248 (/29) copre 8 indirizzi: 1.1.1.0–1.1.1.7. Gli indirizzi 1.1.1.0 (network) e 1.1.1.7 (broadcast) non sono utilizzabili, quindi il pool contiene effettivamente 6 indirizzi (1.1.1.1–1.1.1.6). La parola chiave overload abilita il PAT: più client interni possono condividere lo stesso indirizzo pubblico del pool.

**3) Associazione NAT — lista ACL + pool:**

> ip nat inside source list 1 pool PUBLIC_POOL overload

**4) Designazione delle interfacce:**

> interface GigabitEthernet0/0
>
> ip address 192.168.0.1 255.255.255.0
>
> ip nat inside
>
> !
>
> interface GigabitEthernet0/1
>
> ip address 1.1.1.7 255.255.255.248
>
> ip nat outside

I comandi ip nat inside e ip nat outside sono obbligatori: senza di essi il router non sa quale direzione considerare "interna" e quale "esterna" per applicare le traduzioni.

**Comandi di verifica:**

> show ip nat translations
>
> show ip nat statistics
>
> debug ip nat

## 1.10 NAT Statico e Port Forwarding

Il NAT statico crea un'associazione permanente e bidirezionale tra un indirizzo (o una porta) pubblico e un host interno. A differenza del NAT dinamico, il traffico può essere avviato dall'esterno: questo permette di pubblicare servizi interni su Internet.

La forma più comune è il Port Forwarding (o DNAT — Destination NAT): il router mappa una porta pubblica su un host e una porta interni specifici.

Esempio: server HTTPS interno raggiungibile dall'esterno.

| **IP pubblico** | **Porta pubblica** | **IP interno** | **Porta interna** | **Protocollo** |
|----|----|----|----|----|
| 1.1.1.1 | 443 | 192.168.0.254 | 443 | TCP |

**Configurazione Cisco:**

> ip nat inside source static tcp 192.168.0.254 443 1.1.1.1 443

|  |
|----|
| **Nota — Port Forwarding e sicurezza** |
| Il Port Forwarding espone direttamente un servizio interno su Internet. |
| Prima di abilitarlo è necessario: |
| \- Verificare che il servizio esposto sia aggiornato e privo di vulnerabilità note. |
| \- Applicare ACL che limitino ulteriormente il traffico in ingresso (es. solo da IP noti). |
| \- Considerare l'uso di un reverse proxy (es. nginx) o di una DMZ invece di esporre |
| direttamente il server al traffico pubblico non filtrato. |

## 1.11 NAT e Sicurezza

Il NAT dinamico produce un effetto collaterale importante dal punto di vista della sicurezza: gli host interni non sono direttamente raggiungibili dall'esterno. Il traffico in ingresso viene accettato soltanto se corrisponde a una voce esistente nella NAT Table, cioè a una sessione avviata dall'interno.

Questo comportamento è superficialmente simile a quello di un firewall stateful, ma occorre precisare che **il NAT non è un firewall.** Non applica policy di sicurezza esplicite, non registra eventi di sicurezza e non ispeziona il payload dei pacchetti. La protezione che offre è un effetto collaterale della sua funzione primaria di traduzione. In ambienti di produzione, il NAT non sostituisce un firewall: i due meccanismi sono complementari.

|  |
|----|
| **Approfondimento — Firewall Stateful** |
| Un firewall stateful mantiene una state table delle connessioni attive (simile alla NAT Table) |
| e consente il traffico di ritorno solo per le sessioni che ha autorizzato in base a policy |
| esplicite. Oltre al tracking delle sessioni, un firewall moderno (NGFW) può ispezionare |
| il payload applicativo (Deep Packet Inspection), rilevare intrusioni (IDS/IPS), filtrare |
| URL e applicazioni, e generare log dettagliati. Il NAT non svolge nessuna di queste funzioni. |

## 1.12 NAT e IPv6

IPv6 è stato progettato con spazio di indirizzamento praticamente illimitato: 2^128 ≈ 3,4 × 10^38 indirizzi. Ogni dispositivo può avere un indirizzo globale univoco, rendendo il NAT tecnicamente superfluo.

La transizione a IPv6 introduce tuttavia implicazioni di sicurezza rilevanti: gli host diventano potenzialmente raggiungibili direttamente dall'esterno, eliminando la protezione implicita offerta dal NAT. Questo rende il firewall ancora più critico nelle reti IPv6 rispetto alle reti IPv4 con NAT.

|  |
|----|
| **📘 Approfondimento — NPTv6 e NAT64** |
| Sebbene IPv6 non richieda NAT, esistono due meccanismi correlati: |
| \- NPTv6 (Network Prefix Translation, RFC 6296): traduce il prefisso di rete IPv6 senza |
| modificare le porte, utile per la ridondanza multi-homing senza BGP. |
| \- NAT64 (RFC 6146): consente a host IPv6-only di comunicare con server IPv4. |
| Un gateway traduce le intestazioni IPv6 in IPv4 e viceversa. Richiede DNS64 (RFC 6147) |
| per sintetizzare risposte DNS AAAA per host che hanno solo record A. |

## 1.13 NAT Traversal

Alcuni protocolli applicativi incorporano indirizzi IP o numeri di porta nel payload (nel corpo del messaggio applicativo, non solo negli header IP/TCP). Quando il NAT traduce gli header del pacchetto, il payload rimane invariato, causando inconsistenze. Questo problema è detto NAT Traversal.

### FTP e Application Layer Gateway (ALG)

FTP utilizza due canali TCP distinti:

- Canale di controllo (porta 21): per comandi (USER, PASS, LIST, RETR, ecc.)

- Canale dati: in modalità attiva (PORT), il client comunica nel payload il proprio IP e porta; in modalità passiva (PASV), è il server a comunicare IP e porta nel payload.

In presenza di NAT, gli indirizzi IP trasmessi nel payload FTP sono quelli privati interni, che non sono raggiungibili dall'esterno. La soluzione è un Application Layer Gateway (ALG): un modulo del router NAT che ispeziona e riscrive il payload FTP sostituendo gli indirizzi privati con quelli pubblici. Cisco IOS implementa l'ALG FTP automaticamente.

### VPN e IPsec

IPsec in modalità tunnel cifra l'intero pacchetto IP originale, incluse le porte TCP/UDP. Questo impedisce al NAT di leggere o modificare le porte, rendendo impossibile il PAT in assenza di meccanismi appositi.

La soluzione è NAT-T (NAT Traversal per IPsec, RFC 3948): i pacchetti IPsec ESP vengono incapsulati in UDP sulla porta 4500, permettendo al NAT di gestirli come normale traffico UDP.

### UDP e Timer NAT

UDP è un protocollo connectionless: non esiste un handshake o una chiusura esplicita della sessione. Il router NAT gestisce le voci UDP tramite timer: se non arriva traffico entro il timeout (tipicamente 30–300 secondi secondo la configurazione), la voce viene eliminata. Il traffico UDP successivo verrebbe scartato perché non corrisponderebbe ad alcuna voce attiva nella NAT Table.

Questo causa problemi a protocolli come SIP (VoIP) e QUIC (HTTP/3), che utilizzano UDP con sessioni di lunga durata. La soluzione comune è il keepalive applicativo: il client invia periodicamente piccoli pacchetti per mantenere attiva la voce NAT.

### NAT Hole Punching

Il NAT Hole Punching è una tecnica che permette a due host, entrambi dietro un NAT, di stabilire una comunicazione peer-to-peer diretta senza server intermedi. È ampiamente utilizzato in VoIP, WebRTC, gaming online e applicazioni P2P.

Il meccanismo si basa su un server STUN (Session Traversal Utilities for NAT, RFC 8489) pubblico che aiuta i peer a scoprire i propri IP e porte pubblici:

1.  Client A e Client B si connettono entrambi al server STUN

2.  Il server STUN comunica a ciascun client l'IP e la porta pubblica dell'altro

3.  Entrambi i client iniziano a inviare traffico verso l'IP:porta pubblico dell'altro

4.  I router NAT di entrambi i client, vedendo traffico uscente verso quell'IP:porta,

5.  aprono una voce nella NAT Table; il traffico di risposta viene quindi accettato

6.  La connessione diretta peer-to-peer è stabilita

|  |
|----|
| **📘 Approfondimento — STUN, TURN, ICE** |
| Il framework ICE (Interactive Connectivity Establishment, RFC 8445) è il meccanismo |
| standardizzato per il NAT Traversal nei protocolli moderni (WebRTC, SIP): |
| \- STUN (RFC 8489): scoperta dell'indirizzo pubblico e tentativo di hole punching. |
| \- TURN (RFC 8656): relay server di fallback quando il hole punching fallisce |
| (es. NAT simmetrico). Il traffico transita attraverso il server TURN, con impatto |
| su latenza e costi di banda. |
| \- ICE raccoglie tutti i candidate address (host, STUN, TURN) e seleziona il percorso |
| ottimale tramite connectivity checks. |

# 2. ICMP — Internet Control Message Protocol

## 2.1 Introduzione

ICMP (definito in RFC 792 per IPv4 e RFC 4443 per IPv6) è il protocollo di segnalazione e diagnostica del livello IP. Nonostante operi sopra IP (il campo Protocol dell'header IPv4 ha valore 1 per ICMP), ICMP è considerato parte integrante del livello 3 (Network) e non del livello 4 (Transport): non fornisce connessioni, porte, o consegna affidabile.

ICMP è utilizzato per:

- Segnalare errori di consegna dei pacchetti al mittente originale

- Diagnosticare la raggiungibilità degli host (ping)

- Mappare il percorso di rete (traceroute)

- Comunicare informazioni di routing (ICMP Redirect)

## 2.2 Formato dell'Header ICMP

Ogni messaggio ICMP è composto da un header di 8 byte fissi seguito da un payload variabile:

| **Campo** | **Dimensione** | **Descrizione** |
|----|----|----|
| Type | 8 bit | Tipo di messaggio ICMP (es. 8 = Echo Request, 0 = Echo Reply, 3 = Destination Unreachable) |
| Code | 8 bit | Sotto-tipo che specifica la causa del messaggio (es. Type 3 Code 0 = Network Unreachable) |
| Checksum | 16 bit | Checksum dell'intero messaggio ICMP (header + payload); calcolato con complemento a uno |
| Rest of Header | 32 bit | Campo variabile il cui significato dipende dal tipo (es. Identifier+Sequence per Echo) |

Il payload di un messaggio di errore ICMP contiene generalmente:

- L'header IP del pacchetto originale che ha causato l'errore (20 byte)

- I primi 8 byte del payload del pacchetto originale (che contengono la testata TCP/UDP)

Questo permette al mittente di identificare quale socket applicativo ha generato il pacchetto che ha causato l'errore.

## 2.3 Messaggi ICMP Principali (IPv4)

| **Type** | **Code** | **Nome** | **Descrizione** |
|----|----|----|----|
| 0 | 0 | Echo Reply | Risposta a un Echo Request (ping) |
| 3 | 0 | Destination Unreachable — Network | La rete di destinazione non è raggiungibile (nessuna rotta) |
| 3 | 1 | Destination Unreachable — Host | L'host di destinazione non risponde |
| 3 | 3 | Destination Unreachable — Port | La porta destinazione non è in ascolto (UDP) |
| 3 | 13 | Comm. Administratively Prohibited | Traffico bloccato da una ACL o da un firewall |
| 5 | 0/1 | Redirect | Suggerisce al mittente un gateway migliore |
| 8 | 0 | Echo Request | Richiesta ping |
| 11 | 0 | Time Exceeded — TTL in transit | TTL sceso a 0 durante il transito (usato da traceroute) |
| 11 | 1 | Time Exceeded — Fragment reassembly | Timeout nella riassemblazione dei frammenti IP |

|  |
|----|
| **📘 Approfondimento — ICMPv6** |
| ICMPv6 (RFC 4443) svolge in IPv6 funzioni aggiuntive rispetto a ICMPv4: |
| \- Sostituisce ARP: il Neighbor Discovery Protocol (NDP, RFC 4861) usa ICMPv6 Type 135 |
| (Neighbor Solicitation) e Type 136 (Neighbor Advertisement) per la risoluzione |
| degli indirizzi MAC, equivalente a ARP in IPv4. |
| \- Router Discovery: i messaggi Router Solicitation (Type 133) e Router Advertisement |
| (Type 134) sostituiscono i broadcast DHCP per la configurazione automatica degli indirizzi. |
| \- Bloccare ICMPv6 con un firewall può rendere non funzionante una rete IPv6: a differenza |
| di ICMPv4, ICMPv6 è strettamente necessario al funzionamento di IPv6. |

## 2.4 Echo Request e Echo Reply — il comando ping

Il comando ping utilizza:

- ICMP Type 8 Code 0 (Echo Request) per la richiesta

- ICMP Type 0 Code 0 (Echo Reply) per la risposta

L'header specifico del messaggio Echo contiene:

- Identifier (16 bit): identifica il processo che ha originato il ping (tipicamente il PID)

- Sequence Number (16 bit): numerazione sequenziale dei pacchetti inviati

Dall'output del ping è possibile ricavare:

- Round Trip Time (RTT): latenza andata-ritorno in millisecondi

- Packet Loss: percentuale di pacchetti senza risposta

- TTL del pacchetto di risposta: permette di stimare il numero di hop

## 2.5 Time Exceeded e traceroute

Il comando traceroute sfrutta il meccanismo TTL (Time To Live) dell'header IP:

7.  Invia un pacchetto con TTL = 1; il primo router lo scarta e risponde con ICMP Type 11 Code 0

8.  Invia un pacchetto con TTL = 2; il secondo router risponde

9.  Incrementa il TTL fino a raggiungere la destinazione

In questo modo viene identificato l'indirizzo IP di ciascun router intermedio (hop) lungo il percorso, insieme alla latenza verso ogni hop.

Implementazioni diverse usano protocolli diversi per traceroute:

- Unix/Linux: UDP su porte alte (default) o ICMP (opzione -I)

- Windows (tracert): ICMP Echo Request

- Alcune implementazioni moderne: TCP SYN (tcptraceroute) per attraversare firewall che bloccano ICMP e UDP

# 3. Wireless Networking

## 3.1 Panoramica delle Tecnologie Wireless

Il termine "wireless networking" copre uno spettro ampio di tecnologie che differiscono per range, throughput, consumo energetico e caso d'uso:

| **Tecnologia** | **Standard** | **Range tipico** | **Throughput tipico** | **Utilizzo** |
|----|----|----|----|----|
| Wi-Fi | IEEE 802.11 | ~50–300 m | Mbps–Gbps | LAN wireless, accesso Internet |
| Bluetooth | IEEE 802.15.1 | ~10 m (Class 2) | 1–50 Mbps | Periferiche, audio, IoT |
| Bluetooth LE | IEEE 802.15.1 | ~10–100 m | ~1 Mbps | IoT a bassa potenza |
| Zigbee | IEEE 802.15.4 | ~10–100 m | 250 kbps | Domotica, sensori |
| RFID/NFC | ISO 14443/18000 | cm–metri | kbps | Identificazione, pagamenti |
| 4G LTE | 3GPP Rel.8+ | km | ~100 Mbps | Rete mobile |
| 5G NR | 3GPP Rel.15+ | variabile | ~Gbps | Rete mobile, IoT massivo |

Il corso si concentra su Wi-Fi (IEEE 802.11) in quanto tecnologia dominante per le reti locali wireless.

## 3.2 Bande di Frequenza e Canali Wi-Fi

Il Wi-Fi utilizza principalmente le bande ISM (Industrial, Scientific and Medical), che non richiedono licenza per l'uso:

| **Banda** | **Standard 802.11** | **Canali disponibili (IT/EU)** | **Note** |
|----|----|----|----|
| 2.4 GHz | 802.11b/g/n/ax | 13 (canali 1–13) | Maggiore penetrazione dei muri; più congestionata; canali sovrapposti |
| 5 GHz | 802.11a/n/ac/ax | 19+ (dipende dal paese) | Meno congestionata; range ridotto; più throughput disponibile |
| 6 GHz | 802.11ax (Wi-Fi 6E) | Fino a 59 canali da 20 MHz | Solo Wi-Fi 6E e superiori; meno interferenze; range più limitato |

Nella banda 2.4 GHz i canali sono separati di 5 MHz ma ciascuno occupa 20 MHz di banda, quindi si sovrappongono. I soli tre canali non sovrapposti in Europa sono 1, 6, 11. In un ambiente con molti Access Point, la scelta accurata dei canali è critica per evitare Co-Channel Interference (CCI).

## 3.3 Standard IEEE 802.11

| **Standard** | **Nome commerciale** | **Banda** | **Velocità massima teorica** | **Anno** |
|----|----|----|----|----|
| 802.11b | — | 2.4 GHz | 11 Mbps | 1999 |
| 802.11a | — | 5 GHz | 54 Mbps | 1999 |
| 802.11g | — | 2.4 GHz | 54 Mbps | 2003 |
| 802.11n | Wi-Fi 4 | 2.4/5 GHz | 600 Mbps (4×4 MIMO) | 2009 |
| 802.11ac | Wi-Fi 5 | 5 GHz | 3.5 Gbps (MU-MIMO) | 2013 |
| 802.11ax | Wi-Fi 6/6E | 2.4/5/6 GHz | 9.6 Gbps | 2019/2021 |
| 802.11be | Wi-Fi 7 | 2.4/5/6 GHz | 46 Gbps (teorico) | 2024 |

|  |
|----|
| **Approfondimento — Modulazione e OFDM** |
| Le velocità indicate sono valori teorici massimi ottenibili con condizioni radio ideali. |
| Le prestazioni reali dipendono da: |
| \- Tecnica di modulazione: 802.11ax usa fino a 1024-QAM (10 bit per simbolo). |
| \- OFDM (Orthogonal Frequency-Division Multiplexing): divide il canale in sottoportanti |
| ortogonali; usato da 802.11a/g/n/ac/ax. |
| \- MIMO (Multiple Input Multiple Output): più antenne trasmittenti e riceventi per |
| aumentare throughput e affidabilità tramite spatial multiplexing e beamforming. |
| \- MU-MIMO (Multi-User MIMO, introdotto in 802.11ac Wave 2): consente all'AP di |
| trasmettere simultaneamente a più client su canali spaziali distinti. |
| \- OFDMA (introdotto in 802.11ax): suddivide il canale in Resource Units assegnabili |
| a client diversi nella stessa trasmissione, riducendo la latenza. |

## 3.4 Architettura — Access Point, SSID, BSSID

Il Wi-Fi in modalità infrastructure si articola su:

- BSS (Basic Service Set): una cella wireless composta da un AP e dai client associati

- SSID (Service Set Identifier): nome della rete (fino a 32 byte); trasmesso nei beacon frame

- BSSID (Basic Service Set Identifier): MAC Address dell'AP; identifica univocamente la BSS

- ESS (Extended Service Set): insieme di più BSS con lo stesso SSID collegate da un distribution system (DS); permette il roaming

La modalità Ad-Hoc (IBSS — Independent BSS) consente la comunicazione diretta tra dispositivi senza AP. Oggi è in gran parte sostituita da Wi-Fi Direct (P2P), che offre le stesse funzionalità con migliore gestione dell'energia.

## 3.5 Accesso al Mezzo — CSMA/CA

A differenza di Ethernet (che usa CSMA/CD — Collision Detection), il Wi-Fi utilizza CSMA/CA (Carrier Sense Multiple Access with Collision Avoidance). La collision detection non è applicabile nel wireless perché:

- Un dispositivo non può trasmettere e ascoltare simultaneamente sulla stessa frequenza

- La potenza del segnale trasmesso è molto superiore a quella del segnale ricevuto

Il meccanismo CSMA/CA si articola nei seguenti passi:

10. Il dispositivo ascolta il canale (Carrier Sense)

11. Se il canale è libero per un intervallo DIFS (Distributed IFS), avvia il backoff casuale

12. Decrementa un contatore casuale (random backoff) mentre il canale rimane libero

13. Quando il contatore raggiunge zero, trasmette il frame

14. Attende l'ACK dal destinatario per un intervallo SIFS (Short IFS)

15. Se l'ACK non arriva, raddoppia il backoff (exponential backoff) e riprova

| **Intervallo** | **Valore tipico (802.11g)** | **Funzione** |
|----|----|----|
| SIFS | 10 µs | Separazione tra frame che appartengono alla stessa sequenza (ACK, CTS) |
| DIFS | 50 µs | Tempo di attesa prima di tentare una nuova trasmissione dati |
| AIFS | variabile | Arbitration IFS: versione parametrizzabile di DIFS usata in 802.11e (QoS) |

## 3.6 Hidden Node Problem e RTS/CTS

Il problema del nodo nascosto (Hidden Node Problem) si verifica quando due client possono entrambi comunicare con l'AP ma non si "vedono" reciprocamente (sono fuori dal reciproco range radio). Entrambi potrebbero iniziare a trasmettere simultaneamente, causando collisioni all'AP.

La soluzione è il meccanismo RTS/CTS (Request To Send / Clear To Send):

16. Il client invia un frame RTS all'AP (piccolo pacchetto di controllo)

17. L'AP risponde con un frame CTS visibile a tutti i client nel raggio dell'AP

18. Il CTS riserva il canale per la durata della trasmissione imminente

19. Gli altri client che sentono il CTS si astengono dal trasmettere (Network Allocation Vector — NAV)

RTS/CTS introduce overhead (4 frame aggiuntivi per ogni trasmissione dati) e viene tipicamente abilitato solo per frame di grandi dimensioni tramite il parametro RTS Threshold (default: 2347 byte, equivale a disabilitare RTS/CTS per la maggior parte del traffico).

## 3.7 Struttura dei Frame IEEE 802.11

Un frame IEEE 802.11 ha una struttura più complessa rispetto a un frame Ethernet. I campi principali del MAC header sono:

| **Campo** | **Dimensione** | **Descrizione** |
|----|----|----|
| Frame Control | 2 byte | Tipo (Management/Control/Data), sottotipo, flag (To DS, From DS, WEP, ecc.) |
| Duration/ID | 2 byte | Durata della trasmissione (usata per il NAV) o AID del client |
| Address 1 | 6 byte | Destinazione immediata (ricevitore) |
| Address 2 | 6 byte | Sorgente immediata (trasmettitore) |
| Address 3 | 6 byte | Indirizzo filtro (es. BSSID o indirizzo IP destinazione mappato a MAC) |
| Sequence Control | 2 byte | Numero di sequenza per il controllo duplicati |
| Address 4 | 6 byte | Presente solo in WDS (Wireless Distribution System) |
| QoS Control | 2 byte | Priorità (solo frame QoS Data, 802.11e) |
| FCS | 4 byte | Frame Check Sequence (CRC-32) |

| **Tipo di frame** | **Sottotipi comuni** | **Funzione** |
|----|----|----|
| Management (00) | Beacon, Probe Request/Response, Authentication, Association Request/Response, Deauthentication | Gestione della rete e dell'associazione dei client |
| Control (01) | RTS, CTS, ACK, Block ACK | Coordinamento dell'accesso al mezzo |
| Data (10) | Data, Null Data, QoS Data | Trasporto del payload utente |

## 3.8 Beacon Frame

Il beacon frame è trasmesso periodicamente dall'AP (default: ogni 102.4 ms = 10 volte/secondo) e contiene:

- SSID (o stringa vuota se SSID broadcasting è disabilitato)

- BSSID (MAC Address dell'AP)

- Timestamp (per la sincronizzazione dei timer)

- Beacon Interval (intervallo tra beacon consecutivi)

- Capability Information (es. privacy/cifratura supportata, QoS)

- Supported Rates e Extended Supported Rates (velocità supportate)

- Informazioni RSN (Robust Security Network): cifratura e autenticazione supportate

- Informazioni HT/VHT/HE (802.11n/ac/ax): canali, larghezza, MIMO)

|  |
|----|
| **Nota — SSID nascosto** |
| Disabilitare il beacon SSID (hidden SSID) non costituisce una misura di sicurezza efficace: |
| l'SSID viene comunque trasmesso nei frame di Probe Response e nei frame di Association. |
| Un'analisi passiva con Wireshark in Monitor Mode rivela immediatamente gli SSID nascosti |
| non appena un client si associa all'AP. L'unico effetto pratico è rendere la rete |
| invisibile agli utenti non tecnici. |

## 3.9 Sicurezza Wi-Fi

### WEP (Wired Equivalent Privacy)

WEP (IEEE 802.11, 1997) utilizza l'algoritmo RC4 con chiavi a 40 o 104 bit. È stato dimostrato crittograficamente insicuro già nel 2001: i vettori di inizializzazione (IV) a 24 bit si ripetono statisticamente, consentendo il recupero della chiave in pochi minuti con strumenti come aircrack-ng. WEP non deve essere utilizzato in nessun contesto.

### WPA (Wi-Fi Protected Access)

WPA (2003) è stato introdotto come soluzione emergenziale compatibile con l'hardware WEP esistente. Utilizza TKIP (Temporal Key Integrity Protocol) con chiavi per-pacchetto e un Message Integrity Code (MIC, algoritmo Michael). TKIP è anch'esso considerato insicuro e deprecato.

### WPA2 (IEEE 802.11i, 2004)

WPA2 introduce CCMP (Counter Mode CBC-MAC Protocol) basato su AES-128, che ha costituito per anni lo standard de facto per la sicurezza Wi-Fi. Supporta due modalità:

- Personal (PSK — Pre-Shared Key): password condivisa; usata in ambienti domestici e piccoli uffici

- Enterprise (802.1X/EAP): autenticazione tramite server RADIUS; usata in ambienti aziendali

WPA2 è stato colpito nel 2017 dall'attacco KRACK (Key Reinstallation Attack), che sfrutta una vulnerabilità nel four-way handshake 802.11i per reinstallare una chiave già usata, consentendo la decifratura e in certi casi la manipolazione del traffico. Il problema è stato mitigato tramite patch firmware/software rilasciate dai produttori.

### WPA3 (Wi-Fi Alliance, 2018)

WPA3 sostituisce il four-way handshake PSK con SAE (Simultaneous Authentication of Equals, basato sul protocollo Dragonfly — RFC 7664), che offre protezione contro attacchi offline a dizionario anche se la password è debole, e garantisce Perfect Forward Secrecy (PFS).

| **Protocollo** | **Anno** | **Cifratura** | **Vulnerabilità note** | **Stato** |
|----|----|----|----|----|
| WEP | 1997 | RC4 (40/104 bit) | IV deboli: recupero chiave in minuti | Deprecato — non usare |
| WPA | 2003 | TKIP/RC4 | MIC debole, TKIP vulnerabile | Deprecato |
| WPA2 | 2004 | AES-128 CCMP | KRACK (2017), PMKID attack | Accettabile se patchato |
| WPA3 | 2018 | AES-192/256 GCMP | Dragonblood (parziale, 2019) | Raccomandato |

## 3.10 Evil Twin Attack e Deauthentication Attack

### Evil Twin Attack

Un Evil Twin è un AP malevolo che imita una rete Wi-Fi legittima (stesso SSID e, idealmente, stesso BSSID clonato). L'obiettivo è attrarre i client verso l'AP controllato dall'attaccante per intercettare traffico, rubare credenziali o eseguire attacchi man-in-the-middle.

La tecnica si articola tipicamente in:

20. Creazione di un AP con lo stesso SSID della rete bersaglio

21. Opzionalmente: clonazione del BSSID (MAC spoofing dell'AP)

22. Invio di frame di deauthentication ai client associati all'AP legittimo

23. I client si riconnettono automaticamente e, se il segnale del finto AP è più forte, si associano ad esso

24. L'attaccante esegue il captive portal per raccogliere credenziali o il MITM per intercettare il traffico

### Deauthentication Attack

I frame di deauthentication e disassociation appartengono alla categoria Management e nello standard 802.11 originale non sono autenticati: chiunque può inviare un frame di deauthentication con qualsiasi BSSID e MAC sorgente, forzando la disconnessione dei client.

La contromisura introdotta in 802.11w (Management Frame Protection, MFP) autentica crittograficamente i frame di management critici (deauthentication, disassociation, action). 802.11w è obbligatorio in WPA3 e opzionale ma disponibile in WPA2.

|  |
|----|
| **Nota — Uso etico degli strumenti di analisi wireless** |
| Strumenti come Aircrack-ng, Wireshark in Monitor Mode, Hostapd-wpe e Bettercap possono |
| essere usati sia per attività legittime (audit di sicurezza, troubleshooting) che per |
| attività illegali. L'intercettazione di comunicazioni wireless senza autorizzazione è |
| un reato penale in Italia (art. 617-bis c.p.) e nella maggior parte delle giurisdizioni. |
| Le attività di laboratorio devono essere eseguite esclusivamente su reti di test isolate |
| o su reti per cui si dispone di autorizzazione esplicita scritta. |

## 3.11 Monitor Mode e Analisi del Traffico Wi-Fi

Per catturare tutti i frame wireless visibili nel range radio (inclusi Management e Control), la scheda di rete deve operare in Monitor Mode. In questa modalità la scheda cattura tutti i frame senza filtrare per BSSID o indirizzo MAC.

La modalità managed (normale) filtra tutti i frame non destinati al proprio BSS.

Elementi visibili in Monitor Mode con Wireshark:

- Beacon frame (da tutti gli AP nel range): SSID, BSSID, canale, cifratura, velocità supportate

- Probe Request (dai client): il client cerca reti conosciute; rivela SSID a cui il dispositivo si è connesso in passato

- Probe Response (dagli AP): risposta al Probe Request

- Authentication e Association frame: processo di connessione del client

- Deauthentication / Disassociation frame: disconnessione (legittima o di un attacco)

- Frame dati cifrati: il payload è cifrato (CCMP/GCMP), ma header MAC visibili

|  |
|----|
| **Approfondimento — Radiotap Header** |
| Wireshark in Monitor Mode visualizza un Radiotap Header preposto a ogni frame 802.11. |
| Il Radiotap Header è aggiunto dal driver della scheda wireless e contiene metadati |
| del segnale fisico non presenti nel frame 802.11 standard: |
| \- RSSI (Received Signal Strength Indicator): potenza del segnale in dBm |
| \- Noise floor: livello di rumore di fondo |
| \- Canale radio e frequenza |
| \- Data rate della trasmissione |
| \- Flag: FCS valido, frame corto (short preamble), STBC, ecc. |
| Questi dati sono utili per diagnosticare problemi di copertura radio. |

# 4. Esercizi di Laboratorio

## Esercizio 1 — Configurazione NAT Overload su Cisco IOS

### Scenario

Rete interna: 10.0.0.0/24, gateway 10.0.0.1 su GigabitEthernet0/0. Indirizzo pubblico assegnato dal provider: 203.0.113.1/30 su GigabitEthernet0/1.

### Obiettivo

Consentire a tutti gli host interni di navigare su Internet tramite NAT Overload usando il singolo IP pubblico 203.0.113.1.

### Configurazione

> access-list 10 permit 10.0.0.0 0.0.0.255
>
> !
>
> interface GigabitEthernet0/0
>
> ip address 10.0.0.1 255.255.255.0
>
> ip nat inside
>
> !
>
> interface GigabitEthernet0/1
>
> ip address 203.0.113.1 255.255.255.252
>
> ip nat outside
>
> !
>
> ip nat inside source list 10 interface GigabitEthernet0/1 overload

Nota: il comando ip nat inside source list 10 interface GigabitEthernet0/1 overload riferisce l'IP pubblico all'interfaccia invece di un pool statico. È la configurazione più comune quando si dispone di un solo IP pubblico dinamico.

### Verifica e domande

> show ip nat translations
>
> show ip nat statistics

25. Aprire una connessione HTTP da un host interno. Qual è la porta NAT assegnata nella tabella di traduzione?

26. Aprire due connessioni simultanee da due host diversi verso lo stesso server. Come differiscono le voci nella NAT Table?

27. Cosa succede alle voci nella NAT Table dopo che la connessione viene chiusa?

28. Come si configurerebbe un Port Forwarding per pubblicare un server SSH interno (10.0.0.100:22)?

## Esercizio 2 — Analisi ICMP con Wireshark

### Obiettivo

Catturare e analizzare messaggi ICMP generati da ping e traceroute.

### Attività

29. Avviare la cattura su Wireshark con filtro: icmp

30. Eseguire: ping -c 4 8.8.8.8

31. Eseguire: traceroute 8.8.8.8 (o tracert su Windows)

32. Fermare la cattura e analizzare i frame

### Domande

33. Qual è il valore del campo Type nei pacchetti Echo Request e Echo Reply?

34. Come cambia il campo TTL nell'header IP nei pacchetti ICMP Time Exceeded di traceroute?

35. Quanti hop separa il tuo host dal server 8.8.8.8? Qual è il RTT medio per ciascun hop?

36. Bloccare ICMP con una ACL (deny icmp any any): il traceroute funziona ancora? Perché?

## Esercizio 3 — Analisi Wi-Fi con Monitor Mode

### Obiettivo

Osservare il traffico Wi-Fi a basso livello su una rete di test autorizzata.

### Prerequisiti

- Scheda Wi-Fi con supporto Monitor Mode

- Rete di test isolata (non reti produzione o di terzi)

- Wireshark con dissettore 802.11 attivo

### Attività

37. Abilitare Monitor Mode: sudo ip link set wlan0 down && sudo iw wlan0 set monitor control && sudo ip link set wlan0 up

38. Avviare Wireshark su wlan0 con filtro: wlan.fc.type_subtype == 8 (beacon)

39. Identificare le reti nel range: SSID, BSSID, canale, tipo di cifratura (RSN IE)

40. Aggiungere filtro per i Probe Request: wlan.fc.type_subtype == 4

41. Osservare quali SSID vengono cercati dai dispositivi nell'area

### Domande

42. Qual è il beacon interval dell'AP di test?

43. Quali velocità sono elencate nel campo Supported Rates del beacon?

44. I frame di dati dei client mostrano il payload in chiaro o cifrato? Come si distingue?

45. È possibile determinare il produttore dell'AP dal BSSID? Come?

# 5. Bibliografia

Riferimenti in formato APA 7ª edizione.

## Reti di calcolatori — Testi di riferimento

Forouzan, B. A. (2021). Data communications and networking (6th ed.). McGraw-Hill Education.

Kurose, J. F., & Ross, K. W. (2022). Computer networking: A top-down approach (8th ed.). Pearson.

Tanenbaum, A. S., & Wetherall, D. J. (2021). Computer networks (6th ed.). Pearson.

## NAT

Egevang, K., & Francis, P. (1994). The IP network address translator (NAT) (RFC 1631). Internet Engineering Task Force. https://doi.org/10.17487/RFC1631

Srisuresh, P., & Holdrege, M. (1999). IP network address translator (NAT) terminology and considerations (RFC 2663). Internet Engineering Task Force. https://doi.org/10.17487/RFC2663

Srisuresh, P., & Egevang, K. (2001). Traditional IP network address translator (traditional NAT) (RFC 3022). Internet Engineering Task Force. https://doi.org/10.17487/RFC3022

Rekhter, Y., Moskowitz, B., Karrenberg, D., de Groot, G. J., & Lear, E. (1996). Address allocation for private internets (RFC 1918). Internet Engineering Task Force. https://doi.org/10.17487/RFC1918

Penno, R., Perreault, S., Boucadair, M., Sivakumar, S., & Naito, K. (2013). Requirements for carrier-grade NAT (CGN) (RFC 6888). Internet Engineering Task Force. https://doi.org/10.17487/RFC6888

Rosenberg, J., Mahy, R., Matthews, P., & Wing, D. (2008). Session traversal utilities for NAT (STUN) (RFC 5389, aggiornato da RFC 8489). Internet Engineering Task Force. https://doi.org/10.17487/RFC5389

## ICMP

Postel, J. (1981). Internet control message protocol (RFC 792). Internet Engineering Task Force. https://doi.org/10.17487/RFC0792

Conta, A., Deering, S., & Gupta, M. (2006). Internet control message protocol (ICMPv6) for the Internet Protocol version 6 (IPv6) specification (RFC 4443). Internet Engineering Task Force. https://doi.org/10.17487/RFC4443

## Wi-Fi e IEEE 802.11

IEEE Computer Society. (2021). IEEE standard for information technology — telecommunications and information exchange between systems — local and metropolitan area networks — specific requirements — part 11: Wireless LAN medium access control (MAC) and physical layer (PHY) specifications (IEEE Std 802.11-2020). Institute of Electrical and Electronics Engineers. https://doi.org/10.1109/IEEESTD.2021.9363693

Gast, M. S. (2013). 802.11ac: A survival guide. O'Reilly Media.

Gast, M. S. (2005). 802.11 wireless networks: The definitive guide (2nd ed.). O'Reilly Media.

## Sicurezza Wi-Fi

Vanhoef, M., & Piessens, F. (2017). Key reinstallation attacks: Forcing nonce reuse in WPA2. Proceedings of the 2017 ACM SIGSAC Conference on Computer and Communications Security, 1313–1328. https://doi.org/10.1145/3133956.3134027

Vanhoef, M., & Ronen, E. (2019). Dragonblood: Analyzing the Dragonfly handshake of WPA3 and EAP-PWD. Proceedings of the 2020 IEEE Symposium on Security and Privacy. https://papers.mathyvanhoef.com/dragonblood.pdf

Wi-Fi Alliance. (2018). WPA3 specification version 1.0. Wi-Fi Alliance. https://www.wi-fi.org/discover-wi-fi/security
