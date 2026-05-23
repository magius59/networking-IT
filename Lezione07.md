# Lezione 7

*Access Control List, Routing Inter-VLAN e Protocollo HTTP*

------------------------------------------------------------------------

# 1. Access Control List (ACL) nei Router Cisco

Le Access Control List (ACL) sono meccanismi di filtraggio del traffico di rete implementati nei router e nei firewall. Una ACL definisce un insieme ordinato di regole — dette ACE (Access Control Entry) — che stabiliscono quali pacchetti possano transitare attraverso un'interfaccia e quali debbano essere scartati.

Le ACL operano principalmente a livello 3 (Network) e a livello 4 (Transport) del modello OSI:

- Livello 3 — filtraggio basato su indirizzi IP sorgente e destinazione

- Livello 4 — filtraggio basato su protocollo (TCP, UDP, ICMP) e numeri di porta

Le ACL sono fondamentali per la segmentazione delle reti, la protezione dei server e l'implementazione di politiche di sicurezza perimetrale.

## 1.1 Applicazione delle ACL

Una ACL viene applicata a una specifica interfaccia del router indicando la direzione del traffico da filtrare:

- in ingresso (inbound) — il pacchetto viene esaminato prima di essere instradato

- in uscita (outbound) — il pacchetto viene esaminato dopo la decisione di routing, prima di uscire dall'interfaccia

Questa distinzione è operativamente importante: una ACL inbound agisce sulla base dell'indirizzo sorgente del traffico che entra nel router; una ACL outbound agisce sul traffico già instradato che sta per lasciare il router verso una destinazione.

Su ciascuna interfaccia è possibile applicare:

- una ACL in ingresso per IPv4

- una ACL in uscita per IPv4

- ACL separate per IPv6 (tramite la direttiva ipv6 traffic-filter)

| **Nota importante — Traffico generato dal router** |
|----|
| Le ACL filtrano esclusivamente il traffico che attraversa l'interfaccia (transit traffic). |
| Non filtrano il traffico generato dal router stesso (ad esempio pacchetti di routing dinamico, |
| messaggi SNMP, o sessioni SSH al router). Per proteggere il piano di gestione (management plane) |
| occorrono meccanismi distinti, come le Control Plane Policing (CoPP). |

## 1.2 Struttura di una ACL — Access Control Entry (ACE)

Ogni ACE specifica:

- l'azione da eseguire: permit oppure deny

- il protocollo (ip, tcp, udp, icmp, …)

- l'indirizzo sorgente con la relativa wildcard mask

- l'indirizzo destinazione con la relativa wildcard mask (solo nelle ACL estese)

- il numero di porta TCP/UDP (opzionale, solo nelle ACL estese)

Esempio di ACE estesa:

> permit tcp 192.168.20.0 0.0.0.255 host 192.168.10.100 eq www

Questa regola autorizza:

- traffico di protocollo TCP

- proveniente da qualsiasi host della rete 192.168.20.0/24

- diretto all'host 192.168.10.100

- sulla porta di destinazione 80 (HTTP, identificata dalla keyword www)

## 1.3 Elaborazione Sequenziale delle Regole

Le ACE vengono valutate in ordine strettamente sequenziale, dalla prima all'ultima. Quando un pacchetto soddisfa una regola:

1.  viene eseguita l'azione associata (permit o deny)

2.  l'elaborazione termina immediatamente

3.  le regole successive non vengono più analizzate

**L'ordine delle regole è quindi critico.** Regole più specifiche devono precedere regole più generiche. Un errore frequente è inserire una regola deny ampia prima di regole permit specifiche, con il risultato di bloccare traffico che avrebbe dovuto essere consentito.

## 1.4 Deny Implicito (Implicit Deny)

In calce a ogni ACL Cisco è presente implicitamente la regola:

> deny ip any any

Questa regola non è visibile nell'output di show ip access-lists ma viene applicata a ogni pacchetto che non ha trovato corrispondenza con nessuna ACE precedente. Il comportamento conseguente è denominato default-deny: tutto ciò che non è esplicitamente consentito viene bloccato.

Una ACL che contiene soltanto regole deny — o una ACL vuota — bloccherà quindi tutto il traffico.

## 1.5 Wildcard Mask

Nelle ACL Cisco la corrispondenza degli indirizzi IP viene espressa tramite una wildcard mask, che è il complemento a uno (bitwise NOT) della corrispondente subnet mask.

