**Lezione 5**

Configurazione dei Router, IPv4, QoS e Fondamenti di Routing Avanzato

# 1. Configurazione di base dei router Cisco

Un router e un dispositivo di rete che opera al livello 3 del modello OSI (livello Network). Il suo compito fondamentale e ricevere pacchetti IP da una rete e inoltrarli verso reti diverse in base all'indirizzo IP di destinazione, decapsulando il frame Layer 2 in ingresso e riencapsulando il pacchetto in un nuovo frame Layer 2 per il link di uscita.

La configurazione di un router Cisco IOS condivide la struttura gerarchica delle modalità operative descritta per gli switch (Lezione 4): User EXEC, Privileged EXEC, Global Configuration, Interface Configuration. In questa lezione ci concentriamo sugli aspetti specifici dei router, con particolare attenzione alla gestione delle interfacce, alla sicurezza delle credenziali e alla configurazione del routing.

## 1.1 Gestione dell'hostname e importanza operativa

Il comando 'hostname' modifica il nome visualizzato nel prompt CLI del dispositivo. Sebbene non abbia impatto sul routing o sull'indirizzamento IP, e un elemento critico per l'amministrazione operativa.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><p>Router&gt; enable</p>
<p>Router# configure terminal</p>
<p>Router(config)# hostname R1</p>
<p>R1(config)# ! il prompt cambia immediatamente</p></td>
</tr>
</tbody>
</table>

In ambienti enterprise con decine o centinaia di apparati nello stesso rack o sistema di gestione, l'hostname permette di:

- Identificare immediatamente il dispositivo durante una sessione SSH o console.

- Distinguere tra apparati dello stesso modello (es. R1-CORE-Milano vs R2-DISTRIB-Roma).

- Correlare i log di sistema: syslog e SNMP trap includono l'hostname nel messaggio.

- Popolare automaticamente il prompt nei sistemi di gestione (NMS, Ansible, Netconf/YANG).

**Nota su DNS:** *Il campo 'hostname' di IOS non registra automaticamente il dispositivo nel DNS. Per la risoluzione DNS dell'hostname dell'apparato e necessario configurare esplicitamente il server DNS (ip name-server) e, se richiesto, un record A/PTR nel server DNS. L'hostname influisce però sulla generazione del Subject nelle chiavi RSA per SSH (formato 'hostname.domain').*

## 1.2 Sicurezza delle password: enable password vs enable secret

La protezione dell'accesso alla modalità Privileged EXEC è critica perché da quella modalità è possibile visualizzare e modificare l'intera configurazione del dispositivo. Cisco IOS offre due comandi con comportamenti crittografici radicalmente diversi.

|  |  |  |  |  |
|----|----|----|----|----|
| **Comando** | **Memorizzazione** | **Algoritmo** | **Sicurezza** | **Uso raccomandato** |
| enable password \<pwd\> | Testo in chiaro nella config (o cifratura tipo 7 con 'service password-encryption') | XOR con chiave fissa (tipo 7) — banalmente reversibile | Molto bassa: recuperabile in secondi con tool online | Mai — non usare |
| enable secret \<pwd\> | Hash nella running/startup config | MD5 (\$1\$) per default; SCRYPT (\$9\$) e PBKDF2 (\$8\$) disponibili da IOS-XE | Accettabile (MD5 debole ma non reversibile direttamente; usare \$8\$ o \$9\$ per nuove config) | Sempre — e il comando corretto |

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><p>! ERRATO - non usare</p>
<p>enable password cisco123</p>
<p>! CORRETTO - hash MD5 (minimo accettabile)</p>
<p>enable secret cisco123</p>
<p>! OTTIMALE - hash SCRYPT su IOS-XE 16.x+</p>
<p>enable algorithm-type scrypt secret cisco123</p>
<p>! oppure PBKDF2:</p>
<p>enable algorithm-type sha256 secret cisco123</p>
<p>! Verifica: nella running-config apparira come:</p>
<p>! enable secret 9 $9$... (SCRYPT) -- NON reversibile</p>
<p>! Cifratura debole di tutte le password in chiaro (tipo 7)</p>
<p>service password-encryption</p>
<p>! Attenzione: tipo 7 e reversibile, non e sicurezza reale.</p>
<p>! Usare solo come deterrente visivo per 'shoulder surfing'.</p></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><strong>Approfondimento — Tipi di cifratura password in Cisco IOS</strong></td>
</tr>
<tr>
<td><p>Cisco IOS identifica il tipo di hash con un numero che precede il valore cifrato nella configurazione. Conoscere questi tipi e fondamentale per valutare la resistenza della configurazione a un attacco di credential theft.</p>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<tbody>
<tr>
<td><p>Tipo 0: password in chiaro (es. password 0 cisco)</p>
<p>Tipo 7: XOR con chiave fissa ASCII. Reversibile banalmente.</p>
<p>Esempio: password 7 0822455D0A16 -&gt; decodifica -&gt; 'cisco'</p>
<p>Tool: http://www.ifm.net.nz/cookbooks/passwordcracker.html</p>
<p>Tipo 5: MD5 crypt ($1$salt$hash). Usato da 'enable secret' tradizionale.</p>
<p>Non reversibile, ma vulnerabile a dizionario/brute force su GPU.</p>
<p>Esempio: enable secret 5 $1$mERr$hx5rVt7rPNoS4wqbXKX7m0</p>
<p>Tipo 8: PBKDF2-SHA-256 con 20000 iterazioni. Resistente a GPU.</p>
<p>Disponibile da IOS 15.3(3)M+ e IOS-XE 3.10+.</p>
<p>Esempio: enable secret 8 $8$...</p>
<p>Tipo 9: SCRYPT (N=16384, r=1, p=1). Il più resistente disponibile su IOS.</p>
<p>Disponibile da IOS-XE 16.x+.</p>
<p>Esempio: enable secret 9 $9$...</p></td>
</tr>
</tbody>
</table>
<p>Raccomandazione: in nuove installazioni su piattaforme che lo supportano, usare sempre tipo 9 (SCRYPT) o tipo 8 (PBKDF2). Il tipo 5 (MD5) rimane accettabile ma subottimale. Il tipo 7 non offre protezione reale e deve essere considerato testo in chiaro a tutti gli effetti operativi.</p></td>
</tr>
</tbody>
</table>

## 1.3 Accesso da console e da rete (VTY)

Un router Cisco dispone di più tipi di linee di accesso. Ogni linea può essere configurata con modalità di autenticazione e trasporto differenti.

|  |  |  |  |
|----|----|----|----|
| **Linea** | **Accesso fisico** | **Protocollo** | **Uso tipico** |
| Console (CON 0) | Cavo console RJ45-DB9 o USB-A | Locale - nessun protocollo di rete | Configurazione iniziale, recovery, accesso out-of-band |
| Auxiliary (AUX 0) | Porta AUX, spesso collegata a modem analogico | Modem analogico | Accesso di emergenza dial-up (raro oggi) |
| VTY 0-15 | Rete IP (qualsiasi interfaccia attiva) | Telnet (TCP 23) o SSH (TCP 22) | Gestione remota ordinaria |

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><p>! --- Console: timeout e password ---</p>
<p>line console 0</p>
<p>password console-pwd-sicura</p>
<p>login</p>
<p>exec-timeout 5 0 ! disconnetti dopo 5 min di inattivita</p>
<p>logging synchronous ! evita che i log interrompano l'input</p>
<p>! --- VTY: solo SSH, autenticazione locale ---</p>
<p>line vty 0 15</p>
<p>transport input ssh ! disabilita Telnet esplicitamente</p>
<p>login local ! usa il database utenti locale</p>
<p>exec-timeout 10 0</p>
<p>! --- Utente locale con privilegi completi ---</p>
<p>username admin privilege 15 algorithm-type scrypt secret AdminPass123!</p></td>
</tr>
</tbody>
</table>

