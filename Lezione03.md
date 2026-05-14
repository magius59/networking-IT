**Lezione 3**

Il Livello Fisico e il Livello Data Link nel Modello OSI

# 1. Il livello fisico nel modello OSI

Il livello fisico (Layer 1) e il primo livello del modello OSI ed e responsabile della trasmissione e ricezione dei segnali grezzi sul mezzo trasmissivo. Non si occupa del significato dei bit, ma esclusivamente della loro rappresentazione fisica: tensioni elettriche, impulsi luminosi o onde radio. E l'unico livello che interagisce direttamente con il mezzo fisico.

Le sue funzioni comprendono:

- trasmissione e ricezione dei segnali fisici (elettrici, ottici, radio);

- definizione dei connettori e delle interfacce meccaniche;

- specifica delle caratteristiche dei cavi (impedenza, attenuazione, capacita);

- modulazione e demodulazione dei segnali;

- codifica fisica dei bit (es. Manchester encoding, 4B/5B, NRZ, PAM4);

- sincronizzazione di clock tra trasmettitore e ricevitore;

- definizione delle velocita di trasmissione e delle modalita duplex;

- gestione dei mezzi trasmissivi e delle frequenze radio.

**Espansione — codifiche di linea:** *La codifica di linea e il metodo con cui i bit logici (0 e 1) vengono rappresentati come variazioni del segnale fisico. Non tutti i metodi trasmettono un bit per simbolo. Ad esempio:\
NRZ (Non-Return-to-Zero) mappa direttamente 0 e 1 in livelli di tensione;\
Manchester encoding codifica ogni bit come una transizione (usata in 10Base-T);\
4B/5B mappa 4 bit in simboli a 5 bit per garantire la sincronizzazione;\
PAM4 (Pulse Amplitude Modulation 4-level) usa 4 livelli di ampiezza per trasmettere 2 bit per simbolo, impiegata in Ethernet a 25/100/400 Gbps.*

# 2. Mezzi trasmissivi e interfacce fisiche

I segnali di rete possono essere trasportati attraverso tre categorie principali di mezzi trasmissivi, ciascuna con caratteristiche fisiche, prestazioni e ambiti di impiego distinti:

- Rame (doppino intrecciato, cavo coassiale): trasmissione di segnali elettrici. Economico, diffuso, soggetto ad attenuazione e interferenze elettromagnetiche.

- Fibra ottica (monomodale, multimodale): trasmissione di impulsi luminosi. Alta velocita, grandi distanze, immunita EMI.

- Wireless (Wi-Fi, Bluetooth, LTE, 5G): trasmissione via onde radio. Mobilita, ma soggetto a interferenze, attenuazione e problemi di sicurezza.

## 2.1 Connettori RJ45 e RJ11

Tra i connettori piu comuni nelle reti cablate troviamo:

- RJ45 (Registered Jack 45): connettore a 8 pin (8P8C - 8 Position 8 Contact) utilizzato nelle reti Ethernet su doppino intrecciato. E lo standard fisico delle reti LAN moderne. Il nome tecnico corretto e 8P8C, ma RJ45 e universalmente usato in ambito di rete.

- RJ11 (Registered Jack 11): connettore a 6 pin (6P2C o 6P4C) utilizzato nelle connessioni telefoniche tradizionali (POTS) e nei modem DSL. E fisicamente piu stretto dell'RJ45 e non compatibile.

**Precisazione:** *Il termine 'RJ45' e tecnicamente impreciso: RJ45 e uno standard di cablaggio telefonico specifico. Il connettore usato per Ethernet e fisicamente un 8P8C. Tuttavia, 'RJ45' e il termine universalmente adottato nel settore networking ed e accettato come sinonimo.*

*Per chi si appassionasse all’argomento: https://www.linkedin.com/pulse/epic-network-battles-history-rj45-vs-8p8c-henry-mckelvey-mis/*

# 3. Trasmissione dei segnali e modulazione

## 3.1 Modulazione di ampiezza (AM) e di frequenza (FM)

La modulazione e il processo di variare le proprieta di un segnale portante (carrier) in funzione del segnale informativo (modulante). Le due forme fondamentali analogiche sono:

- Modulazione di ampiezza (AM - Amplitude Modulation): l'ampiezza della portante varia proporzionalmente al segnale trasmesso. La frequenza rimane costante. Storicamente usata nelle trasmissioni radio a onde medie e corte, nelle comunicazioni aeronautiche (banda VHF 118-137 MHz) e nelle prime trasmissioni televisive. Mantiene buona intelligibilità anche con rapporti segnale/rumore sfavorevoli, ma è vulnerabile alle interferenze impulsive.

- Modulazione di frequenza (FM - Frequency Modulation): la frequenza della portante varia in funzione del segnale, mantenendo costante l'ampiezza. Offre qualità audio superiore all'AM e maggiore immunità al rumore di ampiezza. Usata nelle trasmissioni radiofoniche FM (87.5-108 MHz), nelle comunicazioni radio professionali e in alcune comunicazioni satellitari.

**Espansione — modulazioni digitali:** *Le reti moderne usano modulazioni digitali molto più efficienti: QAM (Quadrature Amplitude Modulation) varia simultaneamente ampiezza e fase del segnale per trasmettere più bit per simbolo. 256-QAM trasmette 8 bit per simbolo (2^8=256 stati); 4096-QAM (usato in Wi-Fi 6E e DOCSIS 3.1) ne trasmette 12. PSK (Phase Shift Keying) varia solo la fase. OFDM (Orthogonal Frequency Division Multiplexing) divide la banda in molte sotto-portanti ortogonali, ciascuna modulata con QAM o PSK, rendendo la trasmissione robusta contro interferenze selettive in frequenza (multipath fading).*

## 3.2 Tecniche di modulazione nelle reti wireless

Le reti Wi-Fi (IEEE 802.11) utilizzano diverse tecniche di modulazione e accesso:

|  |  |  |
|----|----|----|
| **Tecnica** | **Descrizione** | **Standard tipici** |
| DSSS | Direct Sequence Spread Spectrum: il segnale viene distribuito su una banda larga tramite una sequenza pseudocasuale. Robusto contro interferenze narrowband. | 802.11b (2.4 GHz) |
| OFDM | Orthogonal Frequency Division Multiplexing: suddivide il canale in molte sotto-portanti ortogonali, ciascuna modulata con QAM/PSK. Molto efficiente e robusto al multipath. | 802.11a/g/n/ac/ax |
| OFDMA | Versione multi-utente di OFDM: le sotto-portanti vengono assegnate dinamicamente a utenti diversi nello stesso slot temporale. Riduce la latenza e aumenta l'efficienza in reti dense. | 802.11ax (Wi-Fi 6/6E) |
| MU-MIMO | Multiple Input Multiple Output multi-utente: trasmissione simultanea a più client tramite antenne multiple e beamforming. Aumenta la capacita complessiva. | 802.11ac Wave 2, 802.11ax |