| **Subnet Mask** | **Wildcard Mask** | **Significato**                         |
|-----------------|-------------------|-----------------------------------------|
| 255.255.255.0   | 0.0.0.255         | Intera rete /24                         |
| 255.255.0.0     | 0.0.255.255       | Intera rete /16                         |
| 255.255.255.252 | 0.0.0.3           | Subnet /30 (4 indirizzi)                |
| 255.255.255.255 | 0.0.0.0           | Singolo host (equivalente a host)       |
| 0.0.0.0         | 255.255.255.255   | Qualsiasi indirizzo (equivalente a any) |

Nella wildcard mask:

- Il bit 0 significa "questo bit deve corrispondere esattamente"

- Il bit 1 significa "questo bit può avere qualsiasi valore"

Esempio: la specifica 192.168.10.0 0.0.0.255 corrisponde a tutti gli indirizzi da 192.168.10.0 a 192.168.10.255.

## 1.6 Keyword any e host

Cisco IOS fornisce due keyword abbreviate di uso comune:

**any**

La keyword any equivale alla coppia 0.0.0.0 255.255.255.255 e corrisponde a qualunque indirizzo IP.

**host**

La keyword host \<indirizzo\> equivale a \<indirizzo\> 0.0.0.0 e identifica un singolo host specifico.

## 1.7 ACL Standard ed Estese

### ACL Standard

Le ACL standard filtrano il traffico esclusivamente in base all'indirizzo IP sorgente. Sono semplici da configurare ma scarsamente selettive: non distinguono il tipo di traffico né la destinazione.

Intervallo di numeri assegnati storicamente:

- 1–99 (standard IPv4)

- 1300–1999 (esteso, introdotto in IOS 12.0)

Esempio:

> access-list 10 permit 192.168.10.0 0.0.0.255

### ACL Estese

Le ACL estese consentono di filtrare in base a:

- indirizzo IP sorgente

- indirizzo IP destinazione

- protocollo (TCP, UDP, ICMP, ecc.)

- porta TCP/UDP sorgente e/o destinazione

- flag TCP (es. established — per consentire risposte a sessioni TCP già avviate)

Intervallo di numeri assegnati storicamente:

- 100–199 (esteso IPv4)

- 2000–2699 (esteso, introdotto in IOS 12.0)

Esempio:

> access-list 101 permit tcp 192.168.20.0 0.0.0.255 host 192.168.10.100 eq 80

### ACL Nominate (Named ACL)

A partire da Cisco IOS 11.2 è possibile assegnare un nome descrittivo alle ACL:

> ip access-list extended DMZ_FILTER
>
> permit tcp 192.168.20.0 0.0.0.255 host 192.168.10.100 eq 80
>
> deny ip any any

I vantaggi delle ACL nominate rispetto a quelle numeriche sono:

- Maggiore leggibilità della configurazione

- Possibilità di eliminare singole ACE con il comando no \<n. sequenza\> senza dover riscrivere l'intera lista

- Possibilità di riordinare le ACE modificando i numeri di sequenza

## 1.8 Posizionamento delle ACL

La regola generale di posizionamento dipende dal tipo di ACL:

| **Tipo ACL** | **Posizionamento consigliato** | **Motivazione** |
|----|----|----|
| Estesa | Il più vicino possibile alla sorgente | Blocca il traffico indesiderato prima che percorra inutilmente la rete |
| Standard | Il più vicino possibile alla destinazione | Filtra solo su IP sorgente: applicarla vicino alla sorgente potrebbe bloccare tutto il traffico proveniente da quella rete, indipendentemente dalla destinazione |

| **📘 Approfondimento — Riflessione e ACL dinamiche** |
|----|
| Cisco IOS supporta funzionalità avanzate di filtraggio: |
| \- Reflexive ACL: generano automaticamente regole permit temporanee per consentire |
| il traffico di ritorno di sessioni TCP/UDP avviate dall'interno della rete. |
| \- Dynamic ACL (Lock-and-Key): attivano una regola permit temporanea dopo che un utente |
| si è autenticato con successo (tipicamente via Telnet o SSH). |
| Queste funzionalità sono oggi in gran parte superate dagli stateful firewall, ma rimangono |
| disponibili nelle piattaforme IOS tradizionali. |

## 1.9 Contatori dei Match e Verifica

Cisco IOS mantiene un contatore del numero di pacchetti che hanno corrisposto a ciascuna ACE. Il comando per visualizzarli è:

> show ip access-lists
>
> show ip access-lists DMZ_FILTER

Il conteggio è utile per verificare:

- che il traffico venga effettivamente filtrato

- quali regole sono operative e quali non vengono mai raggiunte

- la correttezza dell'implementazione prima della messa in produzione

I contatori possono essere azzerati con:

> clear ip access-list counters

## 1.10 ICMP Type 3 Code 13 — Communication Administratively Prohibited

Quando una ACL scarta un pacchetto, il router può restituire al mittente un messaggio ICMP:

- Type 3 — Destination Unreachable

- Code 13 — Communication Administratively Prohibited

Questo messaggio informa il client che il blocco è dovuto a una policy amministrativa (ad esempio una ACL) e non a un problema di routing o di host irraggiungibile.

**Nota:** il comportamento predefinito di Cisco IOS è di inviare questo messaggio ICMP. È possibile sopprimerlo con il comando ip unreachables a livello di interfaccia per evitare di rivelare informazioni sulla politica di filtraggio a potenziali attaccanti.

| **📘 Approfondimento — ICMP e filtraggio selettivo** |
|----|
| Un host potrebbe risultare irraggiungibile tramite ping ma essere accessibile via HTTP. |
| ping utilizza ICMP Echo Request (Type 8), mentre il browser utilizza TCP/80 o TCP/443. |
| Bloccare ICMP in una ACL non implica necessariamente il blocco degli altri servizi. |
| Viceversa, bloccare TCP/80 non impedisce il ping. Le ACL agiscono per protocollo e porta, |
| pertanto ogni servizio deve essere considerato separatamente nella definizione delle regole. |

# 2. VLAN e Routing Inter-VLAN

## 2.1 VLAN — Definizione e Funzione

Le VLAN sono già state introdotte nella lezione precedente. L’utilizzo di ACL consente di limitare opportunamente il traffico tra le VLAN e istituisce dei punti di controllo sui quali è possibile rilevare il traffico consentito ed i tentativi di accesso non consentiti (verificando gli Hit Counters delle VLAN).

| la rende un vettore di attacco potenziale (VLAN hopping). |
|-----------------------------------------------------------|

## 2.2 Trunk IEEE 802.1Q

Quando più VLAN devono essere trasportate su un unico collegamento fisico (ad esempio tra switch e router), si utilizza un trunk. Lo standard IEEE 802.1Q definisce il meccanismo di tagging: a ogni frame Ethernet viene aggiunto un campo di 4 byte (il tag 802.1Q) immediatamente dopo gli indirizzi MAC.

Struttura del tag 802.1Q (4 byte totali):

| **Campo** | **Dimensione** | **Descrizione** |
|----|----|----|
| TPID (Tag Protocol Identifier) | 16 bit | Valore fisso 0x8100 che identifica il frame come tagged |
| PCP (Priority Code Point) | 3 bit | Priorità QoS (802.1p) |
| DEI (Drop Eligible Indicator) | 1 bit | Indica se il frame può essere scartato in caso di congestione |
| VID (VLAN Identifier) | 12 bit | Identificatore VLAN (valori 0–4095; 0 e 4095 riservati) |

Il campo VID a 12 bit consente di identificare fino a 4094 VLAN utilizzabili (2^12 = 4096, sottratti i valori riservati 0 e 4095).

La VLAN nativa (native VLAN) è la VLAN il cui traffico transita sul trunk senza tag. Per default è la VLAN 1; in ambienti sicuri è consigliabile cambiarla con:

> switchport trunk native vlan \<id\>

## 2.3 Router-on-a-Stick

Con la tecnica Router-on-a-Stick il routing tra VLAN viene realizzato tramite un unico collegamento fisico (il trunk) tra switch e router. Il router utilizza sottointerfacce logiche, una per ciascuna VLAN, create sull'interfaccia fisica. Il router on a stick è stato introdotto nellal lezione precedente. Vale la pena di considerare che per forti quantità di traffico questa configurazione può diventare un collo di bottiglia (usa una sola interfaccia, con i suoi limiti di velocità, per far fare traffico alle diverse VLAN).

| **Approfondimento — Alternativa ai Router-on-a-Stick: Switch Layer 3** |
|----|
| In reti più grandi o con requisiti di prestazione elevati, il routing inter-VLAN viene tipicamente |
| realizzato tramite Switch Layer 3 (Multilayer Switch), che implementano il routing direttamente |
| nell'hardware (tramite ASIC) eliminando il collo di bottiglia del singolo trunk verso il router. |
| La configurazione utilizza Switched Virtual Interface (SVI). Ad esempio: |
| interface Vlan10 |
| ip address 192.168.10.1 255.255.255.0 |
| no shutdown |
| Questa soluzione offre latenza e throughput significativamente migliori rispetto al Router-on-a-Stick. |