**Nota sul cavo console:** *Il cavo console tradizionale ha un connettore RJ45 da un lato (porta console del router) e DB9 (RS-232, porta COM) dall'altro. Poichè i PC moderni non hanno più la porta seriale, si usano adattatori USB-to-Serial (chipset FTDI o Prolific). La velocita di default è 9600 bps, 8N1 (8 bit, nessuna parita, 1 stop bit). Software di emulazione terminale: PuTTY (Windows), minicom/screen (Linux), Serial (macOS).*

## 1.4 Configurazione delle interfacce

Le interfacce fisiche di un router Cisco sono inizialmente in stato 'administratively down' (comando 'shutdown' applicato per default). Ogni interfaccia deve ricevere un indirizzo IP e essere abilitata esplicitamente.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><p>! Configurazione interfaccia GigabitEthernet verso LAN locale</p>
<p>interface GigabitEthernet 0/0</p>
<p>description LAN-192.168.0.0/24</p>
<p>ip address 192.168.0.1 255.255.255.0</p>
<p>no shutdown</p>
<p>! Configurazione interfaccia GigabitEthernet verso rete WAN/ISP</p>
<p>interface GigabitEthernet 0/1</p>
<p>description UPLINK-WAN-ISP</p>
<p>ip address 203.0.113.1 255.255.255.252</p>
<p>no shutdown</p>
<p>! Interfaccia seriale (WAN legacy, es. leased line T1)</p>
<p>interface Serial 0/0/0</p>
<p>ip address 10.0.0.1 255.255.255.252</p>
<p>encapsulation ppp</p>
<p>clock rate 2000000 ! solo lato DCE della connessione seriale</p>
<p>no shutdown</p>
<p>! Salvataggio della configurazione</p>
<p>end</p>
<p>copy running-config startup-config</p>
<p>! oppure: write memory (abbreviazione identica)</p></td>
</tr>
</tbody>
</table>

## 1.5 Show ip interface brief: interpretazione degli stati

Il comando 'show ip interface brief' è lo strumento diagnostico più usato per un rapido controllo dello stato di tutte le interfacce. Mostra due colonne di stato che vanno interpretate insieme.

|  |  |  |  |
|----|----|----|----|
| **Status (L1/L2)** | **Protocol (L3)** | **Causa possible** | **Azione** |
| administratively down | down | Comando 'shutdown' applicato | Applicare 'no shutdown' |
| down | down | Nessun cavo collegato, cavo difettoso, problema fisico Layer 1 | Verificare cavo, connettori, NIC dell'altro lato |
| up | down | Problema Layer 2: mismatch duplex/speed, incapsulamento errato (PPP vs HDLC), keepalive mancanti | Verificare configurazione L2, encapsulation, duplex |
| up | up | Interfaccia operativa a tutti i livelli | Nessuna azione necessaria |

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><p>R1# show ip interface brief</p>
<p>Interface IP-Address OK? Method Status Protocol</p>
<p>GigabitEthernet0/0 192.168.0.1 YES NVRAM up up</p>
<p>GigabitEthernet0/1 203.0.113.1 YES NVRAM up up</p>
<p>GigabitEthernet0/2 unassigned YES unset admin down down</p>
<p>Loopback0 10.10.10.1 YES NVRAM up up</p>
<p>Serial0/0/0 10.0.0.1 YES NVRAM up up</p></td>
</tr>
</tbody>
</table>

## 1.6 Interfaccia Loopback

La loopback è un'interfaccia logica (virtuale) che non corrisponde ad alcuna porta fisica. E sempre attiva finchè il router e acceso e non dipende dallo stato di nessuna interfaccia fisica.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><p>interface Loopback0</p>
<p>ip address 10.10.10.1 255.255.255.255</p>
<p>description OSPF-Router-ID</p>
<p>! /32 perché e un singolo host - nessun indirizzo di rete o broadcast da riservare</p></td>
</tr>
</tbody>
</table>

Usi principali delle interfacce loopback:

- Router ID per OSPF e BGP: i protocolli di routing usano il Router ID per identificare il router. Configurare il RID come IP della loopback garantisce che il RID non cambi mai a causa di un link down.

- Management: assegnare un IP di gestione stabile (raggiungibile via qualsiasi percorso attivo) per SSH, SNMP, syslog.

- Testing e diagnostica: target stabile per ping e traceroute indipendente dallo stato dei link fisici.

- Tunneling: endpoint stabile per tunnel GRE, IPsec o MPLS.

- BGP: i neighbor BGP iBGP usano tipicamente gli indirizzi loopback come endpoint per la sessione TCP.

## 1.7 Salvataggio della configurazione e memoria

Cisco IOS distingue due memorie distinte per la configurazione:

- running-config: la configurazione attualmente attiva in RAM. Ogni modifica entra immediatamente in effetto ma viene persa al riavvio se non salvata.

- startup-config: la configurazione salvata in NVRAM, caricata all'avvio. Rimane invariata finchè non viene esplicitamente sovrascritta.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><p>! Salvare running -&gt; startup (due forme equivalenti)</p>
<p>copy running-config startup-config</p>
<p>write memory</p>
<p>! Visualizzare la running config corrente</p>
<p>show running-config</p>
<p>! Confrontare running e startup</p>
<p>show archive config differences system:running-config system:startup-config</p>
<p>! Cancellare la startup config (reset al factory default al prossimo boot)</p>
<p>write erase</p>
<p>! oppure:</p>
<p>erase startup-config</p>
<p>! Backup su TFTP server</p>
<p>copy running-config tftp:</p>
<p>! (il sistema chiede indirizzo TFTP e nome file)</p></td>
</tr>
</tbody>
</table>

## 1.8 Moduli espandibili

I router enterprise (es. Cisco ISR 4000, ASR 1000) supportano moduli hardware aggiuntivi inseribili in slot dedicati. A differenza degli switch enterprise modulari, molti router ISR richiedono lo spegnimento del dispositivo per l'installazione di alcuni moduli (NIM - Network Interface Modules, SFP ecc.).

Interfacce comuni nei router:

- GigabitEthernet: porte LAN/WAN su rame RJ45 o fibra SFP.

- Serial (seriale sincrono): usato per WAN legacy (T1/E1, Frame Relay, HDLC). In declino ma presente in molti ambienti.

- FastEthernet: presente su router più vecchi (100 Mbps).

- NIM-2T: modulo con 2 porte seriali per ISR 4000.

- PVDM (Packet Voice DSP Module): per funzionalita VoIP integrate.

- SSD/Flash: storage per immagini IOS e file di configurazione.

# 2. IPv4 — Internet Protocol versione 4