Le reti GSM (2G) utilizzano TDMA (Time Division Multiple Access) combinato con FDMA: il canale radio e diviso in frequenze (FDMA), e ogni frequenza e suddivisa in 8 slot temporali (TDMA). Ogni slot e assegnato a una comunicazione. I canali GSM si distinguono in canali di controllo (per segnalazione, autenticazione, gestione della cella) e canali di traffico (per voce e dati).

## 3.3 Banda, throughput e teorema di Shannon

E fondamentale distinguere tre concetti correlati ma distinti:

- Larghezza di banda (bandwidth): l'intervallo di frequenze disponibile per la trasmissione, misurato in Hz. Determina la capacita teorica massima del canale.

- Link rate (data rate): la velocita nominale di trasmissione del collegamento fisico, misurata in bps. E’ una proprietà dell'hardware e della tecnologia usata.

- Throughput (goodput): la quantità reale di dati utili (escludendo overhead di protocollo) trasferiti nell'unita di tempo. E sempre inferiore al link rate.

Le cause del divario tra link rate e throughput includono:

- overhead degli header di protocollo (Ethernet, IP, TCP);

- meccanismi di controllo del flusso e della congestione;

- collisioni e ritrasmissioni;

- interferenze ed errori di trasmissione;

- frame di controllo e segnalazione;

- jitter e buffering.

**Espansione — Teorema di Nyquist e di Shannon:** *Il teorema di Nyquist (1928) stabilisce la capacità massima di un canale ideale senza rumore: C = 2B \* log2(M) bit/s, dove B e la larghezza di banda in Hz e M il numero di livelli del segnale.\
Il teorema di Shannon-Hartley (1948) estende il risultato al canale reale con rumore: C = B \* log2(1 + S/N) bit/s, dove S/N e il rapporto segnale-rumore. Shannon dimostra che non esiste alcun sistema di codifica in grado di superare questo limite, indipendentemente dalla modulazione usata. Il teorema di campionamento di Nyquist-Shannon afferma che per ricostruire fedelmente un segnale analogico di banda B, la frequenza di campionamento deve essere almeno 2B campioni/secondo. L'audio CD usa 44.100 campioni/secondo per un segnale audio fino a (circa) 22.000 Hz.*

# 4. Trasmissioni su rame: cablaggio Ethernet

## 4.1 UTP e STP

I principali tipi di cavo Ethernet su rame utilizzano coppie di fili intrecciati (twisted pair). L'intreccio riduce l'interferenza elettromagnetica per un principio fisico: i campi magnetici indotti su ciascun semigiro di torsione si oppongono e si cancellano parzialmente (cancellazione per accoppiamento). Piu e fitto l'intreccio, maggiore e l'attenuazione del crosstalk (e maggiore il consumo di rame).

- UTP (Unshielded Twisted Pair): coppie intrecciate senza schermatura esterna. Economico, flessibile, facile da installare. Suscettibile alle interferenze EMI esterne. Standard nelle LAN office.

- STP (Shielded Twisted Pair): aggiunge una schermatura metallica (foglio o treccia) intorno alle coppie o all'intero cavo. Riduce l'emissione EMI e la suscettibilita alle interferenze esterne. Usato in ambienti industriali, ospedali, studi di registrazione. Richiede messa a terra corretta per essere efficace.

- FTP (Foiled Twisted Pair): schermatura a foglio su tutte le coppie (ma non su ciascuna singola coppia). Meno rigido dell'STP.

- S/FTP: schermatura a treccia esterna + foglio su ogni coppia. Massima schermatura, usato in ambienti con forte disturbo EMI.

Il crosstalk è l'interferenza tra coppie adiacenti all'interno dello stesso cavo. Si distinguono:

- NEXT (Near-End Crosstalk): interferenza rilevata all'estremità trasmittente. E il parametro piu critico per le prestazioni.

- FEXT (Far-End Crosstalk): interferenza rilevata all'estremità opposta.

- ELFEXT / ACR-F (Attenuation-to-Crosstalk Ratio Far-End): rapporto tra attenuazione del segnale e FEXT.

## 4.2 Categorie dei cavi Ethernet

|  |  |  |  |  |
|----|----|----|----|----|
| **Categoria** | **Banda passante** | **Velocita supportata** | **Distanza max** | **Utilizzo tipico** |
| Cat 3 | 16 MHz | 10 Mbps (10Base-T) | 100 m | Telefonia, reti obsolete 10Base-T |
| Cat 5 | 100 MHz | 100 Mbps (100Base-TX) | 100 m | Fast Ethernet - obsoleto |
| Cat 5e | 100 MHz | 1 Gbps (1000Base-T) | 100 m | Gigabit Ethernet - ancora diffuso |
| Cat 6 | 250 MHz | 1 Gbps (100 m), 10 Gbps (55 m) | 100 m / 55 m | Gigabit e 10G su distanze brevi |
| Cat 6a | 500 MHz | 10 Gbps (10GBase-T) | 100 m | 10 Gbps a piena distanza |
| Cat 7 | 600 MHz | 10 Gbps | 100 m | Data center, schermatura S/FTP |
| Cat 8 | 2000 MHz | 25/40 Gbps (25/40GBase-T) | 30 m | Top-of-rack data center |

## 4.3 Limiti di distanza e segmentazione

Per Ethernet su rame (IEEE 802.3), il limite di 100 metri per segmento e definito dallo standard ed e determinato principalmente dall'attenuazione del segnale e dai requisiti di timing del CSMA/CD (il segnale deve propagarsi avanti e indietro nel tempo di trasmissione di un frame minimo).

Il limite si decompone convenzionalmente in:

- 90 metri: cablaggio permanente (tra patch panel e presa a muro), detto channel link.

- 10 metri: patch cable complessivi (lato switch + lato dispositivo).

Per distanze superiori si usano:

- Repeater / hub (obsoleti): rigenerano il segnale, ma condividono il dominio di collisione.

- Switch: rigenerano il frame a livello 2 su ogni porta, senza limite di distanza cumulativa.

- Fibra ottica: nessuna limitazione pratica per distanze LAN e WAN.

**Nota: 10GBase-T:** *A 10 Gbps su Cat 6a, la distanza massima rimane 100 m. Su Cat 6 (non augmented), la distanza si riduce a 55 m per via dei maggiori requisiti di attenuazione del crosstalk ad alta frequenza.*

## 4.4 Standard di cablaggio T568A e T568B

I cavi Ethernet RJ45 devono rispettare uno degli standard di cablaggio TIA/EIA-568 che definisce l'ordine dei fili nei pin del connettore. I due standard differiscono solo nell'ordine delle coppie arancio e verde.