## 2.4 Associazione tra VLAN e Indirizzi IP

Non esiste alcun vincolo tecnico che imponga una corrispondenza tra numero VLAN e subnet IP. Ad esempio, la VLAN 20 potrebbe utilizzare la rete 192.168.20.0/24 oppure la rete 10.1.1.0/24: la scelta è puramente amministrativa.

La prassi comune di far corrispondere il numero VLAN al terzo ottetto dell'indirizzo (es. VLAN 10 → 192.168.10.0/24) è una convenzione mnemonica che semplifica la gestione e la documentazione della rete.

## 2.5 Tabelle di Routing

Il router mantiene una routing table che indica come raggiungere ciascuna rete di destinazione. Il comando per visualizzarla è:

> show ip route

Le rotte possono essere:

- Direttamente connesse (codice C) — reti raggiungibili tramite interfacce attive del router

- Statiche (codice S) — configurate manualmente dall'amministratore

- Dinamiche — apprese tramite protocolli di routing (OSPF, EIGRP, BGP, ecc.)

Esempio di rotta statica:

> ip route 192.168.110.0 255.255.255.0 192.168.10.2

Questa configurazione indica che per raggiungere la rete 192.168.110.0/24 il router deve inviare i pacchetti al next-hop 192.168.10.2.

# 3. ACL per la Protezione di una DMZ

## 3.1 Concetto di DMZ

Una DMZ (Demilitarized Zone) è un segmento di rete isolato che ospita servizi accessibili dall'esterno (server web, mail, FTP) senza esporre la rete interna. Il traffico verso la DMZ è strettamente controllato da regole firewall o ACL.

Nel laboratorio la VLAN 10 svolge il ruolo di DMZ semplificata, contenendo:

- un server web all'indirizzo 192.168.10.100 (porta HTTP/80)

- un server FTP all'indirizzo 192.168.10.101 (porta FTP/21)

La VLAN 20 contiene i client autorizzati ad accedere ai servizi della DMZ.

## 3.2 Configurazione ACL per la DMZ

Obiettivo: consentire alla VLAN 20 l'accesso HTTP al server web e l'accesso FTP al server FTP; bloccare tutto il resto.

Tabella dei servizi ammessi:

| **Servizio**   | **Protocollo** | **Porta** | **Sorgente**    | **Destinazione** |
|----------------|----------------|-----------|-----------------|------------------|
| HTTP           | TCP            | 80        | 192.168.20.0/24 | 192.168.10.100   |
| FTP (comandi)  | TCP            | 21        | 192.168.20.0/24 | 192.168.10.101   |
| Tutto il resto | —              | —         | any             | any → DENY       |

Configurazione ACL:

> ip access-list extended DMZ
>
> permit tcp 192.168.20.0 0.0.0.255 host 192.168.10.100 eq 80
>
> permit tcp 192.168.20.0 0.0.0.255 host 192.168.10.101 eq 21
>
> deny ip any any

Applicazione della ACL in uscita sulla sottointerfaccia della VLAN 10:

> interface GigabitEthernet0/0.10
>
> ip access-group DMZ out

| **FTP in modalità attiva e passiva** |
|----|
| Il protocollo FTP utilizza due canali TCP separati: |
| \- Canale di controllo: porta 21 (per comandi e risposte) |
| \- Canale dati (modalità attiva): il server apre una connessione dalla porta 20 verso il client |
| \- Canale dati (modalità passiva / PASV): il client apre una connessione verso una porta |
| alta (\>1023) del server, negoziata tramite il canale di controllo |
|  |
| Una ACL che consente solo la porta 21 permette il canale di controllo ma può bloccare il |
| trasferimento dei dati in modalità attiva. Per gestire correttamente FTP in presenza di ACL |
| stateless occorre valutare la modalità di connessione oppure utilizzare un firewall stateful. |

Tabella delle porte principali per riferimento:

| **Servizio**    | **Protocollo** | **Porta** |
|-----------------|----------------|-----------|
| FTP (dati)      | TCP            | 20        |
| FTP (controllo) | TCP            | 21        |
| SSH             | TCP            | 22        |
| Telnet          | TCP            | 23        |
| SMTP            | TCP            | 25        |
| DNS             | UDP/TCP        | 53        |
| HTTP            | TCP            | 80        |
| HTTPS           | TCP            | 443       |
| RDP             | TCP            | 3389      |