IPv4 è definito nell'RFC 791 (1981) e rappresenta il protocollo di rete dominante di Internet. Fornisce un servizio di consegna dei pacchetti di tipo best-effort: non garantisce consegna, ordine, assenza di duplicati né assenza di corruzione dei dati. Queste garanzie sono delegate ai livelli superiori (TCP a livello trasporto, o alle applicazioni stesse).

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><strong>Formato dell'header IPv4 (RFC 791) — 20 byte minimi</strong></td>
</tr>
<tr>
<td><p>L'header IPv4 ha una dimensione minima di 20 byte (IHL=5, dove IHL e in unita di 4 byte). Può estendersi fino a 60 byte in presenza di opzioni (IHL=15).</p>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<tbody>
<tr>
<td><p>0 1 2 3</p>
<p>0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>|Ver(4)| IHL | DSCP |ECN| Total Length (16) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Identification (16) |Flags(3)| Fragment Offset(13)|</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| TTL (8) | Protocol (8) | Header Checksum (16) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Source Address (32) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Destination Address (32) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Options (0-40 byte) | Padding |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Data / Payload |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p></td>
</tr>
</tbody>
</table>
<p>Descrizione dei campi:</p>
<ul>
<li><p>Version (4 bit): versione del protocollo. Per IPv4 = 4; per IPv6 = 6.</p></li>
<li><p>IHL - Internet Header Length (4 bit): lunghezza dell'header in unita di 32 bit (4 byte). Valore minimo 5 (=20 byte), massimo 15 (=60 byte con opzioni).</p></li>
<li><p>DSCP - Differentiated Services Code Point (6 bit): usato per QoS (qualita del servizio). Sostituisce il vecchio campo ToS (Type of Service). Indica la classe di priorita del pacchetto (es. EF per Expedited Forwarding/VoIP, AF per Assured Forwarding/dati business, BE per Best Effort).</p></li>
<li><p>ECN - Explicit Congestion Notification (2 bit): segnalazione di congestione end-to-end senza scartare pacchetti (RFC 3168). Richiede supporto da entrambi gli endpoint.</p></li>
<li><p>Total Length (16 bit): lunghezza totale del pacchetto (header + payload) in byte. Massimo 65535 byte. In pratica limitato dall'MTU del link (tipicamente 1500 byte per Ethernet).</p></li>
<li><p>Identification (16 bit): identificatore univoco del datagramma originale. Tutti i frammenti dello stesso datagramma portano lo stesso Identification.</p></li>
<li><p>Flags (3 bit): bit 0 riservato (=0); bit 1 DF (Don't Fragment); bit 2 MF (More Fragments).</p></li>
<li><p>Fragment Offset (13 bit): posizione di questo frammento nel datagramma originale, in unita di 8 byte. Il primo frammento ha offset 0.</p></li>
<li><p>TTL - Time To Live (8 bit): decrementato di 1 da ogni router. Quando raggiunge 0, il pacchetto viene scartato e il router invia un ICMP Time Exceeded al mittente. Valori default comuni: Windows 128, Linux 64, Cisco IOS 255.</p></li>
<li><p>Protocol (8 bit): identifica il protocollo del payload. 6 = TCP, 17 = UDP, 1 = ICMP, 89 = OSPF, 47 = GRE, 50 = ESP (IPsec), 41 = IPv6-in-IPv4.</p></li>
<li><p>Header Checksum (16 bit): checksum dell'header IP (non del payload). Ricalcolato ad ogni hop perché il TTL cambia. Rilevazione errori — non correzione.</p></li>
<li><p>Source / Destination Address (32 bit ciascuno): indirizzi IP sorgente e destinazione. Rimangono invariati per tutto il percorso end-to-end (eccetto in caso di NAT/PAT).</p></li>
<li><p>Options (variabile, 0-40 byte): campi opzionali (Record Route, Timestamp, Loose/Strict Source Routing, ecc.). Raramente usati in produzione; alcuni IDS li considerano sospetti.</p></li>
</ul></td>
</tr>
</tbody>
</table>

## 2.1 TTL e diagnostica

Il campo TTL ha una doppia funzione: prevenire i loop di routing infiniti e fornire un meccanismo di diagnostica. Ogni router decrementa il TTL di 1. Se il TTL raggiunge 0, il pacchetto viene scartato e il router invia un messaggio ICMP Time Exceeded (tipo 11, codice 0) all'indirizzo IP sorgente.

Lo strumento 'traceroute' sfrutta deliberatamente questo meccanismo: invia pacchetti con TTL = 1, 2, 3, ... e raccoglie i messaggi ICMP Time Exceeded dai router intermedi per mappare il percorso end-to-end.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><p>! Linux/macOS: traceroute usa UDP per default</p>
<p>traceroute 8.8.8.8</p>
<p>! Windows: tracert usa ICMP Echo Request</p>
<p>tracert 8.8.8.8</p>
<p>! Cisco IOS: usa UDP per default (come Linux)</p>
<p>traceroute 8.8.8.8</p>
<p>! Extended traceroute (interattivo, più opzioni):</p>
<p>traceroute</p></td>
</tr>
</tbody>
</table>

I valori di TTL iniziale sono caratteristici del sistema operativo mittente e possono essere usati per OS fingerprinting passivo:

|                          |                          |
|--------------------------|--------------------------|
| **Sistema operativo**    | **TTL iniziale default** |
| Linux (la maggior parte) | 64                       |
| Windows (XP, 7, 10, 11)  | 128                      |
| Cisco IOS                | 255                      |
| macOS                    | 64                       |
| Solaris                  | 255                      |

## 2.2 Frammentazione IP e MTU

La frammentazione avviene quando un pacchetto IP ha dimensioni superiori alla MTU (Maximum Transmission Unit) del link su cui deve essere trasmesso. La MTU di Ethernet e 1500 byte (payload IP); la MTU di una connessione PPPoE è tipicamente 1492 byte.

Processo di frammentazione:

1.  Il router determina che il pacchetto (es. 3000 byte) supera la MTU del link di uscita (es. 1500 byte).

2.  Se il bit DF (Don't Fragment) è impostato NOT: il router frammenta il pacchetto. I frammenti hanno tutti lo stesso Identification; il Fragment Offset indica la posizione di ogni frammento (in unita di 8 byte); tutti i frammenti tranne l'ultimo hanno MF = 1; l'ultimo ha MF = 0.

3.  Se il bit DF è impostato: il router scarta il pacchetto e invia un ICMP Destination Unreachable (tipo 3, codice 4: Fragmentation Needed and DF Set) al mittente. Il messaggio ICMP include la MTU del link che ha causato il problema.

4.  La riassemblatura avviene solo sul destinatario finale, non sui router intermedi.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><p>! Test ping con DF bit e dimensione payload variabile (Windows)</p>
<p>ping -f -l 1472 192.168.1.1</p>
<p>! -f = Don't Fragment; -l = payload in byte</p>
<p>! 1472 + 8 (UDP) + 20 (IP) = 1500 = MTU Ethernet esatta -&gt; deve passare</p>
<p>! 1473 -&gt; 1501 byte -&gt; superera la MTU -&gt; ICMP Fragmentation Needed</p>
<p>! Linux equivalente:</p>
<p>ping -M do -s 1472 192.168.1.1</p>
<p>! -M do = DF bit impostato; -s = payload in byte</p></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><strong>Approfondimento — Path MTU Discovery (PMTUD)</strong></td>
</tr>
<tr>
<td><p>Path MTU Discovery (RFC 1191) è il meccanismo con cui un host TCP determina automaticamente la MTU massima lungo il percorso verso il destinatario, evitando la frammentazione.</p>
<ol start="5" type="1">
<li><p>L'host imposta DF=1 su tutti i pacchetti TCP.</p></li>
<li><p>Se un router lungo il percorso non può inoltrare il pacchetto perché supera la propria MTU, invia ICMP Fragmentation Needed (tipo 3, codice 4) con la MTU del link nel payload del messaggio.</p></li>
<li><p>L'host riduce la propria MSS (Maximum Segment Size TCP) al valore indicato dall'ICMP e ritrasmette.</p></li>
<li><p>Il processo si ripete finchè i pacchetti passano senza frammentazione.</p></li>
</ol>
<p>Problema: il PMTUD fallisce se i messaggi ICMP tipo 3/codice 4 vengono bloccati da firewall che applicano politiche di 'deny all ICMP'. Questo causa il fenomeno noto come 'black hole routing': la connessione TCP si stabilisce (SYN/SYN-ACK/ACK passano essendo piccoli), ma non si trasferiscono dati (i segmenti di dati più grandi vengono silenziosamente scartati). Soluzione: MSS clamping sull'interfaccia WAN (ip tcp adjust-mss 1452 su Cisco IOS).</p></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><strong>Sicurezza — Attacchi tramite frammentazione IP</strong></td>
</tr>
<tr>
<td><p>La frammentazione IP e stata sfruttata in diverse categorie di attacchi:</p>
<p><strong>Teardrop Attack (storico):</strong> frammenti con offset sovrapposti o malformati causavano crash in alcune implementazioni dello stack TCP/IP (Windows 3.1, NT 3.5, Linux pre-2.0). Il kernel tentava di riassemblare frammenti con offset incoerenti, causando buffer overflow.</p>
<p><strong>Ping of Death (storico):</strong> pacchetti ICMP con dimensione totale dopo la riassemblatura superiore a 65535 byte (il massimo dell'header IP Total Length). Il buffer di riassemblatura andava in overflow.</p>
<p><strong>Tiny Fragment Attack:</strong> il primo frammento e cosi piccolo da contenere solo il primo byte dell'header TCP (flags, seq number). Alcuni firewall 'stateless' ispezionavano solo il primo frammento per decidere se accettare o bloccare il flusso; con un primo frammento minuscolo, le flags TCP non erano visibili e il traffico passava. Contromisura: i firewall moderni bloccano frammenti con offset=0 e dimensione &lt; 8 byte.</p>
<p><strong>Fragment Reassembly Buffer Exhaustion:</strong> un attaccante invia un grande numero di frammenti parziali (senza mai inviare l'ultimo frammento). Il router o host destinatario mantiene buffer di riassemblatura per ogni flusso frammentato incompleto. Se il buffer si esaurisce, si verifica un denial of service. Contromisura: timeout di riassemblatura (tipicamente 15-60 secondi), dimensione massima del buffer di riassemblatura.</p>
<p><strong>IDS/Firewall Evasion via fragmentation:</strong> payload malevolo diviso tra più frammenti per evadere la signature detection degli IDS che non riassemblano i frammenti prima dell'ispezione. Soluzione: deep packet inspection con riassemblatura virtuale.</p>
<p>Attacchi storici come Teardrop e Ping of Death sono stati corretti nelle implementazioni moderne. La frammentazione-based evasion rimane però un vettore rilevante nella sicurezza di rete e nei test di penetrazione.</p></td>
</tr>
</tbody>
</table>

## 2.3 Indirizzi IPv4 speciali e riservati

Oltre agli indirizzi privati (RFC 1918), l'IANA riserva diversi blocchi per usi specifici. Conoscere questi blocchi e fondamentale per la configurazione corretta di ACL, firewall e route filter.

|  |  |  |  |
|----|----|----|----|
| **Blocco** | **Descrizione** | **RFC** | **Note operative** |
| 10.0.0.0/8 | Indirizzi privati (Class A) | RFC 1918 | Non instradabili su Internet; usabili liberamente in reti private. |
| 100.64.0.0/10 | Shared Address Space (CGNAT) | RFC 6598 | Usato dagli ISP per Carrier Grade NAT. Non deve apparire nelle reti private ne su Internet. |
| 127.0.0.0/8 | Loopback | RFC 1122 | 127.0.0.1 e il loopback standard; tutta la /8 e loopback sul singolo host. Non esce mai sull'interfaccia fisica. |
| 169.254.0.0/16 | Link-Local (APIPA) | RFC 3927 | Assegnato automaticamente se DHCP fallisce (Windows) o per IPv4 link-local. Non instradabile. |
| 172.16.0.0/12 | Indirizzi privati (Class B) | RFC 1918 | Comprende 172.16.0.0 - 172.31.255.255. |
| 192.0.0.0/24 | IETF Protocol Assignments | RFC 6890 | Riservato per documentazione di protocolli IETF. |
| 192.0.2.0/24 | TEST-NET-1 (documentazione) | RFC 5737 | Usato negli esempi di RFC e documentazione tecnica. Non instradabile. |
| 192.88.99.0/24 | 6to4 Relay Anycast (deprecato) | RFC 7526 | Storicamente usato per la transizione IPv6. Ora deprecato. |
| 192.168.0.0/16 | Indirizzi privati (Class C) | RFC 1918 | Usatissimo nelle reti domestiche e piccole LAN. |
| 198.18.0.0/15 | Benchmark Testing | RFC 2544 | Riservato per test di performance di apparati di rete. |
| 198.51.100.0/24 | TEST-NET-2 | RFC 5737 | Documentazione tecnica. |
| 203.0.113.0/24 | TEST-NET-3 | RFC 5737 | Documentazione tecnica. |
| 224.0.0.0/4 | Multicast | RFC 1112 | Classe D. 224.0.0.0/24 = link-local multicast (OSPF: 224.0.0.5/6, RIP: 224.0.0.9). |
| 240.0.0.0/4 | Riservato (futuro) | RFC 1112 | Classe E. Non usato nelle reti correnti. |
| 255.255.255.255/32 | Broadcast limitato | RFC 919 | Broadcast su segmento locale. Non instradabile. |

## 2.4 Subnetting e CIDR

Il subnetting e il processo di suddivisione di un blocco di indirizzi IP in sottoreti più piccole, aumentando i bit della parte rete a scapito dei bit della parte host. CIDR (Classless Inter-Domain Routing, RFC 4632) ha sostituito il sistema classful consentendo prefissi di qualsiasi lunghezza.

Formule fondamentali per una subnet /\<n\>:

- Numero di indirizzi totali: 2^(32-n)

- Numero di host utilizzabili: 2^(32-n) - 2 (si sottraggono indirizzo di rete e broadcast)

- Subnet mask: i primi n bit a 1, i restanti 32-n bit a 0

|  |  |  |  |  |
|----|----|----|----|----|
| **Prefisso CIDR** | **Subnet mask** | **Indirizzi totali** | **Host utilizzabili** | **Uso tipico** |
| /30 | 255.255.255.252 | 4 | 2 | Link point-to-point tra router |
| /29 | 255.255.255.248 | 8 | 6 | Subnet very small |
| /28 | 255.255.255.240 | 16 | 14 | Piccoli segmenti |
| /27 | 255.255.255.224 | 32 | 30 | Piccole subnet |
| /26 | 255.255.255.192 | 64 | 62 | Subnet media |
| /25 | 255.255.255.128 | 128 | 126 | Subnet media |
| /24 | 255.255.255.0 | 256 | 254 | Subnet standard LAN |
| /23 | 255.255.254.0 | 512 | 510 | Subnet media |
| /22 | 255.255.252.0 | 1024 | 1022 | Subnet grande |
| /16 | 255.255.0.0 | 65536 | 65534 | Blocco classe B |
| /8 | 255.0.0.0 | 16777216 | 16777214 | Blocco classe A |

**Subnet /31 e /32:** *RFC 3021 autorizza l'uso di subnet /31 per link point-to-point: contengono esattamente 2 indirizzi senza indirizzo di rete nè broadcast, entrambi assegnabili agli endpoint. I moderni router Cisco supportano /31. La subnet /32 indica un singolo host (usata per loopback, host route, BGP neighbor).*

# 3. Routing: Control Plane e Data Plane

L'architettura di un router moderno separa nettamente due piani funzionali: il control plane e il data plane. Questa separazione è fondamentale per comprendere sia le prestazioni che la sicurezza dei dispositivi di rete.

## 3.1 Control Plane

Il control plane è il cervello del router: gestisce tutta la logica di decisione e i protocolli di controllo. Opera sulla CPU dell'apparato.

- Esegue i protocolli di routing (OSPF, BGP, EIGRP, RIP): scambia informazioni di raggiungibilita con i vicini.

- Costruisce e mantiene la RIB (Routing Information Base): il database completo di tutte le rotte apprese.

- Popola la FIB (Forwarding Information Base): la struttura ottimizzata usata dal data plane per il forwarding rapido.

- Gestisce i protocolli di controllo: ARP, ICMP, STP (negli switch), LACP, LLDP/CDP.

- Risponde ai messaggi di management: SSH, SNMP, Netconf/YANG.

**Protezione del Control Plane:** *Il control plane è vulnerabile ad attacchi DoS che sovraccaricano la CPU del router con traffico di controllo. La funzione Control Plane Policing (CoPP) su Cisco IOS limita la frequenza del traffico inviato alla CPU, proteggendo il router da attacchi di tipo CPU exhaustion. Esempio: limitare le richieste ARP o i pacchetti OSPF a un rate massimo per evitare che un attaccante saturi la CPU.*

## 3.2 Data Plane

Il data plane (o forwarding plane) e responsabile dell'inoltro effettivo dei pacchetti ad alta velocita. Nei router e switch moderni, il data plane e implementato in hardware dedicato (ASIC - Application-Specific Integrated Circuit, o NPU - Network Processing Unit) che opera in modo indipendente dalla CPU.

- Esegue il Longest Prefix Match (LPM) sulla FIB per determinare il next-hop di ogni pacchetto.

- Decrementa il TTL e ricalcola il checksum dell'header IP.

- Riencapsula il pacchetto nel frame Layer 2 appropriato per il link di uscita.

- Applica le policy di QoS (classificazione, marking, queuing, scheduling).

- Applica le ACL (Access Control List) per il filtraggio del traffico.

- Esegue NAT/PAT se configurato.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><strong>Approfondimento — CEF: Cisco Express Forwarding</strong></td>
</tr>
<tr>
<td><p>CEF (Cisco Express Forwarding) e il meccanismo di forwarding hardware-accelerato usato da Cisco IOS. Separa la FIB (Forwarding Information Base) dall'adjacency table per massimizzare la velocita di forwarding.</p>
<p><strong>FIB (Forwarding Information Base):</strong> derivata dalla routing table (RIB), ottimizzata per la ricerca del next-hop tramite LPM. Aggiornata dal control plane ogni volta che la routing table cambia. Il data plane consulta la FIB per ogni pacchetto.</p>
<p><strong>Adjacency Table:</strong> contiene le informazioni Layer 2 necessarie per costruire il frame di uscita (MAC address del next-hop, interfaccia di uscita). Viene popolata da ARP (per IPv4) e da NDP (per IPv6). Quando il data plane trova il next-hop nella FIB, usa l'adjacency table per costruire immediatamente il nuovo frame senza invocare ARP.</p>
<p>Prima di CEF, Cisco IOS usava due modalità più lente:</p>
<ul>
<li><p>Process switching: ogni pacchetto viene consegnato alla CPU per l'elaborazione completa. Molto lento, usato solo per pacchetti che richiedono attenzione speciale (es. pacchetti con opzioni IP, traffico verso la CPU stessa).</p></li>
<li><p>Fast switching (route caching): il primo pacchetto di un flusso viene processato dalla CPU, che crea una entry nella cache. I pacchetti successivi dello stesso flusso usano la cache. Più veloce del process switching ma incoerente.</p></li>
</ul>
<p>Con CEF, la FIB e l'adjacency table sono pre-calcolate e disponibili istantaneamente: il forwarding avviene interamente in hardware senza interruzione della CPU per i pacchetti ordinari.</p></td>
</tr>
</tbody>
</table>

## 3.3 Routing stateless e stateful

Il routing IP tradizionale è stateless per definizione: il router prende una decisione di forwarding indipendente per ogni singolo pacchetto, basandosi esclusivamente sull'IP di destinazione e sulla routing table. Non esiste memoria delle connessioni precedenti.

Implicazioni del routing stateless:

- Due pacchetti della stessa sessione TCP possono seguire percorsi diversi se la routing table cambia tra l'uno e l'altro (ECMP - Equal Cost Multi-Path).

- Un pacchetto non porta con sé informazioni sulla sessione a cui appartiene (quella e nel payload TCP/UDP).

- I router intermedi non devono mantenere tabelle di sessione, il che permette una scalabilita enorme (Internet).

Eccezioni dove il router mantiene stato:

- NAT/PAT: la traduzione degli indirizzi richiede una tabella di sessioni IP+porta per correlare i pacchetti in uscita e in ingresso.

- Stateful firewall: un firewall integrato nel router (es. Cisco Zone-Based Firewall) mantiene la session table per permettere il traffico di risposta correlato.

- QoS con queuing: il router mantiene code per flusso per implementare politiche di scheduling.

- NBAR (Network-Based Application Recognition): il router ispeziona i payload per classificare le applicazioni, richiedendo la correlazione di più pacchetti.

## 3.4 Routing statico vs dinamico: configurazione ed esempi

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><p>! === ROUTING STATICO ===</p>
<p>! Rotta verso la rete 10.0.0.0/8 via next-hop 192.168.1.254</p>
<p>ip route 10.0.0.0 255.0.0.0 192.168.1.254</p>
<p>! Default route (usata per tutto il traffico senza rotta specifica)</p>
<p>ip route 0.0.0.0 0.0.0.0 203.0.113.1 ! gateway ISP</p>
<p>! Rotta floating: backup con AD 200 (usata solo se la rotta OSPF con AD=110 cade)</p>
<p>ip route 192.168.2.0 255.255.255.0 10.0.0.2 200</p>
<p>! === ROUTING OSPF (dinamico) ===</p>
<p>router ospf 1</p>
<p>router-id 10.10.10.1</p>
<p>network 192.168.0.0 0.0.0.255 area 0</p>
<p>network 10.10.10.1 0.0.0.0 area 0</p>
<p>default-information originate always ! redistribuisce la default route in OSPF</p>
<p>passive-interface GigabitEthernet0/0 ! non invia Hello su questa interfaccia</p></td>
</tr>
</tbody>
</table>

## 3.5 Connessione tra due router: subnet /30

I link point-to-point tra router (collegamenti WAN, link inter-router) usano tipicamente subnet /30 (4 indirizzi: rete, R1, R2, broadcast) o /31 (2 indirizzi, RFC 3021).

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><p>! === Router R1 ===</p>
<p>interface GigabitEthernet 0/1</p>
<p>description LINK-verso-R2</p>
<p>ip address 192.168.100.1 255.255.255.252</p>
<p>no shutdown</p>
<p>! === Router R2 ===</p>
<p>interface GigabitEthernet 0/0</p>
<p>description LINK-verso-R1</p>
<p>ip address 192.168.100.2 255.255.255.252</p>
<p>no shutdown</p>
<p>! === Verifica ===</p>
<p>R1# show ip route</p>
<p>C 192.168.100.0/30 is directly connected, GigabitEthernet0/1</p>
<p>L 192.168.100.1/32 is directly connected, GigabitEthernet0/1</p>
<p>! 'C' = Connected (la rete); 'L' = Local (l'indirizzo specifico del router)</p>
<p>R1# ping 192.168.100.2 ! verifica connettivita con R2</p></td>
</tr>
</tbody>
</table>

**Codici C e L nella routing table:** *Il codice 'C' indica una rete direttamente connessa (la subnet intera, es. 192.168.100.0/30). Il codice 'L' indica la host route specifica (/32) dell'indirizzo assegnato all'interfaccia locale del router (es. 192.168.100.1/32). Il codice 'L' e stato introdotto in IOS 15.x e non era presente nelle versioni precedenti. Serve al router per ricevere correttamente i pacchetti destinati a sé stesso senza consultare ARP.*

# 4. Quality of Service (QoS)

La Quality of Service è l'insieme di tecniche e meccanismi che permettono a una rete di gestire il traffico in modo differenziato, garantendo a certe classi di traffico parametri di performance specifici (latenza, jitter, bandwidth, packet loss).

Senza QoS, un router tratta tutti i pacchetti allo stesso modo (best-effort FIFO - First In First Out): in caso di congestione, i pacchetti vengono scartati indiscriminatamente, penalizzando anche le applicazioni sensibili come VoIP e videoconferenza.

## 4.1 I quattro parametri fondamentali di QoS

|  |  |  |  |
|----|----|----|----|
| **Parametro** | **Definizione** | **Impatto applicativo** | **Target tipico (VoIP)** |
| Delay (latenza) | Tempo totale di trasporto di un pacchetto da sorgente a destinazione. Include propagation, transmission, queuing e processing delay. | Ritardo percepito nella risposta; sopra 150ms il VoIP diventa scomodo; sopra 400ms inaccettabile. | \< 150 ms one-way (ITU G.114) |
| Jitter | Variazione del delay tra pacchetti consecutivi dello stesso flusso. Un pacchetto che arriva più tardi del previsto crea 'buchi' nell'audio. | L'applicazione usa un playout buffer per compensare il jitter: buffer troppo piccolo = dropout; troppo grande = latenza aggiuntiva. | \< 30 ms |
| Throughput (goodput) | Quantità di dati utili (escluso overhead) trasferiti per unita di tempo. Può essere limitato da congestione, errori, TCP slow start. | Determina la velocita di download/upload percepita dall'utente. | Banda garantita (es. 87 kbps per G.711) |
| Packet Loss | Percentuale di pacchetti non consegnati. Causata da congestione (code piene), errori fisici o TTL=0. | TCP retransmette i pacchetti persi (throughput ridotto); UDP (VoIP, video) non retransmette -\> dropout audio/video. | \< 1% (ideale \< 0.1%) |

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><strong>Approfondimento — DiffServ e DSCP (RFC 2474, RFC 4594)</strong></td>
</tr>
<tr>
<td><p>Il framework DiffServ (Differentiated Services) definisce come classificare e marcare i pacchetti IP per applicare trattamenti di forwarding differenziati. Il campo DSCP (6 bit nell'header IP, ex-campo ToS) identifica la classe di servizio.</p>
<p>Classi DSCP principali e relative PHB (Per-Hop Behavior):</p>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<tbody>
<tr>
<td><p>DSCP Valore dec. PHB Descrizione</p>
<p>------ ----------- ------ -----------------------------------------</p>
<p>EF 46 (101110) EF Expedited Forwarding: massima priorita,</p>
<p>banda garantita, bassa latenza. -&gt; VoIP RTP</p>
<p>AF41 34 (100010) AF Assured Forwarding class 4, drop pref 1.</p>
<p>AF42 36 AF -&gt; Video conferenza interattiva</p>
<p>AF31 26 AF -&gt; Segnalazione (SIP, H.323)</p>
<p>AF21 18 AF -&gt; Applicazioni business critiche</p>
<p>AF11 10 AF -&gt; Dati bulk prioritari</p>
<p>CS3 24 CS -&gt; Segnalazione di rete</p>
<p>CS2 16 CS -&gt; OAM (Operations, Admin, Maintenance)</p>
<p>CS1 8 CS -&gt; Scavenger (Bittorrent, P2P - minima prio)</p>
<p>BE/CS0 0 (000000) BE Best Effort: traffico standard senza garanzie</p></td>
</tr>
</tbody>
</table>
<p>I router classically implementano QoS tramite il modello MQC (Modular QoS CLI) in tre fasi:</p>
<ol start="9" type="1">
<li><p>Class-map: classificazione del traffico (per DSCP, ACL, protocollo NBAR, porta).</p></li>
<li><p>Policy-map: definizione delle azioni per ogni classe (priority, bandwidth guarantee, queuing algorithm).</p></li>
<li><p>Service-policy: applicazione della policy a un'interfaccia (in ingresso o in uscita).</p></li>
</ol>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<tbody>
<tr>
<td><p>! Esempio configurazione QoS MQC per VoIP</p>
<p>class-map match-any VOIP-CLASS</p>
<p>match dscp ef</p>
<p>match dscp cs3</p>
<p>policy-map QOS-WAN</p>
<p>class VOIP-CLASS</p>
<p>priority percent 30</p>
<p>class class-default</p>
<p>fair-queue</p>
<p>interface GigabitEthernet 0/1</p>
<p>service-policy output QOS-WAN</p></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

# 5. ICMP — Internet Control Message Protocol

ICMP (RFC 792) è un protocollo di supporto a IP che trasporta messaggi di controllo e segnalazione degli errori. Non è un protocollo di trasporto dati ma è fondamentale per la diagnostica e il funzionamento della rete. Viene incapsulato direttamente in IP con Protocol Number = 1.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><strong>Formato e tipi di messaggi ICMP (RFC 792)</strong></td>
</tr>
<tr>
<td><p>Ogni messaggio ICMP ha un header comune di 8 byte seguito da dati specifici del tipo:</p>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<tbody>
<tr>
<td><p>0 1 2 3</p>
<p>0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Type (8) | Code (8) | Checksum (16) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p>
<p>| Dati specifici del tipo (variabile) |</p>
<p>+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+</p></td>
</tr>
</tbody>
</table>
<p>Tipi ICMP principali e loro utilizzo:</p>
<table style="width:96%;">
<colgroup>
<col style="width: 95%" />
</colgroup>
<tbody>
<tr>
<td><p>Tipo Codice Nome Uso</p>
<p>---- ------ ------------------------ ----------------------------------</p>
<p>0 0 Echo Reply Risposta al ping (da destinatario)</p>
<p>3 0 Net Unreachable La rete non è raggiungibile</p>
<p>3 1 Host Unreachable L'host non è raggiungibile</p>
<p>3 2 Protocol Unreachable Il protocollo non è supportato</p>
<p>3 3 Port Unreachable La porta UDP non è in ascolto</p>
<p>3 4 Fragmentation Needed PMTUD: pacchetto troppo grande con DF=1</p>
<p>3 13 Comm. Admin. Prohibited Un firewall ha bloccato il pacchetto</p>
<p>4 0 Source Quench (deprecato) Richiesta di rallentamento al mittente</p>
<p>5 0 Redirect (Host) Un router suggerisce un percorso migliore</p>
<p>8 0 Echo Request Il ping (da mittente)</p>
<p>11 0 TTL Exceeded in Transit TTL=0: usato da traceroute</p>
<p>11 1 TTL Exceeded in Reassembly Timeout riassemblatura frammenti</p>
<p>12 0 Parameter Problem Errore nell'header IP</p>
<p>13 0 Timestamp Request Sincronizzazione oraria (raramente usato)</p>
<p>17 0 Address Mask Request Richiesta subnet mask (obsoleto)</p></td>
</tr>
</tbody>
</table>
<p>Nei messaggi di errore (tipo 3, 11, 12), il payload ICMP include l'header IP originale + i primi 8 byte del payload (header TCP/UDP) del pacchetto che ha causato l'errore. Questo permette al mittente originale di identificare quale connessione ha generato l'errore.</p>
<p>Sicurezza e ICMP: molti firewall bloccano tutto l'ICMP indiscriminatamente. Questa è una cattiva pratica perché blocca PMTUD (tipo 3, codice 4) e traceroute (tipo 11). Le best practice (RFC 4890) raccomandano di permettere selettivamente: Echo Request/Reply (8/0), Destination Unreachable (3), Time Exceeded (11), Parameter Problem (12), e bloccare il resto.</p></td>
</tr>
</tbody>
</table>

# 6. Overview dei protocolli di rete rilevanti

Il livello Network (L3) è il livello Transport (L4) ospitano numerosi protocolli con funzioni distinte. La tabella seguente fornisce una panoramica dei protocolli citati in questa lezione e nelle precedenti, organizzati per livello OSI.

|  |  |  |  |  |
|----|----|----|----|----|
| **Livello OSI** | **Protocollo** | **RFC principale** | **Funzione** | **Trasporto** |
| L3 - Network | IPv4 | RFC 791 | Indirizzamento e routing pacchetti. Best-effort. | — |
| L3 - Network | ICMP | RFC 792 | Messaggi di controllo e segnalazione errori IP. | IP proto 1 |
| L3 - Network | IGMP | RFC 3376 | Gestione dei gruppi multicast IPv4 tra host e router. | IP proto 2 |
| L3 - Network | OSPF | RFC 2328 | Protocollo di routing Link State. Calcola SPF. | IP proto 89 |
| L3 - Network | EIGRP | RFC 7868 | Protocollo di routing Cisco (ibrido DV+LS). DUAL algorithm. | IP proto 88 |
| L3 - Network | BGP | RFC 4271 | Protocollo di routing tra AS su Internet. Path vector. | TCP 179 |
| L4 - Transport | TCP | RFC 793 | Trasporto affidabile, connection-oriented, controllo flusso. | IP proto 6 |
| L4 - Transport | UDP | RFC 768 | Trasporto non affidabile, connectionless, bassa latenza. | IP proto 17 |
| L4 - Transport | SCTP | RFC 4960 | Trasporto affidabile multi-stream, multi-homing. Usato in SS7/telecomunicazioni. | IP proto 132 |

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><strong>Approfondimento — EIGRP: Enhanced Interior Gateway Routing Protocol</strong></td>
</tr>
<tr>
<td><p>EIGRP (originariamente proprietario Cisco, ora RFC 7868) è un protocollo di routing che combina caratteristiche dei Distance Vector e dei Link State. Usa l'algoritmo DUAL (Diffusing Update Algorithm) che garantisce la convergenza senza loop.</p>
<p>Caratteristiche principali:</p>
<ul>
<li><p>Metrica composita: basata su bandwidth e delay del percorso (opzionalmente load e reliability). Metrica = 256 * (K1/bandwidth + K3*delay). Di default K1=K3=1, K2=K4=K5=0.</p></li>
<li><p>Aggiornamenti parziali: EIGRP invia aggiornamenti solo quando c'e un cambio di topologia, non periodicamente (a differenza di RIP). Usa multicast 224.0.0.10.</p></li>
<li><p>DUAL: mantiene una Successor (percorso migliore) e una Feasible Successor (percorso di backup pre-calcolato, garantito loop-free). In caso di guasto del percorso primario, il failover sul Feasible Successor e istantaneo (sub-second).</p></li>
<li><p>Unequal cost load balancing: EIGRP supporta il bilanciamento del carico su percorsi con metrica diversa (tramite il comando 'variance'), funzione non disponibile in OSPF.</p></li>
<li><p>Compatibile con IPv4 e IPv6 (EIGRP for IPv6).</p></li>
</ul>
<p>Confronto AD: EIGRP internal = 90, EIGRP external = 170 (redistribuzione da altri protocolli).</p></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><strong>Approfondimento — BGP: Border Gateway Protocol</strong></td>
</tr>
<tr>
<td><p>BGP (RFC 4271) e il protocollo di routing usato su Internet per lo scambio di routing information tra Autonomous System (AS) diversi. E il protocollo che 'tiene insieme' Internet.</p>
<p>Concetti fondamentali:</p>
<ul>
<li><p>Autonomous System (AS): un insieme di reti sotto il controllo di una singola organizzazione con una politica di routing coerente. Identificato da un numero a 16 o 32 bit (ASN). Esempio: AS3549 (Level3/Lumen), AS20940 (Akamai).</p></li>
<li><p>Path Vector: BGP non usa una metrica numerica semplice ma annuncia percorsi (sequenze di AS). Il decisionale si basa su attributi: AS_PATH (lunghezza), NEXT_HOP, LOCAL_PREF, MED, ORIGIN, Communities.</p></li>
<li><p>eBGP vs iBGP: eBGP (external BGP) connette AS diversi; iBGP (internal BGP) connette router dentro lo stesso AS per propagare le rotte BGP internamente.</p></li>
<li><p>Sessioni TCP: BGP usa TCP porta 179. La stabilita della sessione TCP e fondamentale; BGP su loopback con 'ebgp-multihop' o 'update-source' e prassi comune.</p></li>
<li><p>BGP non è un protocollo 'plug and play': richiede configurazione manuale dei neighbor, delle policy di import/export e dei filtri. E il protocollo di routing più complesso e più potente.</p></li>
</ul>
<p>BGP è fondamentale per chi studia cybersecurity: attacchi di BGP hijacking (annuncio di prefissi altrui), BGP route leak (propagazione errata di rotte) e la relativa difesa tramite RPKI (Resource Public Key Infrastructure) sono argomenti avanzati ma rilevanti nella sicurezza delle infrastrutture Internet.</p></td>
</tr>
</tbody>
</table>

# 7. IETF e il processo di standardizzazione RFC

L'IETF (Internet Engineering Task Force) è l'organizzazione aperta che sviluppa e promuove gli standard tecnici di Internet. Non è un ente governativo né un'associazione formale con quote di iscrizione: chiunque può partecipare alle mailing list e ai working group.

Caratteristiche del processo IETF:

- Consenso approssimativo e codice funzionante: la filosofia IETF privilegia l'implementazione pratica sulla burocrazia ('rough consensus and running code', attribuita a Dave Clark, 1992).

- RFC (Request for Comments): tutti gli standard IETF vengono pubblicati come RFC. Il nome e storico (gli RFC originali del 1969 erano effettivamente richieste di commento); oggi le RFC pubblicate dopo il processo di revisione sono standard definitivi.

- Working Group: i lavori sono organizzati in WG tematici (es. secsh per SSH, ospf per OSPF, tls per TLS). Ogni WG ha una charter e produce RFC.

- Categorie di RFC: Proposed Standard -\> Internet Standard (Standards Track); Best Current Practice (BCP); Informational; Experimental; Historic (obsoleta).

|  |  |  |  |
|----|----|----|----|
| **RFC** | **Titolo** | **Anno** | **Rilevanza per questa lezione** |
| RFC 791 | Internet Protocol (IPv4) | 1981 | Specifica fondamentale di IPv4 e dell'header IP |
| RFC 792 | Internet Control Message Protocol (ICMP) | 1981 | Messaggi ICMP, Destination Unreachable, Time Exceeded |
| RFC 793 | Transmission Control Protocol (TCP) | 1981 | Trasporto affidabile, connection-oriented |
| RFC 768 | User Datagram Protocol (UDP) | 1980 | Trasporto connectionless |
| RFC 1122 | Requirements for Internet Hosts — Communication Layers | 1989 | 127.0.0.0/8 loopback, comportamento corretto degli host |
| RFC 1191 | Path MTU Discovery | 1990 | PMTUD per IPv4 |
| RFC 2131 | Dynamic Host Configuration Protocol | 1997 | DHCP (trattato in Lezione 4) |
| RFC 2474 | Definition of the Differentiated Services Field in IPv4 and IPv6 Headers | 1998 | DSCP e DiffServ per QoS |
| RFC 2328 | OSPF Version 2 | 1998 | Protocollo Link State (trattato in Lezione 4) |
| RFC 3021 | Using 31-Bit Prefixes on IPv4 Point-to-Point Links | 2000 | Subnet /31 per link P2P |
| RFC 4271 | A Border Gateway Protocol 4 (BGP-4) | 2006 | BGP, protocollo di routing inter-AS |
| RFC 4594 | Configuration Guidelines for DiffServ Service Classes | 2006 | Best practice DSCP e classi QoS |
| RFC 5737 | IPv4 Address Blocks Reserved for Documentation | 2010 | 192.0.2.0/24, 198.51.100.0/24, 203.0.113.0/24 |
| RFC 6598 | IANA-Reserved IPv4 Prefix for Shared Address Space (CGNAT) | 2012 | 100.64.0.0/10 per Carrier Grade NAT |
| RFC 7868 | Cisco's Enhanced Interior Gateway Routing Protocol (EIGRP) | 2016 | Standardizzazione di EIGRP |
| RFC 4960 | Stream Control Transmission Protocol (SCTP) | 2007 | SCTP per telecomunicazioni e multi-homing |

# Bibliografia

Le opere sono organizzate per argomento e citate in formato APA 7a edizione.

## Reti informatiche — testi fondamentali

Tanenbaum, A. S., & Wetherall, D. J. (2011). *Computer networks* (5th ed.). Prentice Hall.

Kurose, J. F., & Ross, K. W. (2021). *Computer networking: A top-down approach* (8th ed.). Pearson.

Forouzan, B. A. (2013). *Data communications and networking* (5th ed.). McGraw-Hill.

## IPv4 e protocolli di livello rete — RFC

Postel, J. (1981). *Internet Protocol* (RFC 791). IETF. https://www.rfc-editor.org/rfc/rfc791

Postel, J. (1981). *Internet Control Message Protocol* (RFC 792). IETF. https://www.rfc-editor.org/rfc/rfc792

Braden, R. (1989). *Requirements for Internet hosts — Communication layers* (RFC 1122). IETF. https://www.rfc-editor.org/rfc/rfc1122

Mogul, J., & Deering, S. (1990). *Path MTU discovery* (RFC 1191). IETF. https://www.rfc-editor.org/rfc/rfc1191

Hinden, R., & Deering, S. (2010). *IPv4 address blocks reserved for documentation* (RFC 5737). IETF. https://www.rfc-editor.org/rfc/rfc5737

Weil, J., Kuarsingh, V., Donley, C., Liljenstolpe, C., & Azinger, M. (2012). *IANA-reserved IPv4 prefix for shared address space* (RFC 6598). IETF. https://www.rfc-editor.org/rfc/rfc6598

Retana, A., & White, R. (2000). *Using 31-bit prefixes on IPv4 point-to-point links* (RFC 3021). IETF. https://www.rfc-editor.org/rfc/rfc3021

## Frammentazione IP e sicurezza

Ptacek, T. H., & Newsham, T. N. (1998). *Insertion, evasion, and denial of service: Eluding network intrusion detection*. Secure Networks, Inc.

Shankar, U., & Paxson, V. (2003). Active mapping: Resisting NIDS evasion without altering traffic. *Proceedings of the IEEE Symposium on Security and Privacy*, 44-61. https://doi.org/10.1109/SECPRI.2003.1199331

## QoS e DiffServ

Blake, S., Black, D., Carlson, M., Davies, E., Wang, Z., & Weiss, W. (1998). *An architecture for differentiated services* (RFC 2475). IETF. https://www.rfc-editor.org/rfc/rfc2475

Nichols, K., Blake, S., Baker, F., & Black, D. (1998). *Definition of the Differentiated Services field in the IPv4 and IPv6 headers* (RFC 2474). IETF. https://www.rfc-editor.org/rfc/rfc2474

Babiarz, J., Chan, K., & Baker, F. (2006). *Configuration guidelines for DiffServ service classes* (RFC 4594). IETF. https://www.rfc-editor.org/rfc/rfc4594

Vegesna, S. (2001). *IP quality of service*. Cisco Press.

## Routing — protocolli e teoria

Moy, J. (1998). *OSPF version 2* (RFC 2328). IETF. https://www.rfc-editor.org/rfc/rfc2328

Rekhter, Y., Li, T., & Hares, S. (2006). *A Border Gateway Protocol 4 (BGP-4)* (RFC 4271). IETF. https://www.rfc-editor.org/rfc/rfc4271

Savage, D., Ng, J., Moore, S., Slice, D., Paluch, P., & White, R. (2016). *Cisco's enhanced interior gateway routing protocol (EIGRP)* (RFC 7868). IETF. https://www.rfc-editor.org/rfc/rfc7868

Doyle, J., & Carroll, J. D. (2005). *Routing TCP/IP* (Vol. 1 & 2, 2nd ed.). Cisco Press.

Dijkstra, E. W. (1959). A note on two problems in connexion with graphs. *Numerische Mathematik, 1*(1), 269-271. https://doi.org/10.1007/BF01386390

## Configurazione Cisco IOS e routing

Lammle, T. (2019). *CCNA Cisco Certified Network Associate study guide* (8th ed.). Sybex.

Cisco Systems. (2024). *Cisco IOS IP routing configuration guide*. https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_basic/configuration/xe-17/irb-xe-17-book.html

Cisco Systems. (2024). *Cisco IOS QoS solutions configuration guide*. https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/qos_mqc/configuration/xe-17/qos-mqc-xe-17-book.html

Cisco Systems. (2024). *Cisco IOS security configuration guide: Control Plane Policing*. https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_data_acl/configuration/xe-17/sec-data-acl-xe-17-book.html

## IETF e processo di standardizzazione

Hoffman, P. (2012). *The Tao of IETF: A novice's guide to the Internet Engineering Task Force*. https://www.ietf.org/tao.html

Bradner, S. (1996). *The Internet standards process — Revision 3* (RFC 2026). IETF. https://www.rfc-editor.org/rfc/rfc2026

**Nota:** *Tutte le RFC citate sono gratuitamente accessibili su https://www.rfc-editor.org. I riferimenti alle guide di configurazione Cisco rimandano alla documentazione ufficiale, soggetta ad aggiornamenti periodici con il rilascio di nuove versioni di IOS-XE.*