|  |  |  |  |  |
|----|----|----|----|----|
| **Pin** | **T568A** | **T568B** | **Coppia** | **Funzione in 100/1000Base-T** |
| 1 | Bianco/Verde | Bianco/Arancio | Coppia 3 (A) / Coppia 2 (B) | TX+ (trasmissione positivo) |
| 2 | Verde | Arancio | Coppia 3 (A) / Coppia 2 (B) | TX- (trasmissione negativo) |
| 3 | Bianco/Arancio | Bianco/Verde | Coppia 2 (A) / Coppia 3 (B) | RX+ (ricezione positivo) |
| 4 | Blu | Blu | Coppia 1 | Usato in 1000Base-T (BI_DA+) |
| 5 | Bianco/Blu | Bianco/Blu | Coppia 1 | Usato in 1000Base-T (BI_DA-) |
| 6 | Arancio | Verde | Coppia 2 (A) / Coppia 3 (B) | RX- (ricezione negativo) |
| 7 | Bianco/Marrone | Bianco/Marrone | Coppia 4 | Usato in 1000Base-T (BI_DB+) |
| 8 | Marrone | Marrone | Coppia 4 | Usato in 1000Base-T (BI_DB-) |

**Importante:** *La regola fondamentale e che entrambe le estremità del cavo straight-through (patch cable) devono usare lo stesso standard (entrambe T568A o entrambe T568B). Il cavo crossover (per collegare due dispositivi dello stesso tipo, come PC-PC o switch-switch) aveva una estremità T568A e l'altra T568B. Tuttavia, gli switch e le schede di rete moderne supportano Auto-MDIX (IEEE 802.3ab), che rileva automaticamente e scambia i pin TX/RX, rendendo il cavo crossover obsoleto nella pratica.*

## 4.5 Crimpatura dei cavi

La crimpatura e il processo di fissaggio meccanico e elettrico dei conduttori nei pin del connettore RJ45 tramite una pinza crimpatrice. E’ una competenza pratica fondamentale per l'installazione di reti (non per la gestione o il progetto; se non facciamo installazione, difficilmente ne avremo bisogno; detto questo chi scrive ne ha due 😊).

Procedura corretta:

1.  Tagliare il cavo a misura con un tronchesino.

2.  Rimuovere circa 3 cm di guaina esterna con lo spellacavi, senza danneggiare l'isolante dei fili.

3.  Separare e raddrizzare le 8 coppie di fili.

4.  Ordinare i fili secondo lo standard scelto (T568A o T568B).

5.  Tagliare i fili in modo uniforme a circa 1.2 cm dalla guaina.

6.  Inserire i fili nel connettore RJ45 mantenendo l'ordine. La guaina deve entrare nel connettore per almeno 6-8 mm.

7.  Crimpare con la pinza: i pin perforano l'isolante dei singoli fili stabilendo il contatto elettrico.

8.  Verificare con il tester la continuità e l'ordine corretto.

Errori comuni e conseguenze:

- Ordine sbagliato dei fili: il tester mostra inversioni o split pair; il cavo non funziona o funziona male a velocita ridotta.

- Split pair: due fili di coppie diverse usati come coppia funzionale. Il tester di base può non rilevarlo, ma le prestazioni sono gravemente degradate per via del crosstalk.

- Guaina non inserita nel connettore: il cavo e soggetto a rottura meccanica nel punto di giunzione.

- Pin non crimpati correttamente: contatto intermittente, difficile da diagnosticare.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><strong>Esercizio 1 - Verifica di un cavo Ethernet</strong></td>
</tr>
<tr>
<td><p>Obiettivo: verificare il corretto cablaggio di un cavo Ethernet e diagnosticare eventuali anomalie.</p>
<p>Strumenti: tester Ethernet (almeno base; preferibilmente cable analyzer), cavo RJ45, pinza crimpatrice.</p>
<p>Attività:</p>
<ol start="9" type="1">
<li><p>Collegare entrambe le estremita del cavo al tester. Avviare il test di continuita.</p></li>
<li><p>Verificare che i LED si accendano nell'ordine corretto: 1-2-3-4-5-6-7-8 su entrambe le estremita in sequenza.</p></li>
<li><p>Controllare eventuali inversioni (due LED che si accendono contemporaneamente su pin diversi) o interruzioni (LED che non si accende).</p></li>
<li><p>Verificare la categoria del cavo leggendo la stampigliatura sulla guaina esterna.</p></li>
</ol>
<p>Domande:</p>
<ul>
<li><p>Quale sequenza di LED indica un cavo T568B corretto?</p></li>
<li><p>Cosa indica un tester che mostra '1-2-3-6-4-5-7-8' al posto di '1-2-3-4-5-6-7-8'?</p></li>
<li><p>Un tester economico mostra una sequenza perfetta ma la connessione Gigabit non funziona. Qual e la possibile causa?</p></li>
<li><p>Cosa si intende per 'split pair' e perche e pericoloso?</p></li>
</ul></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><strong>Soluzione</strong></td>
</tr>
<tr>
<td><p><strong>1. Sequenza LED corretta per T568B:</strong></p>
<p>Un cavo straight-through T568B corretto mostra la sequenza 1-2-3-4-5-6-7-8 su entrambe le estremità in modo speculare: quando si accende il LED del pin 1 su un'estremità, si accende il LED del pin 1 anche sull'altra. Questo conferma che ogni pin e collegato al pin corrispondente senza incroci.</p>
<p><strong>2. Sequenza '1-2-3-6-4-5-7-8':</strong></p>
<p>Questa e la sequenza tipica di un cavo crossover (T568A su un'estremita, T568B sull'altra). I pin 3 e 6 si incrociano con 1 e 2, scambiando le coppie TX e RX. Un cavo crossover e corretto per collegare due dispositivi dello stesso tipo (PC-PC, switch-switch) su hardware senza Auto-MDIX. Su switch moderni con Auto-MDIX funzionera comunque, ma e un cavo non standard per una patch cable.</p>
<p><strong>3. Tester OK ma Gigabit non funziona:</strong></p>
<p>La causa probabile è uno split pair. Un tester di continuità economico verifica solo che ogni pin sia connesso al pin corrispondente, non che i fili appartengano alla stessa coppia fisica. In uno split pair, i fili di due coppie diverse vengono usati come coppia logica: il test di continuità passa (i pin sono collegati), ma le due coppie divise non hanno la schermatura reciproca (twisting) necessaria per cancellare il crosstalk. A 100 Mbps il collegamento può funzionare; a 1000 Mbps, che usa tutte e 4 le coppie con requisiti NEXT molto stringenti, il crosstalk eccessivo causa errori e il negoziato scende a 100 Mbps o fallisce. Un cable analyzer professionale misura i parametri di trasmissione e rileva split pair. Si trovano comunque in vendita tester (odometri) che per uso non continuativo sono comunque efficaci e costano poche decine di euro.</p>
<p><strong>4. Split pair:</strong></p>
<p>Uno split pair si verifica quando i due fili di una coppia funzionale (es. pin 1 e 2, che trasportano TX+ e TX-) appartengono fisicamente a due coppie diverse del cavo. I fili dello stesso colore si confondono facilmente durante la crimpatura (es. Bianco/Verde con Verde finisce con Bianco/Blu con Verde). Il risultato: il test di continuità è corretto, ma l'accoppiamento differenziale e gravemente compromesso perchè i due fili della coppia logica non sono intrecciati insieme. Il crosstalk che ne risulta e molto elevato e degrada le prestazioni soprattutto a velocita Gigabit.</p></td>
</tr>
</tbody>
</table>