# 4. Il Protocollo HTTP

## 4.1 Introduzione e Storia

HyperText Transfer Protocol (HTTP) è il protocollo applicativo alla base del World Wide Web. Fu progettato da Tim Berners-Lee al CERN nel 1989 come parte del sistema di ipertesto distribuito che avrebbe dato origine al Web.

Evoluzione delle versioni principali:

| **Versione** | **Anno** | **Caratteristiche principali** |
|----|----|----|
| HTTP/0.9 | 1991 | Protocollo minimale: unico metodo GET, risposta in solo HTML, nessun header |
| HTTP/1.0 | 1996 (RFC 1945) | Introduzione di header, metodi POST e HEAD, codici di stato, MIME type |
| HTTP/1.1 | 1997 (RFC 2068, aggiornato RFC 7230-7235) | Connessioni persistenti (keep-alive), pipelining, Host header obbligatorio, chunked transfer encoding |
| HTTP/2 | 2015 (RFC 7540) | Multiplexing di stream su una singola connessione TCP, compressione header HPACK, server push |
| HTTP/3 | 2022 (RFC 9114) | Trasporto su QUIC (UDP) invece di TCP: eliminazione del head-of-line blocking, 0-RTT connection establishment |

## 

## 4.2 Caratteristiche Fondamentali

HTTP è un protocollo:

- Applicativo — opera al livello 7 del modello OSI

- undefined— utilizza TCP come protocollo di trasporto (porta 80 per HTTP, porta 443 per HTTPS). HTTP/3 fa eccezione: usa QUIC su UDP.

- Stateless — ogni richiesta è indipendente; il server non conserva alcuna informazione sulle richieste precedenti dello stesso client

La natura stateless semplifica la scalabilità dei server ma richiede meccanismi aggiuntivi per mantenere lo stato delle sessioni. I meccanismi principali sono:

- Cookie HTTP (RFC 6265) — piccoli dati testuali memorizzati dal browser e inviati automaticamente in ogni richiesta

- Token di sessione — identificatori opachi trasmessi negli header o nei parametri URL

- JSON Web Token (JWT, RFC 7519) — token firmati che trasportano claims verificabili

## 4.3 URL — Uniform Resource Locator

Una URL (Uniform Resource Locator, RFC 3986) identifica univocamente una risorsa in rete.

Esempio:

> https://www.example.com:443/images/logo.png?size=large#section1

| **Componente** | **Esempio** | **Descrizione** |
|----|----|----|
| Schema (scheme) | https | Protocollo da utilizzare |
| Autorità (authority) | www.example.com:443 | Host e porta (la porta può essere omessa se è quella di default) |
| Percorso (path) | /images/logo.png | Percorso gerarchico della risorsa sul server |
| Query string | size=large | Parametri opzionali in formato chiave=valore, separati da & |
| Fragment | section1 | Identificatore di un'ancora nella pagina (elaborato solo dal client) |

## 4.4 Formato dei Messaggi HTTP/1.1

HTTP/1.1 è un protocollo testuale: i messaggi sono sequenze di righe ASCII. La struttura è identica per richieste e risposte, con la sola eccezione della prima riga.

### Richiesta HTTP

Una richiesta HTTP è composta da:

4.  Request line: metodo, URI e versione del protocollo

5.  Header fields: coppie nome:valore, uno per riga

6.  Riga vuota (CRLF) che separa header e body

7.  Body (opzionale): dati inviati al server (assente nelle richieste GET)

Esempio completo di richiesta GET:

> GET /index.html HTTP/1.1
>
> Host: www.example.com
>
> User-Agent: Mozilla/5.0
>
> Accept: text/html,application/xhtml+xml
>
> Accept-Language: it-IT,it;q=0.9
>
> Accept-Encoding: gzip, deflate, br
>
> Connection: keep-alive
>
> (body vuoto per le richieste GET)

Esempio di richiesta POST con body:

> POST /login HTTP/1.1
>
> Host: www.example.com
>
> Content-Type: application/x-www-form-urlencoded
>
> Content-Length: 27
>
> username=admin&password=1234

### Risposta HTTP

Una risposta HTTP è composta da:

8.  Status line: versione del protocollo, codice di stato e reason phrase

9.  Header fields

10. Riga vuota (CRLF)

11. Body (opzionale): il contenuto della risorsa richiesta

Esempio di risposta:

