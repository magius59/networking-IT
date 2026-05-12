# Lezione 1 - Introduzione alle reti informatiche e ai fondamenti del networking

# 1. Obiettivi del corso

Lo studio del networking e della cybersecurity richiede la comprensione
di alcuni concetti fondamentali relativi al funzionamento di Internet,
delle reti locali e dei protocolli di comunicazione.

Gli argomenti principali affrontati includono:

- funzionamento di Internet;

- servizi di rete;

- indirizzamento IPv4;

- subnetting;

- DNS e DHCP;

- routing e switching;

- VLAN e trunking;

- access list e filtraggio;

- cattura e analisi del traffico con Wireshark;

- virtualizzazione;

- servizi cloud;

- sicurezza di rete di base.

L\'approccio adottato è orientato alla comprensione pratica delle
tecnologie, più che alla semplice memorizzazione di configurazioni
specifiche.

# 2. Reti informatiche e comunicazione digitale

## 2.1 Definizione di rete

Una rete informatica è un insieme di dispositivi interconnessi in grado
di scambiarsi dati. I dispositivi possono comunicare attraverso mezzi
trasmissivi fisici (cavi in rame, fibra ottica) oppure tramite onde
radio (Wi-Fi, reti cellulari).

I dispositivi tipicamente connessi a una rete includono:

- computer e workstation;

- server;

- router - instradano i pacchetti tra reti diverse;

- switch - interconnettono dispositivi all\'interno della stessa rete
  locale;

- access point - forniscono connettività wireless;

- dispositivi IoT (Internet of Things);

- telefoni IP;

- telecamere IP;

- firewall.

Lo scopo della rete è permettere la comunicazione e la condivisione di
risorse: file, stampanti, accesso a Internet, servizi applicativi.

*Una rete informatica può essere classificata anche in base alla
tecnologia di trasmissione (punto-punto o broadcast), alla topologia
fisica, al protocollo usato e al livello di gestione. La distinzione tra
LAN, MAN e WAN descrive l\'estensione geografica della rete, trattata
più avanti.*

# 3. Link rate e velocita di trasmissione

## 3.1 Definizione di link rate

Il termine link rate (o data rate, o bandwidth) indica la quantità di
bit trasmessi nell\'unita di tempo su un collegamento. E' una proprietà
del mezzo fisico e della tecnologia usata, non del traffico
effettivamente presente.

**Precisazione:** *Nel gergo comune \'banda\' (bandwidth) viene usato
come sinonimo di link rate, ma in senso stretto indica la larghezza di
banda del segnale in Hz. In contesto digitale i due termini sono spesso
usati in modo intercambiabile.*

Le velocità vengono espresse nelle seguenti unità, usando prefissi SI
decimali (potenze di 10):

  -------------------------------------------------------------------------
  **Unita**               **Simbolo**   **Valore**
  ----------------------- ------------- -----------------------------------
  bit per secondo         Bps           1 bit/s

  kilobit per secondo     Kbps          10\^3 bit/s = 1.000 bit/s

  megabit per secondo     Mbps          10\^6 bit/s = 1.000.000 bit/s

  gigabit per secondo     Gbps          10\^9 bit/s

  terabit per secondo     Tbps          10\^12 bit/s
  -------------------------------------------------------------------------

**Attenzione - bit vs byte:** *Nel networking si usano i bit (bps). Nei
file system si usano i byte (B). 1 byte = 8 bit. Una connessione da 100
Mbps trasferisce circa 12,5 MB/s. Confondere le due unità è un errore
molto comune.*

# 4. Evoluzione storica delle connessioni dati

Le prime connessioni dati utilizzavano linee telefoniche analogiche
della rete PSTN (Public Switched Telephone Network). Le velocità erano
estremamente ridotte:

- 240 bps - prime connessioni dati su linea telefonica (anni \'60-\'70);

- 1.200 bps;

- 2.400 bps;

- fino a 56 Kbps con i modem V.90/V.92 (fine anni \'90).

Le comunicazioni avvenivano tramite modem analogici, che convertivano
segnali digitali in segnali audio trasmissibili sulla linea telefonica
(modulazione), e li riconvertivano alla ricezione (demodulazione). Il
termine \'modem\' deriva da MOdulatore-DEModulatore.

Con l\'evoluzione tecnologica si sono diffuse tecnologie
progressivamente più veloci:

- Linee dedicate T1 (1,544 Mbps) e T3 (44,736 Mbps) - standard
  nordamericano; in Europa E1 (2,048 Mbps) ed E3;

- ISDN - fino a 128 Kbps su due canali B da 64 Kbps ciascuno;

- xDSL (ADSL, VDSL) - dati digitali su doppino telefonico, fino a decine
  di Mbps;

- Fibra ottica FTTH - da 100 Mbps fino a 10 Gbps;

- Reti mobili: GPRS/EDGE (2G), UMTS/HSPA (3G), LTE (4G, fino a \~300
  Mbps teorici), NR (5G, fino a \~20 Gbps teorici).

# 5. Commutazione di circuito e commutazione di pacchetto

## 5.1 Circuit switching - Commutazione di circuito

La rete telefonica tradizionale (PSTN) utilizza la commutazione di
circuito. Prima che la comunicazione abbia inizio, viene stabilito un
percorso fisico dedicato tra i due interlocutori, che rimane riservato
per tutta la durata della chiamata.

Storicamente il collegamento avveniva anche manualmente tramite
centralini telefonici, nei quali gli operatori connettevano fisicamente
i circuiti su tavole di commutazione (switchboard).

Caratteristiche principali:

- percorso dedicato per tutta la durata della comunicazione;

- banda riservata - garantita anche se il canale non trasporta dati in
  un dato momento;

- latenza bassa e costante - non vi sono code di attesa;

- comunicazione sincrona;

- spreco di risorse nei periodi di silenzio.

*Il principale svantaggio del circuit switching è l\'inefficienza: la
capacita è riservata anche quando nessuno dei due interlocutori sta
trasmettendo. Questo è accettabile per la voce, ma non ottimale per i
dati, che sono per natura bursty (a raffiche).*

## 

## 5.2 Packet switching - Commutazione di pacchetto

Internet utilizza principalmente la commutazione di pacchetto. I dati
vengono suddivisi in unità discrete chiamate pacchetti, ciascuno dei
quali viene trattato in modo indipendente dalla rete.

Ogni pacchetto contiene:

- header (intestazione): indirizzo IP sorgente, indirizzo IP
  destinazione, protocollo, TTL, checksum e altre informazioni di
  controllo;

- payload (carico utile): i dati dell\'applicazione.

I pacchetti attraversano la rete passando per router intermedi. Ogni
router applica il meccanismo store-and-forward: riceve il pacchetto, ne
legge l\'intestazione, seleziona il percorso migliore tramite algoritmi
di routing e lo invia all\'hop successivo (vale a dire al prossimo
router o al destinatario finale).

Caratteristiche principali:

- nessun percorso dedicato - la banda è condivisa;

- i diversi pacchetti di una stessa sessione possono seguire percorsi
  diversi;

- ordine di arrivo non garantito;

- uso efficiente della banda (multiplexing statistico);

- possibilità di perdita o ritardo in caso di congestione.

**Confronto:** *Il packet switching e molto piu efficiente per il
traffico dati, che e intrinsecamente irregolare. Lo svantaggio è la
variabilità della latenza (jitter), che deve essere gestita dalle
applicazioni real-time (ad esempio telefonia VOIP).*

## 5.3 Componenti del ritardo (delay)

Il ritardo introdotto dai router si compone di quattro componenti:

- Propagation delay: tempo fisico di percorrenza del mezzo (\~200.000
  km/s nella fibra);

- Transmission delay: tempo per immettere i bit sul mezzo = dimensione
  pacchetto / link rate;

- Processing delay: tempo del router per leggere l\'header e determinare
  l\'uscita;

- Queueing delay: tempo di attesa in coda - il componente piu variabile,
  causa del jitter.

# 6. Multiplexing

## 6.1 Time Division Multiplexing (TDM)

Il Time Division Multiplexing (TDM) consente a più comunicazioni di
condividere lo stesso mezzo trasmissivo dividendo il tempo in slot
periodici e assegnando ciascuno slot a un canale specifico.

Tecnologie che hanno usato TDM:

- reti telefoniche digitali (PDH, SDH/SONET);

- ISDN;

- GSM - ogni frequenza radio e suddivisa in 8 slot TDM.

**FDM e OFDM:** *Il Frequency Division Multiplexing (FDM) divide il
mezzo in sottobande di frequenza, ciascuna assegnata a un canale. Le
reti Wi-Fi e 4G/5G usano varianti avanzate dette OFDM (Orthogonal FDM) e
OFDMA.*

## 6.2 Software Defined Radio (SDR) e analisi GSM

Una Software Defined Radio (SDR) è un sistema radio in cui le funzioni
tradizionalmente implementate in hardware analogico vengono eseguite via
software. Con strumenti SDR e possibile:

- osservare i canali di controllo GSM (BCCH, RACH, PCH);

- identificare celle radio e relativi parametri (MCC, MNC, LAC, Cell
  ID);

- visualizzare informazioni di segnalazione;

- analizzare protocolli tramite Wireshark con plugin opportuni.

Le comunicazioni vocali GSM adottano cifratura A5/1 o A5/3 (KASUMI) che
rendono molto più complessa - ma non impossibile - l\'intercettazione
del contenuto.

**Nota legale:** *L\'intercettazione non autorizzata di comunicazioni e
illegale in quasi tutte le giurisdizioni. L\'analisi SDR a scopo
didattico deve limitarsi ai canali di segnalazione pubblici.*

# 7. Protocolli di trasporto: UDP e TCP

## 7.1 UDP - User Datagram Protocol

UDP e un protocollo di trasporto connectionless definito nell\'RFC 768
(1980). Non viene stabilita alcuna sessione preliminare tra mittente e
destinatario prima dell\'invio dei dati.

Caratteristiche principali:

- nessun handshake preliminare;

- nessuna conferma di ricezione (ACK);

- nessun meccanismo di ritrasmissione in caso di perdita;

- nessun controllo della congestione;

- overhead dell\'header ridotto - 8 byte contro i 20+ byte di TCP;

- include checksum opzionale (obbligatorio in IPv6).

UDP viene utilizzato in applicazioni nelle quali la rapidità è più
importante della consegna garantita:

- streaming audio/video (RTP/RTSP);

- VoIP;

- videoconferenze;

- gaming online;

- DNS (query);

- DHCP;

- SNMP;

- TFTP.

## 7.2 TCP - Transmission Control Protocol (cenno)

TCP (RFC 793) e un protocollo connection-oriented che garantisce:

- consegna affidabile e ordinata dei segmenti;

- ritrasmissione automatica in caso di perdita;

- controllo del flusso tramite finestra scorrevole;

- controllo della congestione (slow start, AIMD);

- multiplexing tramite numeri di porta.

## 7.3 Circuiti virtuali

Un circuito virtuale è una comunicazione logica in cui i pacchetti
seguono un percorso predefinito. Possono essere permanenti (PVC) o
commutati (SVC). Esempi di tecnologie:

- ATM (Asynchronous Transfer Mode);

- Frame Relay;

- MPLS - ancora molto usato nelle WAN enterprise.

# 8. Topologie di rete

La topologia di rete descrive la struttura con cui i dispositivi sono
interconnessi. Si distingue la topologia fisica dalla topologia logica
(percorso dei dati).

## 8.1 Topologia a bus

Tutti i dispositivi condividono un unico mezzo trasmissivo. Un guasto al
cavo interrompe l\'intera rete. La gestione delle collisioni era
affidata a CSMA/CD. Tecnologia esclusivamente storica nelle reti
cablate.

## 8.2 Topologia ad anello

Ogni dispositivo e connesso a due vicini, formando un percorso
circolare. L\'anello doppio garantisce ridondanza.

Tecnologie storiche:

- Token Ring (IEEE 802.5) - IBM, anni \'80-\'90;

- FDDI - fibra a 100 Mbps, anni \'90;

- SDH/SONET - usano anelli con protezione automatica APS in ambito
  carrier.

## 8.3 Topologia a stella

Tutti i dispositivi sono connessi a un punto centrale (switch). E la
topologia dominante nelle reti Ethernet moderne.

Vantaggi:

- isolamento dei guasti: un cavo rotto non compromette la rete;

- gestione semplificata;

- comunicazione tra due nodi non richiede la collaborazione degli altri.

Svantaggio principale: lo switch centrale e un single point of failure.

**Hub vs Switch:** *Con un hub il traffico viene replicato su tutte le
porte (dominio di collisione condiviso). Con uno switch ogni porta e un
dominio di collisione separato e il traffico unicast è inoltrato solo
verso la porta di destinazione.*

## 8.4 Altre topologie

- Maglia (mesh): ogni nodo è connesso a più nodi. Alta ridondanza. Full
  mesh con n nodi richiede n(n-1)/2 collegamenti.

- Albero (gerarchica): combinazione di stelle su più livelli. Tipica
  delle reti aziendali (accesso, distribuzione, core).

# 9. Ethernet storica e reti coassiali

## 9.1 10Base5, 10Base2 e 10Base-T

  -------------------------------------------------------------------------------------
  **Standard**   **Cavo**       **Velocita**   **Lunghezza   **Note**
                                               max**         
  -------------- -------------- -------------- ------------- --------------------------
  10Base5        Coassiale RG-8 10 Mbps        500 m         Thick Ethernet; Vampire
                 (thick)                                     Tap

  10Base2        Coassiale      10 Mbps        185 m         Thin Ethernet; connettori
                 RG-58 (thin)                                BNC

  10Base-T       Doppino UTP    10 Mbps        100 m         Prima Ethernet su coppie
                 Cat3                                        intrecciate
  -------------------------------------------------------------------------------------

La codifica usata era Manchester encoding. La gestione delle collisioni
era affidata a CSMA/CD (Carrier Sense Multiple Access with Collision
Detection).

## Manchester Encoding

Il *Manchester Encoding* è una tecnica di codifica di linea utilizzata
nelle comunicazioni digitali per rappresentare i bit attraverso
transizioni del segnale elettrico.

A differenza di una codifica semplice in cui un livello alto rappresenta
1 e un livello basso rappresenta 0, nel Manchester Encoding ogni bit
contiene sempre una variazione di stato al centro dell'intervallo
temporale del bit stesso.

Questo permette al ricevitore di sincronizzare il clock direttamente
osservando il segnale ricevuto, senza necessità di un canale di clock
separato.

### Principio di funzionamento

Nella convenzione più comune (IEEE 802.3 Ethernet classica):

transizione da basso ad alto → bit 1

transizione da alto a basso → bit 0

Esempio semplificato:

Bit: 1 0 1 1

Segnale: \_\|‾ ‾\|\_ \_\|‾ \_\|‾

Ogni bit:

- occupa un intervallo temporale fisso

- contiene obbligatoriamente una transizione centrale

Vantaggi

- Autosincronizzazione del clock

- Facilità di recupero del segnale

- Assenza di componente continua (DC balance)

- Maggiore affidabilità nelle trasmissioni

### Svantaggi

- Richiede banda doppia rispetto a codifiche NRZ

- Maggiore numero di transizioni elettriche

- Minore efficienza spettrale

### Utilizzi storici e pratici

- Il Manchester Encoding è stato utilizzato in:

- Ethernet 10BASE-T e 10BASE5

- RFID

- sistemi industriali

- comunicazioni seriali sincrone

Nelle reti Ethernet moderne ad alta velocità vengono utilizzate
codifiche più efficienti,

ma il Manchester Encoding rimane fondamentale dal punto di vista
didattico per comprendere:

- sincronizzazione

- codifica fisica

- trasmissione digitale dei dati.

## 9.2 Terminatori e Vampire Tap

Le reti coassiali richiedevano terminatori a 50 Ohm alle estremità del
cavo. Senza terminatore il segnale veniva riflesso, causando collisioni
continue.

Nelle reti 10Base5, i Vampire Tap perforavano fisicamente il cavo
coassiale per prelevare il segnale senza interrompere il bus. Il cavo di
collegamento tra il Vampire Tap e la scheda di rete si chiamava AUI
(Attachment Unit Interface), poteva essere lungo al massimo 50 metri.

+-----------------------------------------------------------------------+
| **Esercizio 1 - Analisi di una topologia a stella**                   |
+=======================================================================+
| Obiettivo: comprendere il funzionamento di una rete Ethernet moderna. |
|                                                                       |
| Scenario: una rete locale contiene:                                   |
|                                                                       |
| - 1 switch centrale;                                                  |
|                                                                       |
| - 5 computer connessi allo switch;                                    |
|                                                                       |
| - 1 stampante di rete, connessa allo switch;                          |
|                                                                       |
| - 1 access point Wi-Fi, onnesso allo switch.                          |
|                                                                       |
| Domande:                                                              |
|                                                                       |
| - Quale dispositivo rappresenta il centro della stella?               |
|                                                                       |
| - Cosa succede se un singolo cavo Ethernet si interrompe?             |
|                                                                       |
| - Cosa succede se si guasta lo switch centrale?                       |
|                                                                       |
| - Quali vantaggi offre la topologia a stella rispetto a una topologia |
|   a bus?                                                              |
|                                                                       |
| - Quante connessioni fisiche punto-punto esistono in questa rete?     |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
| **Soluzione**                                                         |
+=======================================================================+
| **1. Centro della stella:** Lo switch centrale. Tutti i dispositivi   |
| (5 computer + stampante + access point) sono collegati direttamente   |
| allo switch tramite un cavo dedicato.                                 |
|                                                                       |
| **2. Interruzione di un singolo cavo:** Solo il dispositivo collegato |
| a quel cavo perde la connettivita. Gli altri 6 dispositivi continuano |
| a comunicare normalmente.                                             |
|                                                                       |
| **3. Guasto dello switch centrale:** Tutti i dispositivi perdono      |
| simultaneamente la connettivita. Lo switch e un single point of       |
| failure. In ambienti critici si usa ridondanza (dual uplink, switch   |
| in failover).                                                         |
|                                                                       |
| **4. Vantaggi rispetto al bus:**                                      |
|                                                                       |
| - Isolamento dei guasti: un cavo rotto non abbatte l\'intera rete     |
|   (nel bus sì).                                                       |
|                                                                       |
| - Nessuna collisione: ogni porta dello switch e un dominio di         |
|   collisione separato.                                                |
|                                                                       |
| - Full-duplex: comunicazione simultaneamente bidirezionale su ogni    |
|   collegamento.                                                       |
|                                                                       |
| - Gestione semplice: aggiungere o rimuovere nodi non richiede         |
|   interventi sugli altri.                                             |
|                                                                       |
| **5. Numero di connessioni fisiche:** 7 (una per ogni dispositivo     |
| periferico: 5 computer + 1 stampante + 1 access point). Nella         |
| topologia a stella il numero di cavi e sempre uguale al numero di     |
| dispositivi periferici.                                               |
+-----------------------------------------------------------------------+

# 10. LAN e WAN

## 10.1 LAN - Local Area Network

Una LAN e una rete locale caratterizzata da:

- bassa latenza - tipicamente \< 1 ms tra dispositivi locali;

- elevata velocita - da 100 Mbps a 10+ Gbps;

- comunicazione diretta tra dispositivi senza attraversare router;

- dominio di broadcast condiviso (modificabile tramite VLAN);

- gestita da un unico soggetto amministrativo.

## 10.2 MAN - Metropolitan Area Network

Una MAN copre un\'area geografica più ampia della LAN ma più limitata
della WAN: tipicamente una citta o un campus distribuito. Può utilizzare
fibra ottica dedicata o Metro Ethernet.

## 10.3 WAN - Wide Area Network

Una WAN collega siti geograficamente distanti. Le tecnologie usate
includono:

- Internet (rete pubblica globale);

- collegamenti dedicati (leased line) - T1, E1, fibra dedicata;

- VPN site-to-site - tunnel cifrati su Internet;

- MPLS - WAN private con QoS garantita;

- SD-WAN - approccio moderno che astrae la connettività WAN via
  software.

# 11. Evoluzione di Internet

## 11.1 ARPANET

ARPANET (Advanced Research Projects Agency Network) fu la prima rete a
commutazione di pacchetto su larga scala, finanziata da DARPA. Il primo
messaggio fu inviato il 29 ottobre 1969 tra l\'UCLA e lo Stanford
Research Institute.

**Nota: *DARPA* - *Defense Advanced Research Projects Agency***

*È un'agenzia del Dipartimento della Difesa degli Stati Uniti fondata
nel 1958, durante la Guerra Fredda, in risposta al lancio sovietico
dello Sputnik.Il suo obiettivo era finanziare e coordinare ricerca
tecnologica avanzata con potenziali applicazioni militari e
strategiche.*

11.2 Dal NCP a TCP/IP

ARPANET usava inizialmente il protocollo NCP (Network Control Program).
Nel 1983 adottò ufficialmente TCP/IP, progettato da Vint Cerf e Robert
Kahn (specifica originale: RFC 675, 1974). Il DNS fu introdotto nel
1983-1984 (RFC 882 e 883) per sostituire il file HOSTS.TXT distribuito
manualmente.

## 11.3 Crescita di Internet

Con l'affermarsi del World Wide Web (Tim Berners-Lee, CERN, 1991)
Internet si espanse verso il pubblico generale. Oggi include:

- server web, CDN;

- cloud services (IaaS, PaaS, SaaS);

- social network;

- API REST e WebSocket;

- container e orchestratori (Kubernetes);

- macchine virtuali;

- piattaforme IoT distribuite.

# 12. TCP/IP e indirizzamento IPv4

## 12.1 Struttura di un indirizzo IPv4

Un indirizzo IPv4 e composto da 32 bit, rappresentati in notazione
decimale puntata (*dotted decimal notation*): quattro numeri decimali
separati da punti, ciascuno dei quali rappresenta un ottetto (8 bit) con
valore da 0 a 255.

Esempio: 192.168.10.12

In binario: 11000000.10101000.00001010.00001100

## 12.2 Subnet mask e notazione CIDR

La subnet mask e una sequenza di 32 bit in cui i bit a 1 identificano la
parte rete e i bit a 0 la parte host. Viene applicata tramite AND bit a
bit tra l\'indirizzo IP e la mask.

  ------------------------------------------------------------------------
  **Campo**      **Decimale**      **Binario**
  -------------- ----------------- ---------------------------------------
  Indirizzo IP   192.168.10.12     11000000.10101000.00001010.00001100

  Subnet mask    255.255.255.0     11111111.11111111.11111111.00000000

  AND (rete)     192.168.10.0      11000000.10101000.00001010.00000000
  ------------------------------------------------------------------------

Notazione CIDR: 192.168.10.12/24 - i primi 24 bit sono bit di rete.

La notazione CIDR fu introdotta nel 1993 (RFC 1517-1520) per sostituire
il sistema classful.

## 12.3 Conversione binario-decimale

Ogni posizione in un byte rappresenta una potenza di 2, dalla più
significativa (MSB, valore 128) a sinistra alla meno significativa (LSB,
valore 1) a destra.

  -------------------------------------------------------------------------------
  **Posizione**   **7**   **6**   **5**   **4**   **3**   **2**   **1**   **0**
  --------------- ------- ------- ------- ------- ------- ------- ------- -------
  Valore (2\^n)   128     64      32      16      8       4       2       1

  Esempio: 192    1       1       0       0       0       0       0       0
  -------------------------------------------------------------------------------

192 in binario = 11000000 perche 128 + 64 = 192.

+-----------------------------------------------------------------------+
| **Esercizio 2 - Conversione binario/decimale**                        |
+=======================================================================+
| Convertire i seguenti valori binari in decimale:                      |
|                                                                       |
| - 10101000                                                            |
|                                                                       |
| - 11111111                                                            |
|                                                                       |
| - 00010001                                                            |
|                                                                       |
| Convertire i seguenti valori decimali in binario (8 bit):             |
|                                                                       |
| - 35                                                                  |
|                                                                       |
| - 17                                                                  |
|                                                                       |
| - 192                                                                 |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
| **Soluzione**                                                         |
+=======================================================================+
| Da binario a decimale:                                                |
|                                                                       |
| 10101000: posizioni attive 7, 5, 3 -\> 128 + 32 + 8 = **168**         |
|                                                                       |
| 11111111: tutti i bit a 1 -\> 128+64+32+16+8+4+2+1 = **255** (massimo |
| valore di un ottetto)                                                 |
|                                                                       |
| 00010001: posizioni attive 4 e 0 -\> 16 + 1 = **17**                  |
|                                                                       |
| Da decimale a binario:                                                |
|                                                                       |
| **35:** 32 + 2 + 1 -\> posizioni 5, 1, 0 -\> 00100011                 |
|                                                                       |
| **17:** 16 + 1 -\> posizioni 4 e 0 -\> 00010001                       |
|                                                                       |
| **192:** 128 + 64 -\> posizioni 7 e 6 -\> 11000000                    |
+-----------------------------------------------------------------------+

# 13. Indirizzi IPv4 speciali

## 13.1 Indirizzo di rete e broadcast

  -----------------------------------------------------------------------
  **Indirizzo**       **Descrizione**             **Esempio (/24)**
  ------------------- --------------------------- -----------------------
  Indirizzo di rete   Parte host con **tutti i    192.168.10.0
                      bit a 0**. Identifica la    
                      subnet.                     

  Broadcast diretto   Parte host con **tutti i    192.168.10.255
                      bit a 1**. Raggiunge tutti  
                      gli host della subnet.      
  -----------------------------------------------------------------------

Altri indirizzi speciali importanti:

  -----------------------------------------------------------------------
  **Indirizzo**       **Scopo**
  ------------------- ---------------------------------------------------
  0.0.0.0/0           Route di default (\'qualsiasi destinazione\')

  127.0.0.0/8         Loopback - 127.0.0.1 e il loopback locale
                      (localhost). Non esce sull\'interfaccia.

  169.254.0.0/16      APIPA - assegnato se il DHCP non risponde.

  255.255.255.255     Broadcast limitato - inviato a tutti gli host del
                      segmento locale.

  224.0.0.0/4         Multicast - range riservato (RFC 1112).
  -----------------------------------------------------------------------

## 13.2 Classi IPv4 storiche (Classful)

  -------------------------------------------------------------------------------------
  **Classe**   **Primo     **Bit        **Maschera      **Host per rete**
               ottetto**   iniziali**   default**       
  ------------ ----------- ------------ --------------- -------------------------------
  A            1-126       0xxx         255.0.0.0 (/8)  16.777.214

  B            128-191     10xx         255.255.0.0     65.534
                                        (/16)           

  C            192-223     110x         255.255.255.0   254
                                        (/24)           

  D            224-239     1110         \- (multicast)  \-

  E            240-255     1111         \- (riservata)  \-
  -------------------------------------------------------------------------------------

**Nota:** *127.x.x.x è nella classe A ma è riservato al loopback; si
tratta di un'interfaccia virtuale che restituisce ogni pacchetto che le
viene inviato.*

## 13.3 Indirizzi privati (RFC 1918)

  --------------------------------------------------------------------------
  **Range**                 **CIDR**         **Indirizzi disponibili**
  ------------------------- ---------------- -------------------------------
  10.0.0.0 - 10.255.255.255 10.0.0.0/8       16.777.216

  172.16.0.0 -              172.16.0.0/12    1.048.576
  172.31.255.255                             

  192.168.0.0 -             192.168.0.0/16   65.536
  192.168.255.255                            
  --------------------------------------------------------------------------

# 14. NAT - Network Address Translation

Gli host con indirizzi privati non possono comunicare direttamente su
Internet. Il NAT, eseguito dal router, risolve questo problema
traducendo gli indirizzi.

Flusso tipico di una connessione in uscita con PAT (NAT overload):

1.  Il client interno (es. 192.168.1.10:50000) invia un pacchetto verso
    Internet dal proprio indirizzo e la porta associata (50000).

2.  Il router sostituisce l\'indirizzo sorgente privato con il proprio
    IP pubblico e una porta disponibile sulla propria interfaccia
    esterna (es. 203.0.113.1:40001). Salva la mappatura nella tabella
    NAT.

3.  Il server remoto risponde all\'IP pubblico 203.0.113.1:40001.

4.  Il router consulta la tabella NAT, trova la mappatura e reinoltra il
    pacchetto al client interno (in sostanza, modifica il pacchetto di
    ritorno dal server remoto cambiando l'indirizzo di destinazione da
    203.0.113.1:40001 a 192.168.1.10:50000).

Tipi di NAT:

- NAT statico (one-to-one): un IP privato e mappato permanentemente a un
  IP pubblico.

- NAT dinamico: pool di IP pubblici assegnati dinamicamente.

- PAT / NAT overload: molti IP privati condividono un unico IP pubblico
  tramite porte diverse. E il tipo usato in quasi tutte le reti
  domestiche.

**Implicazione per la sicurezza:** *Il NAT non è un firewall, ma offre
un isolamento implicito: i pacchetti che da Internet raggiungono
l'interfaccia esterna del router che non corrispondono a sessioni attive
vengono scartati perché non esiste una voce nella tabella NAT.*

+-----------------------------------------------------------------------+
| **Esercizio 3 - Identificazione della rete**                          |
+=======================================================================+
| Dato il seguente host:                                                |
|                                                                       |
| **IP Address:** 192.168.30.12                                         |
|                                                                       |
| **Subnet Mask:** 255.255.255.0                                        |
|                                                                       |
| Determinare:                                                          |
|                                                                       |
| - Indirizzo di rete                                                   |
|                                                                       |
| - Indirizzo broadcast                                                 |
|                                                                       |
| - Primo host utilizzabile                                             |
|                                                                       |
| - Ultimo host utilizzabile                                            |
|                                                                       |
| - Numero massimo di host disponibili                                  |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
| **Soluzione**                                                         |
+=======================================================================+
| La subnet mask 255.255.255.0 corrisponde a /24: i primi 24 bit sono   |
| la parte rete, gli ultimi 8 bit la parte host.                        |
|                                                                       |
| **1. Indirizzo di rete:** 192.168.30.0 (parte host = 00000000)        |
|                                                                       |
| **2. Indirizzo broadcast:** 192.168.30.255 (parte host = 11111111)    |
|                                                                       |
| **3. Primo host utilizzabile:** 192.168.30.1 (indirizzo di rete + 1)  |
|                                                                       |
| **4. Ultimo host utilizzabile:** 192.168.30.254 (broadcast - 1)       |
|                                                                       |
| **5. Numero massimo di host:** 2\^8 - 2 = 256 - 2 = **254 host**      |
|                                                                       |
| Formula generale: per una subnet /\<n\>, il numero di host            |
| disponibili e 2\^(32-n) - 2.                                          |
+-----------------------------------------------------------------------+

# 15. Scansione di rete

La scansione di rete (network scanning) è il processo di identificazione
sistematica degli host attivi, dei servizi in ascolto e delle
vulnerabilita presenti su una rete.

Tipi principali di scansione:

- Host discovery: identifica quali indirizzi IP sono attivi tramite ICMP
  echo request, ARP request o pacchetti TCP/UDP sonda.

- Port scan: identifica quali porte TCP/UDP sono in ascolto. Varianti:
  TCP connect, TCP SYN (stealth), UDP scan.

- Service detection: identifica servizio e versione tramite banner
  grabbing o fingerprinting.

- OS fingerprinting: stima il sistema operativo analizzando TTL, TCP
  window size, opzioni TCP.

- Vulnerability scan: identifica vulnerabilità note nei servizi rilevati
  su un host.

Strumenti comuni:

- Nmap - scanner di rete open source, standard de facto;

- Masscan - scansione massiva ad alta velocita;

- Nessus / OpenVAS - vulnerability scanner;

- Shodan - motore di ricerca per dispositivi connessi a Internet.

**Nota etica e legale:** *La scansione di rete su sistemi non propri o
senza autorizzazione scritta e illegale in molte giurisdizioni. In
Italia configura reati informatici ai sensi degli artt. 615-ter e
seguenti del Codice Penale. In ambito professionale si opera sempre con
un documento di autorizzazione.*

# Bibliografia

Le opere sono organizzate per argomento e citate in formato APA 7a
edizione.

## Reti informatiche - testi di riferimento

Tanenbaum, A. S., & Wetherall, D. J. (2011). *Computer networks* (5th
ed.). Prentice Hall.

Kurose, J. F., & Ross, K. W. (2021). *Computer networking: A top-down
approach* (8th ed.). Pearson.

Forouzan, B. A. (2013). *Data communications and networking* (5th ed.).
McGraw-Hill.

## Protocolli Internet - RFC fondamentali

Postel, J. (1980). *User Datagram Protocol* (RFC 768). IETF.
https://www.rfc-editor.org/rfc/rfc768

Postel, J. (1981). *Internet Protocol* (RFC 791). IETF.
https://www.rfc-editor.org/rfc/rfc791

Postel, J. (1981). *Transmission Control Protocol* (RFC 793). IETF.
https://www.rfc-editor.org/rfc/rfc793

Rekhter, Y., Moskowitz, B., Karrenberg, D., de Groot, G. J., & Lear, E.
(1996). *Address allocation for private internets* (RFC 1918). IETF.
https://www.rfc-editor.org/rfc/rfc1918

Fuller, V., & Li, T. (2006). *Classless inter-domain routing (CIDR)*
(RFC 4632). IETF. https://www.rfc-editor.org/rfc/rfc4632

Srisuresh, P., & Holdrege, M. (1999). *IP network address translator
(NAT) terminology and considerations* (RFC 2663). IETF.
https://www.rfc-editor.org/rfc/rfc2663

## Storia di Internet e ARPANET

Cerf, V., & Kahn, R. E. (1974). A protocol for packet network
intercommunication. *IEEE Transactions on Communications, 22*(5),
637-648. https://doi.org/10.1109/TCOM.1974.1092259

Leiner, B. M., Cerf, V. G., Clark, D. D., Kahn, R. E., Kleinrock, L.,
Lynch, D. C., Postel, J., Roberts, L. G., & Wolff, S. (2009). A brief
history of the Internet. *ACM SIGCOMM Computer Communication Review,
39*(5), 22-31. https://doi.org/10.1145/1629607.1629613

## Indirizzamento IP e subnetting

Lammle, T. (2019). *CCNA Cisco Certified Network Associate study guide*
(8th ed.). Sybex.

Malone, D., & Maier, G. (2010). IPv4 address allocation: The end is
nigh. *IEEE Internet Computing, 14*(3), 12-18.
https://doi.org/10.1109/MIC.2010.51

## Commutazione e multiplexing

Kleinrock, L. (1964). *Communication nets: Stochastic message flow and
delay*. McGraw-Hill.

Roberts, L. G. (1978). The evolution of packet switching. *Proceedings
of the IEEE, 66*(11), 1307-1313. https://doi.org/10.1109/PROC.1978.11141

## Ethernet e reti locali

Metcalfe, R. M., & Boggs, D. R. (1976). Ethernet: Distributed packet
switching for local computer networks. *Communications of the ACM,
19*(7), 395-404. https://doi.org/10.1145/360248.360253

IEEE. (2022). *IEEE 802.3-2022: IEEE standard for Ethernet*. IEEE.
https://doi.org/10.1109/IEEESTD.2022.9844436

## Sicurezza e scansione di rete

Lyon, G. F. (2008). *Nmap network scanning: The official Nmap project
guide*. Insecure.com LLC. https://nmap.org/book/

Stallings, W. (2022). *Network security essentials: Applications and
standards* (7th ed.). Pearson.

**Nota:** *Gli RFC citati sono standard tecnici pubblicamente
accessibili sul sito dell\'IETF (https://www.rfc-editor.org).
Costituiscono la fonte primaria per la specifica dei protocolli
Internet.*