# 5. Fibra ottica

La fibra ottica trasmette dati sotto forma di impulsi luminosi in un nucleo di vetro o plastica ad altissima purezza. La luce si propaga all'interno del nucleo per riflessione interna totale: raggiungendo l'interfaccia tra il nucleo (indice di rifrazione più alto) e il rivestimento (cladding, indice piu basso) con un angolo superiore all'angolo critico, la luce viene completamente riflessa senza perdite all'interfaccia.

Vantaggi principali rispetto al rame:

- Larghezza di banda enormemente superiore (potenzialmente decine di Tbps per fibra).

- Attenuazione molto bassa: la fibra monomodale perde ~0.2 dB/km, contro ~22 dB/km del Cat 6a.

- Immunità completa alle interferenze elettromagnetiche (EMI) e alle scariche elettrostatiche (ESD).

- Nessun rischio di crosstalk tra fibre adiacenti.

- Sicurezza fisica: non emette campi elettromagnetici rilevabili; intercettare il segnale richiederebbe una giunzione fisica.

- Distanze: da centinaia di metri fino a migliaia di chilometri (con amplificatori EDFA).

## 5.1 Fibra multimodale (MMF)

Nella fibra multimodale il nucleo è più largo (tipicamente 50 o 62.5 micrometri di diametro) e permette la propagazione di più modi (percorsi) luminosi contemporaneamente. I diversi modi percorrono distanze leggermente diverse e arrivano al ricevitore in momenti diversi, causando una dispersione modale che limita la larghezza di banda su lunghe distanze.

Sorgente luminosa: LED (per distanze brevi) o VCSEL (Vertical-Cavity Surface-Emitting Laser) per prestazioni migliori.

|  |  |  |  |  |
|----|----|----|----|----|
| **Standard** | **Core** | **Larghezza di banda** | **Distanza max (10G)** | **Uso tipico** |
| OM1 | 62.5 um | 200 MHz\*km | 33 m | Legacy, obsoleto |
| OM2 | 50 um | 500 MHz\*km | 82 m | Legacy |
| OM3 | 50 um | 2000 MHz\*km | 300 m | Data center, 10G |
| OM4 | 50 um | 4700 MHz\*km | 400 m | Data center, 40/100G |
| OM5 | 50 um | 28000 MHz\*km | 150 m (100G) | WDM su multimodale, 400G |

## 5.2 Fibra monomodale (SMF)

Nella fibra monomodale il nucleo è molto piu stretto (tipicamente 8-10 micrometri) e permette la propagazione di un solo modo luminoso. Sorgente: laser diodo a emissione laterale. Assenza di dispersione modale: permette velocita elevatissime su grandissime distanze.

|  |  |  |
|----|----|----|
| **Standard** | **Applicazione** | **Distanza tipica** |
| OS1 | Ambienti interni (indoor) | Fino a 10 km a 10 Gbps |
| OS2 | Ambienti esterni (outdoor), lunghe distanze | Fino a 40-80 km a 10 Gbps; transoceanica con amplificatori EDFA |

**Nota:** *Le fibre OM3/OM4/OM5 usano VCSEL (laser a basso costo), non LED, per raggiungere le prestazioni richieste dagli standard 10G/40G/100G. I LED sono usati solo nelle fibre OM1/OM2 legacy. La fibra monomodale utilizza sempre laser (DFB o Fabry-Perot).*

## 5.3 Connettori in fibra ottica

|  |  |  |
|----|----|----|
| **Connettore** | **Caratteristiche** | **Uso tipico** |
| SC (Subscriber Connector) | Push-pull, form factor quadrato, duplex o simplex | Telecomunicazioni, reti enterprise, FTTH |
| LC (Lucent Connector) | Push-pull, form factor piccolo (SFP), molto diffuso | Data center, SFP/SFP+, switch enterprise |
| ST (Straight Tip) | Baionetta, circolare, ormai obsoleto | Installazioni legacy |
| FC (Ferrule Connector) | Avvitato, alta precisione | Strumenti di misura, ambienti vibranti |
| MPO/MTP | Multi-fibra (12 o 24 fibre in un connettore) | Data center ad alta densita, 40G/100G/400G |

**Espansione:** *Le connessioni ottiche in trasmissione full-duplex utilizzano normalmente due fibre separate (una per TX, una per RX). Tuttavia, le soluzioni BiDi (Bi-Directional) usano una sola fibra con due lunghezze d'onda diverse (WDM a due canali) per le due direzioni, riducendo il consumo di fibra. Usato in accesso GPON (fibra in casa) e in alcuni SFP BiDi per data center.*

## 5.4 Riflettometria OTDR

L'OTDR (Optical Time Domain Reflectometer) e lo strumento di misura fondamentale per la verifica e la manutenzione delle fibre ottiche. Funziona iniettando un impulso laser nella fibra e misurando la luce retrodiffusa (backscattered) nel tempo. Dalla velocita di propagazione della luce nella fibra (~2/3 c) e dai tempi di ritorno, l'OTDR calcola la distanza di ogni evento riflettente.

L'OTDR permette di:

- localizzare con precisione interruzioni, giunzioni, connettori e curve eccessive (con risoluzione centimetrica);

- misurare l'attenuazione della fibra in dB/km;

- misurare la perdita di inserzione di connettori e giunzioni;

- rilevare riflessioni anomale (ORL - Optical Return Loss);

- certificare una fibra secondo gli standard richiesti (es. ISO/IEC 14763-3).

# 6. Reti wireless e standard IEEE 802.11

## 6.1 Bande di frequenza Wi-Fi

Le reti Wi-Fi operano in bande ISM (Industrial, Scientific and Medical) e bande con licenza specifica. Le bande principali sono:

|  |  |  |  |  |
|----|----|----|----|----|
| **Banda** | **Frequenze** | **Canali (esempio EU)** | **Pro** | **Contro** |
| 2.4 GHz | 2.400-2.4835 GHz | 13 (solo 3 non sovrapposti: 1, 6, 13) | Penetrazione pareti, copertura ampia | Congestionata, interferenze (microonde, Bluetooth, ZigBee) |
| 5 GHz | 5.150-5.850 GHz | Da 25 a 45 canali non sovrapposti (dipende dalla normativa) | Meno congestionata, canali piu ampi (80/160 MHz) | Minore penetrazione pareti, distanza ridotta |
| 6 GHz | 5.925-7.125 GHz | Fino a 59 canali da 20 MHz (Wi-Fi 6E) | Bassa congestione, canali ampi, latenza ridotta | Nuova tecnologia, copertura limitata |

**Nota: Wifi ed elettrodomestici.** *I forni a microonde ISM operano a 2450 MHz e possono interferire significativamente con il Wi-Fi 2.4 GHz, specialmente se il forno ha schermatura difettosa. Anche i telefoni DECT (2.4 GHz), i baby monitor e i dispositivi Bluetooth (2.4 GHz, FHSS) sono sorgenti di interferenza nella stessa banda.*