> HTTP/1.1 200 OK
>
> Date: Mon, 12 May 2025 10:00:00 GMT
>
> Server: Apache/2.4.54
>
> Content-Type: text/html; charset=UTF-8
>
> Content-Length: 1234
>
> Connection: keep-alive
>
> \<!DOCTYPE html\>
>
> \<html\>...\</html\>

## 4.5 Metodi HTTP

| **Metodo** | **Idempotente** | **Safe** | **Descrizione** |
|----|----|----|----|
| GET | Sì | Sì | Richiede una risorsa; non modifica lo stato del server |
| HEAD | Sì | Sì | Come GET ma la risposta non contiene body; usato per verificare esistenza/metadati |
| POST | No | No | Invia dati al server; crea una nuova risorsa o avvia un'elaborazione |
| PUT | Sì | No | Sostituisce completamente la risorsa indicata dall'URI con il body inviato |
| PATCH | No | No | Modifica parzialmente la risorsa |
| DELETE | Sì | No | Elimina la risorsa indicata dall'URI |
| OPTIONS | Sì | Sì | Restituisce i metodi supportati per una risorsa; usato nelle richieste CORS preflight |
| CONNECT | No | No | Apre un tunnel TCP (tipicamente per HTTPS attraverso proxy) |
| TRACE | Sì | Sì | Restituisce la richiesta così come ricevuta dal server (diagnostica) |

**Safe:** un metodo è safe se non modifica lo stato del server.

**Idempotente:** un metodo è idempotente se applicarlo più volte produce lo stesso risultato della singola applicazione.

## 4.6 Codici di Stato HTTP

I codici di stato HTTP sono numeri a tre cifre raggruppati in cinque classi:

| **Classe** | **Range** | **Significato** | **Esempi** |
|----|----|----|----|
| 1xx | 100–199 | Informational — richiesta ricevuta, elaborazione in corso | 100 Continue, 101 Switching Protocols |
| 2xx | 200–299 | Success — richiesta ricevuta, compresa e accettata | 200 OK, 201 Created, 204 No Content |
| 3xx | 300–399 | Redirection — ulteriori azioni necessarie per completare la richiesta | 301 Moved Permanently, 302 Found, 304 Not Modified |
| 4xx | 400–499 | Client Error — la richiesta contiene un errore del client | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found |
| 5xx | 500–599 | Server Error — il server non ha potuto soddisfare una richiesta valida | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable |

## 4.7 Header HTTP Principali

Gli header HTTP trasportano metadati sulla richiesta o sulla risposta. Gli header possono essere generali (applicabili a richieste e risposte), specifici della richiesta, specifici della risposta, o relativi al contenuto (entity headers).

### Header di richiesta comuni

| **Header** | **Descrizione** | **Esempio** |
|----|----|----|
| Host | Host e porta del server (obbligatorio in HTTP/1.1) | Host: www.example.com |
| User-Agent | Identificazione del client | User-Agent: Mozilla/5.0 |
| Accept | Tipi di contenuto accettati (MIME) | Accept: text/html, application/json |
| Accept-Encoding | Algoritmi di compressione supportati | Accept-Encoding: gzip, br |
| Authorization | Credenziali di autenticazione | Authorization: Bearer \<token\> |
| Cookie | Cookie trasmessi al server | Cookie: session=abc123 |
| Content-Type | Tipo MIME del body (nelle richieste con body) | Content-Type: application/json |
| Content-Length | Dimensione del body in byte | Content-Length: 348 |

### Header di risposta comuni

| **Header** | **Descrizione** | **Esempio** |
|----|----|----|
| Content-Type | Tipo MIME della risposta | Content-Type: text/html; charset=UTF-8 |
| Content-Length | Dimensione del body in byte | Content-Length: 1024 |
| Set-Cookie | Imposta un cookie nel browser | Set-Cookie: id=a3; HttpOnly; Secure |
| Location | URI di reindirizzamento (usato con 3xx) | Location: https://example.com/new |
| Cache-Control | Direttive di caching | Cache-Control: max-age=3600 |
| Strict-Transport-Security | Forza HTTPS per il dominio (HSTS) | Strict-Transport-Security: max-age=31536000 |
| Server | Identificazione del software server | Server: nginx/1.24.0 |

## 4.8 MIME Type

HTTP utilizza i MIME Type (Multipurpose Internet Mail Extensions, RFC 2045) per identificare il tipo di contenuto trasportato nel body. La struttura è tipo/sottotipo.

| **MIME Type**          | **Contenuto**           |
|------------------------|-------------------------|
| text/html              | Documento HTML          |
| text/css               | Foglio di stile CSS     |
| text/plain             | Testo non formattato    |
| application/javascript | Script JavaScript       |
| application/json       | Dati in formato JSON    |
| application/xml        | Dati in formato XML     |
| image/png              | Immagine PNG            |
| image/jpeg             | Immagine JPEG           |
| image/svg+xml          | Grafica vettoriale SVG  |
| audio/mpeg             | Audio MP3               |
| video/mp4              | Video MP4               |
| application/pdf        | Documento PDF           |
| multipart/form-data    | Form con upload di file |

## 4.9 Connessioni Persistenti e HTTP Pipelining

In HTTP/1.0 ogni richiesta richiedeva l'apertura di una nuova connessione TCP (three-way handshake + chiusura), con un overhead significativo.

HTTP/1.1 introduce le connessioni persistenti (keep-alive): la connessione TCP rimane aperta dopo la risposta, consentendo l'invio di richieste successive senza riaprire il canale. L'header Connection: keep-alive mantiene la connessione aperta; Connection: close la chiude dopo la risposta.

Il pipelining HTTP/1.1 consente di inviare più richieste in sequenza senza attendere la risposta della precedente. Tuttavia, le risposte devono arrivare nello stesso ordine (head-of-line blocking), limitando l'utilità pratica di questa funzionalità.

| **Approfondimento — HTTP/2 e Multiplexing** |
|----|
| HTTP/2 risolve il head-of-line blocking di HTTP/1.1 tramite il concetto di stream: |
| una singola connessione TCP trasporta più stream logici indipendenti identificati da un ID. |
| Richieste e risposte vengono suddivise in frame binari (DATA, HEADERS, PRIORITY, ecc.) |
| che possono essere interleaved sulla connessione. |
|  |
| Un'ulteriore ottimizzazione è la compressione degli header tramite l'algoritmo HPACK, |
| che sfrutta una tabella di header già trasmessi per ridurre la ridondanza. |
|  |
| HTTP/3 elimina completamente TCP: utilizza QUIC, un protocollo di trasporto costruito su |
| UDP che implementa nativamente il controllo di congestione, la correzione degli errori e |
| la crittografia (TLS 1.3 integrato). Questo elimina il head-of-line blocking a livello di |
| trasporto che affliggeva HTTP/2 su TCP in presenza di perdita di pacchetti. |

## 4.10 Analisi del Traffico HTTP con Wireshark

Wireshark è uno strumento di analisi del traffico di rete (packet analyzer) che consente di catturare e ispezionare i frame a livello di singolo byte.

Per analizzare una sessione HTTP:

12. Applicare il filtro di visualizzazione: http

13. Individuare il TCP three-way handshake (SYN, SYN-ACK, ACK)

14. Osservare la richiesta HTTP (GET o POST) con i relativi header

15. Osservare la risposta HTTP con codice di stato e body

16. Osservare la chiusura della connessione TCP (FIN-ACK)

| **Nota — HTTP e confidenzialità** |
|----|
| HTTP trasmette tutto il contenuto in chiaro, inclusi header, cookie e credenziali. |
| Wireshark può catturare e visualizzare username e password trasmessi con HTTP. |
| HTTPS (HTTP over TLS, porta 443) cifra il payload: Wireshark mostra i dati TLS cifrati |
| e non è in grado di decifrare il contenuto a meno che non disponga della chiave privata |
| del server o delle chiavi di sessione (tramite file SSLKEYLOGFILE). |
| In contesti di laboratorio su rete locale, l'uso di HTTP non cifrato è accettabile. |
| In produzione, HTTP non cifrato non deve mai essere utilizzato per dati sensibili. |

# 5. Esercizi di Laboratorio

Per gli esercizi si consiglia l’utilizzo di Cisco Packet Tracer, software di simulazione di rete gratuito distribuito da Cisco.

## Esercizio 1 — Configurazione ACL per una DMZ

### Scenario

La rete di laboratorio presenta la seguente topologia:

- VLAN 10 (192.168.10.0/24) — DMZ con server web (10.100) e server FTP (10.101)

- VLAN 20 (192.168.20.0/24) — client autorizzati

- Router con sottointerfacce GigabitEthernet0/0.10 e GigabitEthernet0/0.20

### Obiettivo

Consentire esclusivamente:

- Traffico HTTP dalla VLAN 20 verso 192.168.10.100

- Traffico FTP dalla VLAN 20 verso 192.168.10.101