## 6.2 Standard IEEE 802.11: evoluzione

|  |  |  |  |  |  |
|----|----|----|----|----|----|
| **Standard** | **Nome** | **Banda** | **Velocità max teorica** | **Anno** | **Modulazione** |
| 802.11b | Wi-Fi 1 | 2.4 GHz | 11 Mbps | 1999 | DSSS |
| 802.11a | Wi-Fi 2 | 5 GHz | 54 Mbps | 1999 | OFDM |
| 802.11g | Wi-Fi 3 | 2.4 GHz | 54 Mbps | 2003 | OFDM |
| 802.11n | Wi-Fi 4 | 2.4 / 5 GHz | 600 Mbps (4x4 MIMO) | 2009 | OFDM + MIMO |
| 802.11ac | Wi-Fi 5 | 5 GHz | 3.5 Gbps (Wave 2) | 2013/2016 | 256-QAM OFDM, MU-MIMO |
| 802.11ax | Wi-Fi 6 | 2.4 / 5 GHz | 9.6 Gbps | 2019 | 1024-QAM OFDMA, MU-MIMO |
| 802.11ax (6E) | Wi-Fi 6E | 2.4 / 5 / 6 GHz | 9.6 Gbps | 2021 | Come Wi-Fi 6 + banda 6 GHz |
| 802.11be | Wi-Fi 7 | 2.4 / 5 / 6 GHz | 46 Gbps (teorici) | 2024 | 4096-QAM, MLO, 320 MHz |

**Nota:** *IEEE 802.11be (Wi-Fi 7) e stato ratificato nel 2024 e i dispositivi commerciali sono già disponibili. I valori di velocità indicati sono sempre teorici (peak PHY rate); il throughput reale e tipicamente il 40-60% del valore nominale.*

## 6.3 Software Defined Radio (SDR)

Una Software Defined Radio e un sistema radio in cui le componenti tradizionalmente implementate in circuiti hardware analogici dedicati (filtri, mixer, demodulatori, decoder) vengono invece eseguite via software su un processore general-purpose o FPGA. L'hardware si riduce al minimo indispensabile: antenna, amplificatore a basso rumore (LNA), convertitore analogico-digitale (ADC) ad alta velocita.

Il segnale radio analogico viene campionato direttamente dall'ADC (o con un down-conversion intermedia a frequenza intermedia) e tutti i processi di filtraggio, demodulazione e decodifica avvengono in software (tipicamente in Python con GNU Radio, o in C con liquidSDR).

Applicazioni (legittime) delle SDR:

- Ricezione ADS-B (Automatic Dependent Surveillance-Broadcast): gli aeromobili trasmettono la propria posizione, quota e identificativo in chiaro a 1090 MHz. Con un RTL-SDR da ~30 euro e il software dump1090, e possibile ricevere e visualizzare tutti i voli in un raggio di ~300 km.

- Ricezione meteo da satelliti NOAA/METEOR: immagini meteorologiche in tempo reale da satelliti in orbita LEO.

- Analisi dei canali di controllo GSM (BCCH): identificazione delle celle, frequenze, LAC/Cell ID. Le comunicazioni vocali GSM (A5/1) non sono facilmente decifrabili senza le chiavi.

- Analisi di segnali ISM: sensori 433 MHz, sistemi TPMS (pressione pneumatici), contatori smart.

- Ricezione GPS/GNSS: con SDR ad alta frequenza, studio dei segnali di navigazione.

- Sperimentazione didattica e ricerca in telecomunicazioni.

**Nota legale:** *La ricezione passiva di segnali radio è generalmente legale nella maggior parte dei paesi. La trasmissione con SDR richiede una licenza radioamatoriale o altra autorizzazione. L'intercettazione di comunicazioni private è illegale indipendentemente dallo strumento usato. In Italia, l'intercettazione non autorizzata di comunicazioni è reato ai sensi dell'art. 617 c.p.*

# 7. Il livello Data Link (Livello 2 OSI)

Il livello Data Link (Layer 2) fornisce il trasferimento affidabile dei dati tra due nodi direttamente connessi sullo stesso mezzo fisico. Si occupa di organizzare i bit del livello fisico in unità logiche chiamate frame, di indirizzare i nodi sullo stesso segmento di rete e di rilevare (e in alcuni casi correggere) gli errori di trasmissione.

Le funzioni principali sono:

- Framing: incapsulamento dei dati del livello superiore in frame con header e trailer.

- Indirizzamento fisico (MAC address): identificazione univoca dei nodi sullo stesso segmento.

- Controllo di accesso al mezzo (MAC - Media Access Control): gestione di chi può trasmettere e quando.

- Rilevamento degli errori (CRC): verifica dell'integrità del frame ricevuto.

- Controllo del flusso: prevenzione del flooding del ricevitore (in alcuni protocolli).

Il livello Data Link e suddiviso in due sottolivelli:

- LLC (Logical Link Control, IEEE 802.2): interfaccia verso il livello Network (IP). Multiplexing dei protocolli di livello superiore (Ethernet, Wi-Fi, PPP possono trasportare IPv4, IPv6, ARP ecc.).

- MAC (Media Access Control): interazione con il livello fisico, controllo dell'accesso al mezzo, indirizzamento MAC.

## 7.1 MAC Address: struttura e proprietà

Il MAC address (Media Access Control address) è un identificatore a 48 bit (6 byte) assegnato a ogni interfaccia di rete. E’ rappresentato in notazione esadecimale con separatori (es. AA:BB:CC:DD:EE:FF oppure AA-BB-CC-DD-EE-FF o AABB.CCDD.EEFF nel formato Cisco).

|  |  |  |
|----|----|----|
| **Byte** | **Campo** | **Descrizione** |
| Byte 1-3 (24 bit) | OUI (Organizationally Unique Identifier) | Identifica il produttore/vendor della scheda di rete. Assegnato dall'IEEE Registration Authority. Il bit 0 del primo byte indica unicast (0) o multicast (1). Il bit 1 del primo byte indica indirizzo universalmente amministrato (0) o localmente amministrato (1). |
| Byte 4-6 (24 bit) | NIC-specific (Extension Identifier) | Assegnato dal produttore per identificare univocamente ogni scheda prodotta. Teoricamente univoco globalmente (2^48 ~= 281 trilioni di combinazioni). |

Tipi di MAC address:

- Unicast: il bit 0 del primo byte e 0. Indirizza un singolo dispositivo. Es. 00:1A:2B:3C:4D:5E.

- Multicast: il bit 0 del primo byte e 1. Indirizza un gruppo di dispositivi. Es. 01:00:5E:xx:xx:xx per multicast IPv4 (IANA).

- Broadcast: tutti i bit a 1 (FF:FF:FF:FF:FF:FF). Indirizza tutti i dispositivi del segmento.

- Locally Administered (LAA): il bit 1 del primo byte e 1. Indica che il MAC e stato assegnato localmente (non dal produttore). Usato nella virtualizzazione (VM), nel MAC randomization (privacy Wi-Fi, iOS/Android), e nelle interfacce VLAN.