- Bloccare tutto il resto

### Configurazione soluzione

> ip access-list extended DMZ
>
> permit tcp 192.168.20.0 0.0.0.255 host 192.168.10.100 eq 80
>
> permit tcp 192.168.20.0 0.0.0.255 host 192.168.10.101 eq 21
>
> deny ip any any
>
> !
>
> interface GigabitEthernet0/0.10
>
> ip access-group DMZ out

### Verifica

> show ip access-lists DMZ
>
> show ip interface GigabitEthernet0/0.10

### Domande di riflessione

17. Perché la ACL viene applicata in outbound sulla sottointerfaccia della VLAN 10 e non in inbound sulla VLAN 20?

18. Cosa succede se un host della VLAN 1 tenta di raggiungere il server web?

19. Come si modifica la configurazione per consentire anche il traffico HTTPS (porta 443)?

20. Un ping dalla VLAN 20 verso 192.168.10.100 funzionerà con questa ACL? Perché?

## Esercizio 2 — Analisi di una Sessione HTTP

### Obiettivo

Osservare con Wireshark il funzionamento di una connessione HTTP verso il server della DMZ.

### Attività

21. Avviare la cattura Wireshark sull'interfaccia di rete del client (VLAN 20)

22. Aprire un browser e collegarsi a http://192.168.10.100

23. Fermare la cattura

24. Applicare il filtro: tcp.port == 80

### Elementi da individuare nella cattura

- Il three-way handshake TCP (SYN, SYN-ACK, ACK)

- La richiesta HTTP GET con gli header

- Il codice di risposta HTTP (200 OK o altro)

- Il body della risposta (HTML del server)

- La chiusura della connessione TCP

### Domande di analisi

25. Qual è l'indirizzo IP sorgente e destinazione nel pacchetto TCP SYN?

26. Quale porta TCP effimera ha scelto il client per questa connessione?

27. Quali header HTTP sono presenti nella richiesta GET?

28. Qual è il Content-Type della risposta del server?

# 6. Bibliografia

Di seguito le opere di riferimento principali per i contenuti di questa unità didattica, in formato APA 7ª edizione.

## Reti di calcolatori e protocolli di rete

Forouzan, B. A. (2021). Data communications and networking (6th ed.). McGraw-Hill Education.

Tanenbaum, A. S., & Wetherall, D. J. (2021). Computer networks (6th ed.). Pearson.

Kurose, J. F., & Ross, K. W. (2022). Computer networking: A top-down approach (8th ed.). Pearson.

## Cisco IOS e ACL

Cisco Systems. (2023). Security configuration guide: Access control lists. Cisco Press. https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_data_acl/

Odom, W. (2022). CCNA 200-301 official cert guide, volume 1. Cisco Press.

Odom, W. (2022). CCNA 200-301 official cert guide, volume 2. Cisco Press.

## IEEE 802.1Q e VLAN

IEEE Computer Society. (2018). IEEE standard for local and metropolitan area networks — bridges and bridged networks (IEEE Std 802.1Q-2018). Institute of Electrical and Electronics Engineers. https://doi.org/10.1109/IEEESTD.2018.8403927

## HTTP

Fielding, R., Nottingham, M., & Reschke, J. (2022). HTTP semantics (RFC 9110). Internet Engineering Task Force. https://doi.org/10.17487/RFC9110

Fielding, R., Nottingham, M., & Reschke, J. (2022). HTTP/1.1 (RFC 9112). Internet Engineering Task Force. https://doi.org/10.17487/RFC9112

Bishop, M. (2022). HTTP/2 (RFC 9113). Internet Engineering Task Force. https://doi.org/10.17487/RFC9113

Bishop, M. (2022). HTTP/3 (RFC 9114). Internet Engineering Task Force. https://doi.org/10.17487/RFC9114

Berners-Lee, T., Fielding, R., & Masinter, L. (2005). Uniform Resource Identifier (URI): Generic syntax (RFC 3986). Internet Engineering Task Force. https://doi.org/10.17487/RFC3986

Barth, A. (2011). HTTP state management mechanism (RFC 6265). Internet Engineering Task Force. https://doi.org/10.17487/RFC6265

## Sicurezza di rete

Stallings, W. (2022). Network security essentials: Applications and standards (7th ed.). Pearson.

Cheswick, W. R., Bellovin, S. M., & Rubin, A. D. (2003). Firewalls and internet security: Repelling the wily hacker (2nd ed.). Addison-Wesley.