**Precisazione sul MAC randomization:** *I moderni sistemi operativi (iOS 14+, Android 10+, Windows 10+) usano MAC address randomizzati nelle scansioni Wi-Fi e, in alcune implementazioni, anche nelle connessioni. Questo protegge la privacy dell'utente impedendo il tracking basato su MAC, ma può creare problemi con sistemi di autenticazione o di inventory basati su MAC address.*

## 7.2 Struttura di un frame Ethernet (IEEE 802.3)

|  |  |  |
|----|----|----|
| **Campo** | **Dimensione** | **Descrizione** |
| Preambolo | 7 byte | Sequenza alternata 10101010 per 7 byte. Permette la sincronizzazione del clock del ricevitore. |
| SFD (Start Frame Delimiter) | 1 byte (10101011) | Indica l'inizio del frame vero e proprio. |
| MAC Destinazione | 6 byte | Indirizzo MAC del destinatario (unicast, multicast o broadcast). |
| MAC Sorgente | 6 byte | Indirizzo MAC del mittente (sempre unicast). |
| EtherType / Lunghezza | 2 byte | Se \>= 0x0600 (1536): EtherType (protocollo payload). Se \< 0x0600: lunghezza del payload (Ethernet II vs IEEE 802.3 originale). Valori comuni: 0x0800 = IPv4, 0x86DD = IPv6, 0x0806 = ARP, 0x8100 = VLAN tag (802.1Q). |
| Payload (dati) | 46-1500 byte | Dati del livello superiore (IP, ARP, ecc.). Il payload minimo di 46 byte e necessario per il rilevamento delle collisioni CSMA/CD (padding se necessario). |
| FCS (Frame Check Sequence) | 4 byte | CRC-32 calcolato su MAC dest + MAC src + EtherType + payload. Il ricevitore ricalcola il CRC e confronta: se diverso, il frame viene scartato silenziosamente. |

**Nota:** *Il preambolo e l'SFD non sono visibili in Wireshark in quanto vengono rimossi dalla scheda di rete prima di consegnare il frame al driver. La dimensione minima di un frame Ethernet e 64 byte (escluso preambolo e SFD): 6+6+2+46+4 = 64. La dimensione massima e 1518 byte (1500 byte di payload + 18 byte di header). I frame jumbo (Jumbo Frame) possono avere payload fino a 9000 byte, ma richiedono supporto esplicito da parte di tutti i dispositivi sul percorso.*

## 7.3 CSMA/CD e CSMA/CA

### CSMA/CD - Ethernet cablata

CSMA/CD (Carrier Sense Multiple Access with Collision Detection) era il meccanismo di accesso al mezzo delle reti Ethernet condivise (topologia a bus, hub):

13. Carrier Sense: prima di trasmettere, la scheda ascolta il mezzo. Se rileva un segnale (carrier), attende.

14. Multiple Access: più dispositivi condividono lo stesso mezzo.

15. Collision Detection: durante la trasmissione, la scheda continua ad ascoltare. Se rileva una collisione (il segnale ricevuto differisce da quello trasmesso), interrompe la trasmissione e invia un jam signal (32 bit) per avvisare tutti.

16. Backoff: dopo la collisione, ogni scheda attende un tempo casuale calcolato con l'algoritmo Truncated Binary Exponential Backoff: al k-esimo tentativo, il tempo di attesa viene scelto casualmente tra 0 e 2^min(k,10) - 1 slot di 51.2 microsecondi. Dopo 16 tentativi falliti, il frame viene scartato e viene segnalato un errore.

Con gli switch moderni, CSMA/CD e praticamente obsoleto: ogni porta dello switch e un dominio di collisione separato con full-duplex, eliminando la possibilità di collisioni. CSMA/CD rimane rilevante solo per comprendere la storia e il dimensionamento del round-trip time.

### CSMA/CA - Wireless (IEEE 802.11)

Il wireless non può usare CSMA/CD perchè un trasmettitore non può ascoltare le collisioni mentre trasmette (il proprio segnale sovrasta qualsiasi altro). Si usa CSMA/CA (Collision Avoidance):

17. Carrier Sense: il dispositivo ascolta il canale. Se occupato, attende.

18. Deferral: se libero, attende un ulteriore periodo fisso (DIFS - Distributed Interframe Space).

19. Backoff casuale: sceglie un numero casuale di slot (contention window) e decrementa il contatore mentre il canale e libero.

20. Trasmissione: quando il contatore raggiunge zero, trasmette.

21. ACK: il ricevitore, dopo un breve intervallo (SIFS - Short IFS), invia un ACK. Se l'ACK non arriva entro il timeout, si assume una collisione e si ritrasmette con backoff aumentato.

Il meccanismo opzionale RTS/CTS (Request to Send / Clear to Send) risolve il problema del nodo nascosto: il mittente invia un breve RTS; il ricevitore risponde con CTS (che include la durata della trasmissione). Tutti i dispositivi che ricevono il CTS si astengono dal trasmettere per quella durata, anche se non hanno visto l'RTS originale.

**Problema del nodo nascosto:** *Due dispositivi A e C non si vedono tra loro (per via di ostacoli fisici o distanza) ma entrambi comunicano con il punto di accesso B. A e C possono trasmettere simultaneamente verso B causando una collisione che ne B può distinguere chiaramente. RTS/CTS risolve questo problema: prima che A trasmetta, invia un RTS a B; B risponde con CTS; C sente il CTS e sa che deve aspettare.*

## 7.4 Half duplex e full duplex

La modalità duplex descrive la capacita di trasmissione contemporanea nelle due direzioni:

- Half duplex: il dispositivo può trasmettere oppure ricevere in un dato momento, ma non entrambe le cose simultaneamente. Necessario in reti con collisioni (CSMA/CD o CSMA/CA). Throughput effettivo ridotto per il contention del mezzo.

- Full duplex: trasmissione e ricezione avvengono contemporaneamente su percorsi fisici separati (coppie diverse nel cavo UTP, o due fibre ottiche). Elimina le collisioni. Raddoppia la banda aggregata teorica (es. 1 Gbps Full Duplex = 1 Gbps TX + 1 Gbps RX simultanei). Richiesto da tutti gli switch Ethernet moderni.

**Nota — Auto-negotiation:** *IEEE 802.3u definisce il meccanismo di Auto-Negotiation (AN) con cui due dispositivi concordano automaticamente velocità e modalità duplex. AN usa segnali Fast Link Pulse (FLP) trasmessi sulle coppie di dati. In caso di disallineamento (un lato in AN, l'altro fisso), il lato AN imposta una velocità ma potrebbe usare half duplex, causando il noto problema 'duplex mismatch': molte collisioni, throughput degradato, errori continui. Soluzione: configurare entrambi i lati allo stesso modo (entrambi AN, o entrambi fissi alla stessa velocita e duplex).*

# 8. Analisi del traffico con Wireshark

Wireshark è il piu diffuso analizzatore di protocollo di rete (packet analyzer / network protocol analyzer) open source. Intercetta i pacchetti direttamente dal buffer della scheda di rete (tramite la libreria libpcap su Linux/macOS o WinPcap/Npcap su Windows) e li decodifica in modo leggibile per l'analista.

Funzionalità principali:

- Cattura live: intercetta il traffico su qualsiasi interfaccia di rete disponibile (Ethernet, Wi-Fi, loopback, VLAN, interfacce virtuali).

- Analisi di file pcap: importa e analizza file di cattura esistenti (formato .pcap o .pcapng).

- Dissectors: decodifica automatica di centinaia di protocolli (Ethernet, IP, TCP, UDP, HTTP, DNS, TLS, DHCP, SMTP, ecc.).

- Filtri di cattura (BPF): filtrano il traffico prima della cattura per ridurre il volume (es. 'host 192.168.1.1 and port 80').

- Filtri di display: filtrano i pacchetti gia catturati per l'analisi (es. 'tcp.port == 443 and http').

- Ricostruzione dei flussi: segue le sessioni TCP o UDP e ricostruisce il payload completo.

- Analisi statistica: statistiche su conversazioni, endpoint, distribuzione dei protocolli, I/O graphs.

Nell'analisi di un frame Ethernet con Wireshark e possibile identificare:

- MAC address sorgente e destinazione (con risoluzione OUI automatica: Wireshark mostra il vendor).

- EtherType: il protocollo incapsulato (IPv4, IPv6, ARP, VLAN).

- Payload: i dati del livello superiore, decodificati ricorsivamente da Wireshark.

- FCS: normalmente non mostrato da Wireshark (rimosso dalla NIC prima della consegna al driver).

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><strong>Esercizio 2 - Analisi di un frame Ethernet con Wireshark</strong></td>
</tr>
<tr>
<td><p>Obiettivo: analizzare un frame Ethernet reale e interpretarne la struttura.</p>
<p>Attivita:</p>
<ol start="22" type="1">
<li><p>Avviare Wireshark. Selezionare l'interfaccia di rete attiva (Ethernet o Wi-Fi).</p></li>
<li><p>Applicare il filtro di cattura 'arp' per catturare solo i pacchetti ARP (molto frequenti e facili da analizzare).</p></li>
<li><p>Attendere qualche secondo e fermare la cattura.</p></li>
<li><p>Selezionare un frame ARP Request. Espandere il layer 'Ethernet II' nel pannello di dissection.</p></li>
<li><p>Rispondere alle domande seguenti basandosi sull'analisi del frame catturato.</p></li>
</ol>
<p>Domande:</p>
<ul>
<li><p>Qual e il MAC address sorgente e il MAC address destinazione del frame ARP Request? Perche la destinazione ha quel valore?</p></li>
<li><p>Qual e il valore del campo EtherType? Cosa indica?</p></li>
<li><p>Aprire il pannello 'Address Resolution Protocol'. Chi e il 'Sender MAC address', il 'Sender IP address', il 'Target MAC address' e il 'Target IP address' in un ARP Request?</p></li>
<li><p>Come si identifica il produttore della scheda di rete dal MAC address sorgente?</p></li>
<li><p>Applicare il filtro di display 'eth.addr == FF:FF:FF:FF:FF:FF'. Cosa mostrano i risultati?</p></li>
</ul></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><strong>Soluzione</strong></td>
</tr>
<tr>
<td><p><strong>1. MAC sorgente e destinazione in ARP Request:</strong></p>
<p>Il MAC sorgente e il MAC address della scheda di rete del dispositivo che genera la richiesta ARP (es. la propria macchina). Il MAC destinazione e FF:FF:FF:FF:FF:FF (broadcast Ethernet) perche il dispositivo mittente non conosce ancora il MAC del destinatario (quello e esattamente lo scopo dell'ARP: scoprire il MAC a partire dall'IP). L'ARP Request deve raggiungere tutti i dispositivi del segmento LAN, quindi viene inviata in broadcast.</p>
<p><strong>2. EtherType dell'ARP:</strong></p>
<p>Il valore del campo EtherType per il protocollo ARP e 0x0806 (2054 in decimale). Questo campo indica al livello Data Link quale protocollo e incapsulato nel payload del frame. Il ricevitore usa questo valore per demultiplexare il frame verso il modulo protocollo corretto (nel caso di ARP, verso il modulo ARP del sistema operativo, non verso IP).</p>
<p><strong>3. Campi dell'ARP Request:</strong></p>
<ul>
<li><p>Sender MAC address: MAC del richiedente (es. 00:1A:2B:3C:4D:5E) - il dispositivo che invia la richiesta.</p></li>
<li><p>Sender IP address: IP del richiedente (es. 192.168.1.10) - l'IP di chi manda la domanda.</p></li>
<li><p>Target MAC address: 00:00:00:00:00:00 - sconosciuto, e quello che si vuole scoprire.</p></li>
<li><p>Target IP address: IP del destinatario cercato (es. 192.168.1.1) - il gateway o l'host di cui si vuole il MAC.</p></li>
</ul>
<p>Il dispositivo che riconosce il proprio IP nel campo Target risponde con un ARP Reply contenente il proprio MAC address nel campo Target MAC address.</p>
<p><strong>4. Identificazione del produttore via OUI:</strong></p>
<p>Wireshark risolve automaticamente i primi 3 byte del MAC address consultando il database OUI dell'IEEE incorporato (aggiornabile). Ad esempio, 00:1A:2B corrisponde a un vendor specifico. Se Wireshark non risolve il vendor, e possibile cercare manualmente su https://regauth.standards.ieee.org/standards-ra-web/pub/view.html#registries. In alternativa, il comando 'nmap --script broadcast-arp' mostra i vendor risolti. Questa tecnica è usata nell'inventory di rete e nel rilevamento di dispositivi non autorizzati.</p>
<p><strong>5. Filtro 'eth.addr == FF:FF:FF:FF:FF:FF':</strong></p>
<p>Il filtro mostra tutti i frame Ethernet con MAC destinazione broadcast (FF:FF:FF:FF:FF:FF). In una rete attiva si vedranno principalmente: ARP Request (il tipo piu comune di broadcast in una LAN); pacchetti DHCP Discover/Request (il client non ha ancora un IP e usa broadcast); alcuni pacchetti di protocolli legacy (NetBIOS, mDNS, SSDP possono usare broadcast o multicast). Analizzare il traffico broadcast e utile per il troubleshooting della rete (broadcast storm, ARP poisoning, scoperta di nuovi dispositivi).</p></td>
</tr>
</tbody>
</table>

# 9. Topologie di rete al livello fisico e Data Link

La topologia di rete descrive la struttura fisica o logica delle connessioni tra i nodi. E’ importante distinguere la topologia fisica (disposizione reale dei cavi e dei dispositivi) dalla topologia logica (percorso dei dati).

|  |  |  |  |  |
|----|----|----|----|----|
| **Topologia** | **Descrizione** | **Vantaggi** | **Svantaggi** | **Uso tipico** |
| Stella | Tutti i dispositivi connessi a un nodo centrale (switch). | Isolamento guasti, gestione semplice, full-duplex. | Single point of failure sul nodo centrale. | LAN Ethernet moderna - standard de facto |
| Bus | Tutti i dispositivi sullo stesso mezzo condiviso. | Semplice, economico. | Collisioni, un guasto abbatte tutta la rete, non scala. | Ethernet coassiale storica (10Base2/5), obsoleta |
| Anello | Ogni nodo connesso ai due adiacenti in un anello chiuso. | Traffico ordinato (token), nessuna collisione. | Un guasto interrompe l'anello (senza protezione). | Token Ring (storico), FDDI, SDH/SONET (con APS) |
| Maglia (mesh) | Ogni nodo connesso a uno o piu altri nodi. | Alta ridondanza e resilienza. | Costo elevato, complessita di gestione. | Internet backbone, reti WAN, Wi-Fi mesh |
| Punto-punto | Collegamento diretto tra due soli nodi. | Semplice, banda dedicata. | Non scala. | WAN dedicati, VPN, ponti radio, uplink fibra |
| Albero (gerarchica) | Stella di stelle su piu livelli. | Scalabile, gerarchica. | Dipendenza dai nodi di distribuzione. | Reti aziendali (core/distribution/access) |

# 10. Standardizzazione delle comunicazioni di rete

Le comunicazioni di rete sono standardizzate da organismi internazionali che garantiscono l'interoperabilita tra prodotti di vendor diversi. I principali enti di standardizzazione sono:

|  |  |  |
|----|----|----|
| **Ente** | **Nome completo** | **Ambito principale** |
| IEEE | Institute of Electrical and Electronics Engineers | Standard Ethernet (802.3), Wi-Fi (802.11), VLAN (802.1Q), sicurezza wireless (802.1X). L'IEEE Standards Association gestisce il registro OUI. |
| IETF | Internet Engineering Task Force | Protocolli Internet (RFC): IP, TCP, UDP, HTTP, DNS, SMTP, TLS, BGP. Standard de facto per i protocolli applicativi. |
| ISO | International Organization for Standardization | Modello OSI (ISO 7498), standard di cablaggio (ISO/IEC 11801), standard di qualita e sicurezza. |
| ITU-T | International Telecommunication Union - Telecom. Standardization Sector | Telecomunicazioni: G.711 (codec voce), G.652 (fibra SMF), H.264 (video), DSL, GPON. |
| TIA/EIA | Telecommunications Industry Association / Electronic Industries Alliance | Standard di cablaggio strutturato TIA-568 (T568A/B, categorie cavi), connettori, installazione. |
| ANSI | American National Standards Institute | Standard USA, spesso in collaborazione con ISO e IEC. |
| ETSI | European Telecommunications Standards Institute | Standard europei: GSM, LTE, 5G, DVB, DECT. |

# Bibliografia

Le opere sono organizzate per argomento e citate in formato APA 7a edizione.

## Livello fisico, trasmissione e segnali

Haykin, S. (2014). *Communication systems* (4th ed.). Wiley.

Proakis, J. G., & Salehi, M. (2008). *Digital communications* (5th ed.). McGraw-Hill.

Couch, L. W. (2013). *Digital and analog communication systems* (8th ed.). Pearson.

Shannon, C. E. (1948). A mathematical theory of communication. *Bell System Technical Journal, 27*(3), 379-423. https://doi.org/10.1002/j.1538-7305.1948.tb01338.x

Nyquist, H. (1928). Certain topics in telegraph transmission theory. *Transactions of the American Institute of Electrical Engineers, 47*(2), 617-644. https://doi.org/10.1109/T-AIEE.1928.5055024

## Cablaggio strutturato e connettori

TIA. (2020). *ANSI/TIA-568.2-D: Balanced twisted-pair telecommunications cabling and components standard*. Telecommunications Industry Association.

ISO/IEC. (2017). *ISO/IEC 11801-1:2017: Information technology - Generic cabling for customer premises - Part 1: General requirements*. ISO.

IEEE. (2022). *IEEE 802.3-2022: IEEE standard for Ethernet*. IEEE. https://doi.org/10.1109/IEEESTD.2022.9844436

## Fibra ottica

Saleh, B. E. A., & Teich, M. C. (2019). *Fundamentals of photonics* (3rd ed.). Wiley.

Agrawal, G. P. (2019). *Fiber-optic communication systems* (6th ed.). Wiley.

ITU-T. (2016). *Recommendation G.652: Characteristics of a single-mode optical fibre and cable*. ITU.

## Reti wireless e IEEE 802.11

IEEE. (2021). *IEEE 802.11-2020: IEEE standard for Information Technology - Telecommunications and information exchange between systems - Local and metropolitan area networks - Specific requirements - Part 11: Wireless LAN Medium Access Control (MAC) and Physical Layer (PHY) specifications*. IEEE. https://doi.org/10.1109/IEEESTD.2021.9363693

Gast, M. (2013). *802.11ac: A survival guide*. O'Reilly Media.

Bellardo, J., & Savage, S. (2003). 802.11 denial-of-service attacks: Real vulnerabilities and practical solutions. *Proceedings of the 12th USENIX Security Symposium*, 15-28.

## Software Defined Radio

Blossom, E. (2004). GNU Radio: Tools for exploring the radio frequency spectrum. *Linux Journal, 2004*(122), articolo 4. https://www.linuxjournal.com/article/7319

Ossmann, M. (2012). *Software defined radio with HackRF* \[Video serie\]. Great Scott Gadgets. https://greatscottgadgets.com/sdr/

## Livello Data Link, Ethernet e MAC

Metcalfe, R. M., & Boggs, D. R. (1976). Ethernet: Distributed packet switching for local computer networks. *Communications of the ACM, 19*(7), 395-404. https://doi.org/10.1145/360248.360253

Spurgeon, C. E., & Zimmerman, J. (2014). *Ethernet: The definitive guide* (2nd ed.). O'Reilly Media.

IEEE. (2022). *IEEE 802.1Q-2022: IEEE standard for local and metropolitan area networks - Bridges and bridged networks*. IEEE.

## Reti informatiche - testi di riferimento

Tanenbaum, A. S., & Wetherall, D. J. (2011). *Computer networks* (5th ed.). Prentice Hall.

Kurose, J. F., & Ross, K. W. (2021). *Computer networking: A top-down approach* (8th ed.). Pearson.

Forouzan, B. A. (2013). *Data communications and networking* (5th ed.). McGraw-Hill.

## Wireshark e analisi di rete

Sanders, C. (2017). *Practical packet analysis: Using Wireshark to solve real-world network problems* (3rd ed.). No Starch Press.

Wireshark Foundation. (2024). *Wireshark user's guide*. https://www.wireshark.org/docs/wsug_html_chunked/

**Nota:** *Gli standard IEEE e ISO citati sono documenti normativi a pagamento; i titoli e i DOI forniti permettono di localizzarli nei cataloghi ufficiali. Le RFC dell'IETF sono accessibili gratuitamente su https://www.rfc-editor.org.*
