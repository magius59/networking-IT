**Lezione 2**

Posta Elettronica, SMTP e Architettura dei Protocolli di Rete

# 1. Architettura generale della posta elettronica

La posta elettronica e uno dei servizi fondamentali di Internet, in uso commerciale dal 1971 e standardizzato progressivamente attraverso RFC. Il suo funzionamento introduce concetti centrali del networking e della sicurezza informatica: autenticazione, trasporto dati, DNS, protocolli applicativi e protezione contro il phishing.

Un sistema di posta elettronica e composto da tre componenti principali:

- User Agent (UA): il software client usato dall'utente per comporre, inviare e leggere messaggi (es. Outlook, Thunderbird, Apple Mail, interfacce webmail).

- Mail Transfer Agent (MTA): il server che riceve la posta dal client mittente tramite SMTP, la instrada e la consegna al server destinatario. Esempi: Postfix, Sendmail, Microsoft Exchange, Exim.

- Mail Delivery Agent (MDA): il componente del server che deposita il messaggio nella mailbox del destinatario. Esempi: Dovecot, Procmail.

Il flusso completo della trasmissione di una e-mail è il seguente:

1.  L'utente compone il messaggio nel client (UA).

2.  Il client si collega al Mail Submission Agent del proprio provider tramite SMTP autenticato (porta 587 o 465).

3.  L'MSA/MTA del mittente consulta il DNS per trovare il record MX del dominio destinatario.

4.  Il MTA mittente apre una connessione SMTP verso il MTA destinatario (porta 25).

5.  Il MDA consegna il messaggio nella mailbox del destinatario.

6.  Il destinatario recupera il messaggio tramite POP3 o IMAP.

# 2. SMTP - Simple Mail Transfer Protocol

## 2.1 Porte e versioni

SMTP (RFC 5321, che aggiorna RFC 2821) e il protocollo di trasferimento delle e-mail tra server. Opera su TCP e utilizza tre porte con ruoli distinti:

| **Porta** | **Nome** | **Uso** | **Cifratura** |
|----|----|----|----|
| 25 | SMTP | Relay tra MTA (server-to-server). Bloccata da molti ISP per i client. | Nessuna / STARTTLS |
| 465 | SMTPS | Invio da client con TLS implicito (wrapped TLS). Deprecato poi reintrodotto (RFC 8314). | TLS implicito |
| 587 | Submission | Invio da client autenticato (MSA). Porta raccomandata per i client. | STARTTLS obbligatorio |

## Premessa: chi comunica con chi nella posta elettronica?

Nel modello classico della posta elettronica esistono **due ruoli principali:**

- **Mail User Agent (MUA)** – il client di posta elettronica, ad esempio Outlook, Thunderbird o un’interfaccia webmail.

- **Mail Transfer Agent (MTA)** – il server di posta elettronica, ad esempio Postfix, Exim o Sendmail.

Nel funzionamento della posta elettronica esistono due scenari distinti:

- Invio dal client verso il proprio server di posta (MUA → MTA)

- Trasferimento tra server di posta (MTA → MTA remoto)

Le porte TCP 25, 587 e 465 corrispondono a differenti modalità operative del protocollo SMTP.

------------------------------------------------------------------------

## 1. Porta 25 – SMTP tradizionale (relay MTA → MTA)

*La porta 25 è utilizzata per il relay SMTP tra server di posta e non dovrebbe essere usata dai client.*

La porta TCP 25 nasce come porta standard del protocollo SMTP ed è utilizzata principalmente per la comunicazione:

- tra server SMTP

- tra MTA differenti

Il suo utilizzo corretto riguarda quindi il relay della posta verso il dominio destinatario.

## Utilizzo storico

In passato anche i client di posta utilizzavano la porta 25 per inviare e-mail. Oggi questa pratica è generalmente sconsigliata e spesso bloccata dai provider Internet.

La ragione principale è legata alla sicurezza:

- malware

- botnet

- spammer

utilizzano frequentemente la porta 25 per inviare spam direttamente ai server remoti senza autenticazione.

## Caratteristiche tecniche

La porta 25:

- non richiede autenticazione per definizione storica

- avvia la connessione inizialmente in chiaro

- può utilizzare STARTTLS per attivare TLS successivamente

Molti ISP bloccano le connessioni in uscita verso la porta 25, consentendole solo verso i propri mail server.

**Nota\**

La porta 25 può essere vista come *«la strada tra uffici postali»*

e non come *«la strada tra utente e ufficio postale».*

## 2. Porta 587 – Message Submission autenticata

*La porta 587 è la porta standard raccomandata per l’invio delle e-mail da parte dei client.*

La porta TCP 587 è definita per il servizio di:

message submission

ed è regolata principalmente dalla RFC 6409

Ruolo operativo

In questo caso:

il client di posta (MUA) invia il messaggio al proprio server SMTP

Non si tratta quindi di relay tra server, ma di autenticazione dell’utente presso il server di submission.

Caratteristiche principali

Sulla porta 587:

- l’autenticazione è obbligatoria

- vengono usati meccanismi SASL (Simple authentication and security layer)

- è normalmente supportato STARTTLS

Il server SMTP richiede tipicamente:

- username

- password

prima di consentire l’invio della posta.

## Nota 

## SASL è uno strato che aggiunge autenticazione (e opzionalmente sicurezza) ai protocolli di rete. In sostanza, SASL è il meccanismo che permette a SMTP di autenticare l’utente tramite metodi standard (LOGIN, PLAIN, ecc.), rendendo obbligatorio dimostrare la propria identità prima di inviare email.

\
STARTTLS
--------

La cifratura avviene tramite STARTTLS

In questo modello:

- la connessione inizia in chiaro

- il client invia il comando STARTTLS

- la sessione viene convertita in TLS

Questo approccio viene definito:

- TLS esplicito

- opportunistic TLS

## 

## 

## 3. Porta 465 – SMTPS con TLS implicito

*La porta 465 utilizza TLS implicito e rappresenta una modalità moderna e sicura di submission SMTP.*

La storia della porta 465 è complessa.

**Evoluzione storica**

Negli anni ’90 la porta 465 venne utilizzata per:

SMTP over SSL

Successivamente fu deprecata e rimossa dalle assegnazioni ufficiali IANA.

Nel 2018 la RFC 8314 ha nuovamente raccomandato l’utilizzo della porta 465 per la submission SMTP sicura.

**TLS implicito vs TLS esplicito**

Porta 587 – TLS esplicito (STARTTLS)

Con STARTTLS:

- la connessione parte in chiaro

- successivamente viene richiesta la cifratura

Questo approccio mantiene compatibilità con sistemi legacy ma può essere vulnerabile ad attacchi di downgrade.

Porta 465 – TLS implicito

Con la porta 465:

- la connessione TLS viene stabilita immediatamente

- non esiste una fase iniziale in chiaro

Questo approccio riduce il rischio di:

- STARTTLS stripping

- downgrade attack

(vale a dire, non possiamo aggirare le richieste di connessione TLS).

## Confronto tra le tre porte

| Porta | Utilizzo principale | TLS | Autenticazione | Uso tipico |
|----|----|----|----|----|
| 25 | Relay MTA → MTA | STARTTLS opzionale | Generalmente no | Server di posta |
| 587 | Submission client → server | STARTTLS | Obbligatoria | Client moderni |
| 465 | Submission client → server | TLS implicito | Obbligatoria | Client moderni sicuri |

------------------------------------------------------------------------

## Esempio pratico

**Caso 1 – Client di posta**

Un utente che utilizza Thunderbird o Outlook normalmente configurerà:

porta 587 oppure porta 465 per inviare le e-mail.

L’utilizzo della porta 25 è generalmente scoraggiato.

**Caso 2 – Comunicazione tra server**

Quando il server SMTP di Gmail deve consegnare un messaggio al server SMTP di un altro provider:

viene utilizzata la porta 25 per il relay SMTP tra MTA.

## Sicurezza e attacchi STARTTLS

Uno dei problemi storici della porta 587 riguarda gli attacchi:

STARTTLS stripping

In questo scenario un attaccante man-in-the-middle modifica il traffico SMTP eliminando il comando: STARTTLS

In questo modo client e server continuano a comunicare in chiaro senza cifratura.

Con TLS implicito sulla porta 465 questo rischio è significativamente ridotto, poiché la sessione TLS viene stabilita immediatamente.

## Considerazioni moderne

Nelle infrastrutture contemporanee:

porta 25 → relay tra server

porta 587 → submission standard autenticata

porta 465 → submission con TLS implicito

Molti provider supportano sia 587 sia 465.

## Cos’è STARTTLS

STARTTLS è un’estensione di protocollo utilizzata per aggiungere cifratura TLS (Transport Layer Security) a una connessione inizialmente aperta in chiaro. Viene utilizzata principalmente nei protocolli di posta elettronica:

- SMTP

- IMAP

- POP3

L’idea fondamentale è che la comunicazione inizi come connessione normale non cifrata e venga successivamente “aggiornata” a una connessione TLS sicura tramite il comando STARTTLS.

## Funzionamento di STARTTLS

Il processo tipico è il seguente:

1.  Il client apre una connessione TCP standard.

2.  Client e server comunicano inizialmente in chiaro.

3.  Il server dichiara il supporto a STARTTLS.

4.  Il client invia il comando STARTTLS.

5.  Inizia l’handshake TLS.

6.  La connessione prosegue cifrata.

## Esempio SMTP con STARTTLS

**Connessione iniziale**

Client -\> Server : TCP porta 587

Server -\> Client : 220 mail.example.com ESMTP ready

**Identificazione SMTP**

Client -\> Server : EHLO client.example.com

Server -\> Client : 250-STARTTLS

Il server comunica di supportare STARTTLS.

**Attivazione TLS**

Client -\> Server : STARTTLS

Server -\> Client : 220 Ready to start TLS

A questo punto inizia il normale handshake TLS.

| **Protocollo**  | **Porta STARTTLS** |
|-----------------|--------------------|
| SMTP Submission | 587                |
| IMAP            | 143                |
| POP3            | 110                |

**Porte Tipiche**

**STARTTLS vs SMTPS**

Esistono due approcci differenti alla cifratura SMTP.

**STARTTLS**

La connessione:

- inizia in chiaro;

- viene successivamente cifrata.

Porta tipica: 587/TCP

**SMTPS (Implicit TLS)**

La connessione TLS viene avviata immediatamente all’apertura della sessione TCP.

Porta tipica: 465/TCP

In questo caso non esiste una fase iniziale in chiaro.

------------------------------------------------------------------------

**Vantaggi di STARTTLS**

- Compatibilità con sistemi legacy

- Possibilità di upgrade graduale della sicurezza

- Ampia diffusione nei server di posta

- Supporto standardizzato nei principali MTA

**Limiti di Sicurezza**

STARTTLS può essere vulnerabile ad attacchi di downgrade.

Un attaccante in posizione Man-in-the-Middle potrebbe:

- rimuovere l’annuncio STARTTLS;

- impedire il passaggio a TLS;

- mantenere la comunicazione in chiaro.

Questo tipo di attacco è noto come:

STARTTLS Stripping

**Contromisure**

Per mitigare tali rischi vengono utilizzati:

- TLS obbligatorio

- MTA-STS

- DANE

- validazione rigorosa dei certificati

- policy di cifratura forzata

------------------------------------------------------------------------

**Verifica Manuale con OpenSSL**

È possibile testare STARTTLS manualmente:

**SMTP**

openssl s_client -starttls smtp -connect mail.example.com:587

**IMAP**

openssl s_client -starttls imap -connect mail.example.com:143

**POP3**

openssl s_client -starttls pop3 -connect mail.example.com:110

Questi comandi permettono di:

- verificare i certificati;

- osservare l’handshake TLS;

- controllare protocolli e cipher suite supportate.

## 2.2 Dialogo SMTP

SMTP usa una comunicazione testuale basata su comandi ASCII. La versione moderna usa EHLO al posto di HELO per segnalare il supporto alle estensioni ESMTP (Extended SMTP, RFC 5321).

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><p>S: 220 mail.example.org ESMTP Postfix</p>
<p>C: EHLO mail.mittente.com</p>
<p>S: 250-mail.example.org</p>
<p>S: 250-SIZE 52428800</p>
<p>S: 250-STARTTLS</p>
<p>S: 250-AUTH PLAIN LOGIN</p>
<p>S: 250 8BITMIME</p>
<p>C: STARTTLS</p>
<p>S: 220 Ready to start TLS</p>
<p>[negoziazione TLS - da qui in poi il traffico e cifrato]</p>
<p>C: AUTH LOGIN</p>
<p>S: 334 Username:</p>
<p>C: [base64(username)]</p>
<p>S: 334 Password:</p>
<p>C: [base64(password)]</p>
<p>S: 235 Authentication successful</p>
<p>C: MAIL FROM:&lt;alice@mittente.com&gt;</p>
<p>S: 250 Ok</p>
<p>C: RCPT TO:&lt;bob@example.org&gt;</p>
<p>S: 250 Ok</p>
<p>C: DATA</p>
<p>S: 354 End data with &lt;CR&gt;&lt;LF&gt;.&lt;CR&gt;&lt;LF&gt;</p>
<p>C: Subject: Test</p>
<p>C: From: alice@mittente.com</p>
<p>C: To: bob@example.org</p>
<p>C:</p>
<p>C: Corpo del messaggio.</p>
<p>C: .</p>
<p>S: 250 Ok: queued as AB1234</p>
<p>C: QUIT</p>
<p>S: 221 Bye</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**Nota su HELO vs EHLO:** *HELO e il comando originale di SMTP (RFC 821, 1982). EHLO fu introdotto con ESMTP (RFC 1869, 1995) e permette al server di annunciare le estensioni supportate. Tutti i server moderni supportano EHLO; HELO è mantenuto per compatibilità.*

## 2.3 Codici di risposta SMTP

SMTP usa codici numerici a tre cifre strutturati secondo uno schema preciso: la prima cifra indica la categoria (2=successo, 3=attesa dati, 4=errore temporaneo, 5=errore permanente), la seconda e la terza specificano il tipo di risposta.

| **Codice** | **Significato** | **Contesto** |
|----|----|----|
| 220 | Servizio disponibile | Apertura connessione - benvenuto del server |
| 221 | Servizio in chiusura | Risposta a QUIT |
| 235 | Autenticazione riuscita | Dopo AUTH |
| 250 | Operazione completata | Risposta positiva a EHLO, MAIL FROM, RCPT TO, DATA |
| 354 | Invio corpo del messaggio | Il server e pronto a ricevere DATA - termina con . |
| 421 | Servizio non disponibile (transitorio) | Errore temporaneo - riprovare |
| 450 | Mailbox non disponibile (transitorio) | Errore temporaneo - es. volume pieno |
| 550 | Operazione rifiutata | Destinatario inesistente, policy di rifiuto |
| 551 | Utente non locale | Il server suggerisce un indirizzo alternativo |
| 553 | Nome mailbox non ammesso | Sintassi dell'indirizzo non valida |

## Perché non è corretto utilizzare la porta 25 per i client di posta?

La porta 25 è progettata principalmente per il relay SMTP tra server MTA. L’utilizzo da parte dei client è sconsigliato perché:

- spesso non richiede autenticazione

- è storicamente abusata da spammer e malware

- molti ISP la bloccano in uscita

Le porte raccomandate per i client sono:

- 587 con STARTTLS

- 465 con TLS implicito

poiché supportano autenticazione e trasporto cifrato in maniera più sicura.

**Esistono tool che cercano di sfruttare la porta 25 per attaccare i server SMTP?\**

Si, ad esempio **smtp-user-enum** che serve per enumerare utenti validi usa comandi come VRFY, EXPN, RCPT TO. Viene usato per scoprire email valide in un dominio. Spesso le richieste vengono rifiutate sai server smtp.

# 3. Evoluzione della sicurezza SMTP

## 3.1 Vulnerabilita storiche

Le prime implementazioni SMTP (RFC 821, 1982) non prevedevano autenticazione, cifratura ne verifica del mittente. Il campo MAIL FROM: (envelope sender) e il campo From: dell'header potevano contenere qualsiasi valore, rendendo il mail spoofing banale.

Era possibile connettersi manualmente tramite Telnet alla porta 25 di un mail server aperto (open relay) e inviare messaggi con qualsiasi mittente apparente. Questa tecnica e ancora usata in ambito forense e in penetration testing su ambienti autorizzati.

I problemi principali dell'SMTP originale erano:

- Open relay: server configurati per accettare e ritrasmettere posta da chiunque, sfruttati per spam massivo.

- Nessuna autenticazione del mittente: il campo From: e falsificabile senza vincoli tecnici.

- Traffico in chiaro: credenziali e contenuti intercettabili sulla rete.

- Header injection: possibilita di iniettare header aggiuntivi tramite input non sanificato.

## 3.2 STARTTLS e SMTPS

Per risolvere il problema della cifratura sono stati introdotti due meccanismi:

- STARTTLS (RFC 3207): estensione ESMTP che permette di aggiornare una connessione TCP inizialmente in chiaro a una connessione TLS. Il client annuncia il supporto via EHLO, il server risponde con '250 STARTTLS', il client invia il comando STARTTLS e si avvia la negoziazione TLS. Vantaggio: usa la stessa porta (25 o 587). Rischio: vulnerabile agli attacchi di downgrade se non configurato con enforce obbligatorio (MTA-STS).

- SMTPS / TLS implicito (porta 465): la connessione TCP viene stabilita e immediatamente avvolta in TLS, senza fase in chiaro. E il metodo preferito per i client moderni (RFC 8314, 2018).

## 3.3 Autenticazione SASL

SASL (Simple Authentication and Security Layer, RFC 4422) e un framework che separa i meccanismi di autenticazione dai protocolli applicativi. In SMTP viene usato tramite il comando AUTH.

Meccanismi comuni supportati da SMTP:

- PLAIN: username e password in base64 in un unico campo. Sicuro solo su canale cifrato (TLS).

- LOGIN: username e password trasmessi in due passaggi separati, entrambi in base64. Anch'esso sicuro solo su TLS.

- CRAM-MD5: challenge-response con HMAC-MD5. Non richiede TLS ma e considerato obsoleto.

- XOAUTH2: autenticazione tramite token OAuth 2.0 (usato da Gmail, Outlook moderni).

**Nota:** *PLAIN e LOGIN trasmettono le credenziali in base64, che e una codifica e non una cifratura. Se usati su connessione non cifrata, le credenziali sono recuperabili in chiaro da chiunque intercetti il traffico. E’ obbligatorio usarli solo dopo STARTTLS o su connessione TLS implicita.*

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Esercizio 1 - Analisi di una sessione SMTP</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p>Data la seguente sessione SMTP (parziale), rispondere alle domande:</p>
<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><p>220 mail.university.edu ESMTP</p>
<p>EHLO mail.attacker.com</p>
<p>250-mail.university.edu</p>
<p>250-STARTTLS</p>
<p>250 AUTH PLAIN LOGIN</p>
<p>MAIL FROM:&lt;president@university.edu&gt;</p>
<p>250 Ok</p>
<p>RCPT TO:&lt;all-students@university.edu&gt;</p>
<p>250 Ok</p>
<p>DATA</p>
<p>354 End data with &lt;CR&gt;&lt;LF&gt;.&lt;CR&gt;&lt;LF&gt;</p>
<p>Subject: Avviso urgente</p>
<p>From: Rettore &lt;president@university.edu&gt;</p>
<p>To: all-students@university.edu</p>
<p>Si comunica che le lezioni di domani sono sospese.</p>
<p>.</p>
<p>250 Ok: queued</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
<p>Domande:</p>
<ul>
<li><p>Quale campo è stato falsificato?</p></li>
<li><p>Quale vulnerabilità ha permesso questo invio?</p></li>
<li><p>Come avrebbero potuto SPF, DKIM e DMARC bloccare o rilevare questo messaggio?</p></li>
<li><p>Il server ha usato STARTTLS prima di ricevere MAIL FROM. Cosa implica questa assenza?</p></li>
<li><p>Quali header aggiuntivi, presenti in un messaggio reale, avrebbero permesso di rilevare la frode?</p></li>
</ul></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Soluzione</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p><strong>1. Campo falsificato:</strong></p>
<p>Il campo MAIL FROM: dell'envelope (alice@university.edu) e il campo From: dell'header (visualizzato dal client come mittente). Entrambi contengono un indirizzo del dominio university.edu, ma la connessione proviene da mail.attacker.com - un server non autorizzato per quel dominio.</p>
<p><strong>2. Vulnerabilita sfruttata:<br />
</strong>Il server ha accettato la connessione da un host esterno senza verificare che il dominio indicato in MAIL FROM fosse compatibile con l'IP sorgente (assenza di SPF enforcement). Inoltre non e stato richiesto AUTH prima di accettare MAIL FROM: il server si comporta come open relay parziale.</p>
<p><strong>3. SPF, DKIM, DMARC:<br />
</strong>Ne vedremo in dettaglio le caratteristico più avanti in questo capitolo. <strong><br />
</strong>Per ora osserviamo che si tratta di metodi per garantire l’identità del mail server mittente.</p>
<ul>
<li><p>SPF: un record SPF per university.edu autorizzerebbe solo gli IP dei mail server legittimi. La connessione da mail.attacker.com fallisce il check SPF (risultato 'fail' o 'softfail').</p></li>
<li><p>DKIM: il messaggio non porta una firma DKIM valida per university.edu (il server attaccante non ha la chiave privata del dominio). Il destinatario trova 'DKIM=none' o 'DKIM=fail'.</p></li>
<li><p>DMARC: con policy 'reject' o 'quarantine', un messaggio che fallisce SPF e DKIM viene rifiutato o messo in spam automaticamente, indipendentemente dal campo From:.</p></li>
</ul>
<p><strong>4. Assenza di STARTTLS:<br />
</strong>La sessione e avvenuta in chiaro sulla rete. Chiunque intercetti il traffico (attacco MITM o packet capture) puo leggere il contenuto del messaggio e gli header. Inoltre, se il server supporta AUTH ma non ha richiesto STARTTLS prima, le credenziali saranno trasmesse in chiaro.</p>
<p><strong>5. Header rivelatori:</strong></p>
<ul>
<li><p>Received: ogni MTA attraversato aggiunge un header Received: con IP sorgente, hostname e timestamp. Analizzando la catena di Received si risale al vero server di origine (mail.attacker.com).</p></li>
<li><p>Return-Path: contiene l'envelope sender reale (quello di MAIL FROM), che potrebbe differire dal From: dell'header.</p></li>
<li><p>Authentication-Results: aggiunto dai sistemi di filtraggio, riporta i risultati di SPF, DKIM e DMARC.</p></li>
<li><p>X-Originating-IP: header non standard aggiunto da alcuni server per tracciare l'IP del client originante.</p></li>
</ul></td>
</tr>
</tbody>
</table>

# 4. Protocolli di accesso alla mailbox: POP3 e IMAP

Dopo la consegna della posta al server destinatario, il client dell'utente usa un protocollo di accesso per recuperare o visualizzare i messaggi. I due protocolli principali sono POP3 e IMAP. Entrambi operano su TCP e hanno varianti cifrate.

| **Caratteristica** | **POP3** | **IMAP** |
|----|----|----|
| RFC | RFC 1939 | RFC 3501 (IMAP4rev1); RFC 9051 (IMAP4rev2, 2021) |
| Porta standard | 110 (in chiaro) | 143 (in chiaro) |
| Porta cifrata | 995 (POP3S / TLS) | 993 (IMAPS / TLS) |
| Modello | Download-and-delete (default) o keep | Server-side storage con sincronizzazione |
| Multi-device | Limitato - un dispositivo scarica, gli altri non vedono | Ottimo - tutti i dispositivi sincronizzati |
| Cartelle | Solo INBOX | Gerarchia di cartelle gestita lato server |
| Stato messaggi | Non sincronizzato | Letto/non letto, flag, risposto - sincronizzati |
| Uso offline | Ottimo (messaggi locali) | Limitato senza cache locale |
| Caso d'uso tipico | Client singolo, bassa dipendenza dal server | Webmail, smartphone, multi-device |

## 4.1 POP3 - comandi principali

Una sessione POP3 e strutturata in tre fasi: autorizzazione, transazione e aggiornamento.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><p>+OK POP3 server ready</p>
<p>USER alice</p>
<p>+OK</p>
<p>PASS secretpassword</p>
<p>+OK Logged in</p>
<p>LIST</p>
<p>+OK 3 messages:</p>
<p>1 1024</p>
<p>2 2048</p>
<p>3 512</p>
<p>.</p>
<p>RETR 1</p>
<p>+OK 1024 octets</p>
<p>[contenuto del messaggio 1]</p>
<p>.</p>
<p>DELE 1</p>
<p>+OK Message 1 deleted</p>
<p>QUIT</p>
<p>+OK Bye</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

Comandi POP3 principali:

- USER / PASS: autenticazione (trasmessa in chiaro su porta 110 - usare sempre POP3S).

- STAT: numero di messaggi e dimensione totale della mailbox.

- LIST \[n\]: elenca i messaggi con le rispettive dimensioni.

- RETR n: scarica il messaggio numero n.

- DELE n: marca il messaggio n per la cancellazione (eseguita effettivamente al QUIT).

- NOOP: nessuna operazione, usato per mantenere viva la connessione.

- RSET: annulla tutte le cancellazioni marcate nella sessione corrente.

- QUIT: chiude la sessione e applica le cancellazioni.

## 4.2 IMAP - caratteristiche avanzate

IMAP (RFC 3501) mantiene i messaggi sul server e permette operazioni sofisticate:

- Gestione di cartelle (INBOX, Sent, Drafts, Trash, cartelle personalizzate).

- Recupero parziale: e possibile scaricare solo header, solo corpo testo o solo un allegato specifico, risparmiando banda.

- Ricerca lato server: il client puo inviare criteri di ricerca (SEARCH FROM alice SINCE 1-Jan-2025) ed eseguire la ricerca direttamente sul server.

- Flag di stato: \Seen, \Answered, \Flagged, \Deleted, \Draft sincronizzati tra tutti i dispositivi.

- IDLE: meccanismo push che permette al server di notificare immediatamente il client dell'arrivo di nuovi messaggi, senza polling continuo.

**Nota su IMAP4rev2:** *RFC 9051 (2021) aggiorna IMAP4rev1 con vari miglioramenti: supporto obbligatorio a UTF-8, rimozione di meccanismi obsoleti, miglioramenti alla sicurezza..*

# 5. MIME e struttura delle e-mail moderne

## 5.1 MIME - Multipurpose Internet Mail Extensions

SMTP nacque per trasportare testo ASCII a 7 bit (RFC 821, 1982). Questo impediva l'invio di allegati binari, caratteri non ASCII (accenti, ideogrammi) e contenuto HTML. MIME (RFC 2045-2049, 1996) estese il formato delle e-mail senza modificare SMTP.

MIME introduce header aggiuntivi nel messaggio:

- MIME-Version: 1.0 - dichiara l'uso di MIME.

- Content-Type: tipo/sottotipo - specifica il tipo di contenuto (es. text/plain, text/html, image/jpeg, application/pdf).

- Content-Transfer-Encoding: specifica come il contenuto e stato codificato (7bit, 8bit, base64, quoted-printable).

- Content-Disposition: indica se il contenuto e da visualizzare inline o come allegato scaricabile (attachment; filename='doc.pdf').

## 5.2 Struttura multipart

Un messaggio MIME puo contenere piu parti separate da un boundary, una stringa univoca scelta dal mittente.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><p>MIME-Version: 1.0</p>
<p>Content-Type: multipart/mixed; boundary="----=_Part_12345"</p>
<p>------=_Part_12345</p>
<p>Content-Type: multipart/alternative; boundary="----=_Alt_67890"</p>
<p>------=_Alt_67890</p>
<p>Content-Type: text/plain; charset=UTF-8</p>
<p>Content-Transfer-Encoding: quoted-printable</p>
<p>Testo del messaggio in formato plain.</p>
<p>------=_Alt_67890</p>
<p>Content-Type: text/html; charset=UTF-8</p>
<p>Content-Transfer-Encoding: quoted-printable</p>
<p>&lt;html&gt;&lt;body&gt;&lt;p&gt;Testo del messaggio in &lt;b&gt;HTML&lt;/b&gt;.&lt;/p&gt;&lt;/body&gt;&lt;/html&gt;</p>
<p>------=_Alt_67890--</p>
<p>------=_Part_12345</p>
<p>Content-Type: application/pdf; name="documento.pdf"</p>
<p>Content-Transfer-Encoding: base64</p>
<p>Content-Disposition: attachment; filename="documento.pdf"</p>
<p>JVBERi0xLjQKJeLjz9MKMSAwIG9iag... [base64 del PDF]</p>
<p>------=_Part_12345--</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

Tipi MIME multipart comuni:

- multipart/mixed: parti di tipo diverso (testo + allegati).

- multipart/alternative: versioni alternative dello stesso contenuto (plain + HTML). Il client sceglie la migliore.

- multipart/related: contenuto con risorse correlate (HTML con immagini inline, usando Content-ID).

- multipart/signed e multipart/encrypted: usati con S/MIME per firme digitali e cifratura (RFC 5751).

## 5.3 Codifica Base64

Base64 (RFC 4648) e una codifica che rappresenta dati binari arbitrari usando solo 64 caratteri ASCII stampabili: lettere maiuscole (A-Z), minuscole (a-z), cifre (0-9), e i simboli + e /. Il carattere = e usato come padding per allineare l'output a multipli di 4 caratteri.

Funzionamento: ogni 3 byte (24 bit) di input vengono divisi in 4 gruppi da 6 bit, ciascuno mappato su un carattere dell'alfabeto Base64. Questo determina un aumento di dimensione del 33% circa (3 byte -\> 4 caratteri).

Esempio: il testo Man (3 byte: 0x4D 0x61 0x6E) diventa TWFu in Base64.

Proprieta fondamentali di Base64:

- Non è crittografia: chiunque può decodificarla senza chiave.

- Non protegge il contenuto: è una codifica, non una cifratura.

- Non rileva errori: non ha checksum integrato.

- Aumenta la dimensione del 33% circa.

Usi comuni oltre alle e-mail:

- JSON Web Token (JWT): il payload e l'header sono codificati in Base64URL (variante che usa - e \_ al posto di + e /).

- HTTP Basic Authentication: le credenziali username:password sono trasmesse in Base64 nell'header Authorization.

- Data URI: immagini inline in HTML/CSS (data:image/png;base64,...).

- API Web e REST: trasferimento di contenuto binario in payload JSON.

- Certificati digitali PEM: il contenuto DER e avvolto in Base64 tra header -----BEGIN CERTIFICATE-----.

**Attenzione - Base64 in sicurezza:** *HTTP Basic Auth con Base64 e banalmente decodificabile. Va usato solo su HTTPS. In JWT, la mancata verifica della firma (o l'accettazione dell'algoritmo 'none') è una vulnerabilità critica (CVE documentate).\*

## 5.4 Quoted-Printable

Quoted-Printable (RFC 2045) e un'alternativa a Base64 per contenuti prevalentemente ASCII con pochi caratteri non-ASCII (tipico per testo con accenti). I caratteri ASCII stampabili rimangono invariati; i caratteri fuori range vengono codificati come =XX dove XX e il valore esadecimale del byte. Aumenta la dimensione in misura molto minore di Base64 per testo europeo.

Esempio: Ciao, è una bella giornata! -\> Ciao, =C3=A8 una bella giornata!

# 6. Header delle e-mail e analisi forense

## 6.1 Struttura degli header

Ogni e-mail contiene un blocco di header separato dal corpo da una riga vuota. Gli header sono coppie 'Campo: Valore' e forniscono informazioni tecniche complete sul messaggio. Alcuni sono obbligatori, altri facoltativi, altri vengono aggiunti dagli MTA durante il transito.

| **Header** | **Tipo** | **Descrizione** |
|----|----|----|
| From: | Mittente originale | Indirizzo visualizzato dal client - FALSIFICABILE |
| To: / Cc: / Bcc: | Destinatari | Bcc: non appare nel messaggio consegnato |
| Subject: | Oggetto | Testo libero - non autenticato |
| Date: | Data/ora | Generata dal client - FALSIFICABILE |
| Message-ID: | Identificatore univoco | Generato dal MTA del mittente (es. \<abc123@mail.example.com\>) |
| Received: | Traccia di transito | Aggiunto da ogni MTA attraversato - difficile da falsificare completamente |
| Return-Path: | Envelope sender | Impostato dal MTA ricevente - corrisponde a MAIL FROM |
| MIME-Version: | Versione MIME | Tipicamente 1.0 |
| Content-Type: | Tipo contenuto | Tipo MIME del corpo o struttura multipart |
| Authentication-Results: | Esiti autenticazione | Aggiunto dal ricevente: risultati SPF, DKIM, DMARC |
| DKIM-Signature: | Firma DKIM | Aggiunta dal MTA mittente se configurato |
| X-Spam-Score: | Punteggio antispam | Header non standard aggiunto dai filtri antispam |

## 6.2 Analisi della catena Received

La catena di header Received: e lo strumento principale per tracciare il percorso reale di un messaggio. Ogni MTA che riceve e ritrasmette un messaggio aggiunge un header Received: in cima alla lista (il piu recente e in alto). L'analisi si legge dal basso verso l'alto: il Received: più in basso è quello aggiunto dal primo server che ha ricevuto la mail.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><p>Received: from mail.legitimate.com (mail.legitimate.com [198.51.100.10])</p>
<p>by mx.destinatario.com with ESMTPS id abc123</p>
<p>for &lt;user@destinatario.com&gt;;</p>
<p>Mon, 9 May 2026 10:23:14 +0000</p>
<p>Received: from [192.168.1.50] (unknown)</p>
<p>by mail.legitimate.com with ESMTP id xyz456;</p>
<p>Mon, 9 May 2026 10:23:10 +0000</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

In questo esempio: il messaggio e passato prima per 192.168.1.50 (IP del client mittente), poi per mail.legitimate.com, poi per mx.destinatario.com. L'IP del client originante (192.168.1.50) e spesso il dato piu utile per l'analisi forense.

**Nota:** *Gli header Received: possono essere parzialmente falsificati, ma solo quelli aggiunti dagli MTA che il mittente non controlla sono affidabili. Tipicamente, l'header Received: aggiunto dal server di ricezione finale è affidabile in quanto è aggiunto dal ricevente stesso.*

# 7. Contraffazione delle e-mail e meccanismi di autenticazione

## 7.1 Email spoofing

L'email spoofing e la tecnica con cui un attaccante invia messaggi con un indirizzo mittente falsificato, facendolo apparire come proveniente da un'organizzazione o persona fidata. E il vettore tecnico alla base di:

- Phishing: furto di credenziali tramite pagine false.

- Spear phishing: phishing mirato a individui specifici con informazioni personalizzate.

- BEC (Business Email Compromise): frodi aziendali tramite impersonificazione di dirigenti o fornitori.

- CEO fraud: variante di BEC in cui si impersona un CEO per autorizzare bonifici fraudolenti.

I campi falsificabili senza controlli tecnici sono:

- MAIL FROM: (envelope sender): usato per i bounce, non visibile all'utente finale.

- From: (header): quello visualizzato dal client mail - il piu sfruttato per il phishing.

- Reply-To: puo essere impostato su un indirizzo diverso per ricevere risposte.

- Display name: il nome visualizzato prima dell'indirizzo (es. 'Apple Support \<noreply@phisher.com\>').

**Importante:** *Il campo From: e interamente sotto il controllo del mittente. Un client di posta che mostra solo il display name e non l'indirizzo completo facilita il phishing basato sul display name spoofing.*

## 7.2 SPF - Sender Policy Framework

SPF (RFC 7208, 2014) permette al proprietario di un dominio di dichiarare nel DNS quali server IP sono autorizzati a inviare posta per quel dominio. Il server ricevente verifica l'IP del server mittente contro il record SPF del dominio indicato in MAIL FROM:.

Esempio di record SPF nel DNS:

| example.com. IN TXT "v=spf1 ip4:192.0.2.10 ip4:192.0.2.11 include:\_spf.google.com ~all" |
|----|

Meccanismi SPF principali:

- ip4: / ip6:: specifica un singolo indirizzo o range CIDR autorizzato.

- include:: include il record SPF di un altro dominio (utile per servizi terzi come G Suite).

- a:: autorizza gli IP nel record A del dominio.

- mx:: autorizza gli IP nei record MX del dominio.

- all: corrisponde a qualsiasi IP. Con i qualificatori: +all (autorizza tutto, inutile), ~all (softfail, segnala ma accetta), -all (fail, rifiuta).

Risultati della verifica SPF:

- pass: IP autorizzato.

- fail (-all): IP non autorizzato, dovrebbe essere rifiutato.

- softfail (~all): IP sospetto, trattato come spam ma non rifiutato.

- neutral (?all): nessuna affermazione.

- none: nessun record SPF pubblicato.

- temperror / permerror: errore temporaneo o di configurazione.

**Limitazione di SPF:** *SPF verifica solo il dominio dell'envelope sender (MAIL FROM:), non il campo From: dell'header visualizzato all'utente. Un attaccante puo avere SPF valido per il proprio dominio malevolo e comunque falsificare il From: header. DMARC risolve questo problema richiedendo l'allineamento tra i due.*

## 7.3 DKIM - DomainKeys Identified Mail

DKIM (RFC 6376, 2011) aggiunge una firma digitale al messaggio. Il server mittente firma porzioni del messaggio con la propria chiave privata RSA o Ed25519. La chiave pubblica e pubblicata nel DNS del dominio mittente come record TXT.

Processo di firma e verifica:

7.  Il MTA mittente calcola un hash di specifici header (From:, Subject:, To:, Date:) e del corpo del messaggio.

8.  L'hash viene firmato con la chiave privata RSA/Ed25519 del dominio.

9.  La firma viene aggiunta al messaggio come header DKIM-Signature:.

10. Il MTA destinatario legge il tag d= (dominio) e s= (selector) dalla firma.

11. Recupera la chiave pubblica dal DNS: \<selector\>.\_domainkey.\<dominio\>. IN TXT

12. Verifica la firma. Se il contenuto e stato modificato durante il transito, la verifica fallisce.

Esempio di header DKIM-Signature:

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><p>DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;</p>
<p>d=example.com; s=mail2024;</p>
<p>h=from:to:subject:date:message-id;</p>
<p>bh=47DEQpj8HBSa+/TImW+5JCeuQeRkm5NMpJWZG3hSuFU=;</p>
<p>b=AbCdEf....[firma base64]....XyZ=</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

Parametri chiave:

- d=: dominio che firma.

- s=: selector, permette di avere piu chiavi attive contemporaneamente (utile per rotazione).

- h=: lista degli header firmati.

- bh=: hash del corpo (body hash).

- b=: firma digitale vera e propria.

- a=: algoritmo (rsa-sha256 o ed25519-sha256).

**Limitazione di DKIM:** *DKIM non impedisce la ritrasmissione di un messaggio legittimo (replay attack): una mail firmata e valida puo essere rispedita a destinatari diversi senza invalidare la firma. DMARC con 'reject' mitiga questa classe di attacchi richiedendo ulteriori controlli.*

## 7.4 DMARC - Domain-based Message Authentication, Reporting and Conformance

DMARC (RFC 7489, 2015) completa e coordina SPF e DKIM aggiungendo due funzioni critiche: la policy di enforcement e l'allineamento del dominio.

DMARC richiede che il dominio nel campo From: (header) sia allineato con:

- il dominio che ha superato il check SPF (MAIL FROM:), oppure

- il dominio che ha apposto una firma DKIM valida (campo d=).

Esempio di record DMARC nel DNS:

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><p>_dmarc.example.com. IN TXT "v=DMARC1; p=reject; sp=quarantine; pct=100;</p>
<p>rua=mailto:dmarc-reports@example.com;</p>
<p>ruf=mailto:forensic@example.com; adkim=r; aspf=r"</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

Parametri principali:

- p=: policy per il dominio principale (none \| quarantine \| reject).

- sp=: policy per i sottodomini.

- pct=: percentuale di messaggi a cui applicare la policy (utile per rollout graduale).

- rua=: indirizzo per i report aggregati (ricezione giornaliera di statistiche in XML).

- ruf=: indirizzo per i report forensi (copia dei messaggi falliti - non sempre supportato).

- adkim=: modalita di allineamento DKIM (r=relaxed, s=strict).

- aspf=: modalita di allineamento SPF (r=relaxed, s=strict).

Le tre policy DMARC e le loro implicazioni operative:

| **Policy** | **Comportamento** | **Uso tipico** |
|----|----|----|
| none | Accetta il messaggio, invia solo report. Non blocca nulla. | Fase di monitoraggio iniziale, raccolta dati |
| quarantine | Consegna in spam/quarantena. Il messaggio arriva ma e isolato. | Fase intermedia, dopo aver verificato i report |
| reject | Rifiuta il messaggio a livello SMTP (non consegnato). Protezione massima. | Produzione, dopo validazione completa |

**Deployment progressivo:** *La raccomandazione per l'adozione di DMARC e incrementale: iniziare con p=none per raccogliere report e verificare che tutta la posta legittima superi SPF e DKIM, poi passare a quarantine, infine a reject. Un deployment affrettato con reject puo bloccare mail legittime.*

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Esercizio 3 - Analisi SPF, DKIM e DMARC</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p>Un'azienda ha configurato i seguenti record DNS:</p>
<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><p>company.com. IN TXT "v=spf1 ip4:203.0.113.5 include:sendgrid.net ~all"</p>
<p>_dmarc.company.com IN TXT "v=DMARC1; p=quarantine; pct=50; rua=mailto:dmarc@company.com"</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
<p>Un attaccante invia una mail con From: ceo@company.com dal proprio server 45.33.32.156, senza firma DKIM.</p>
<p>Domande:</p>
<ul>
<li><p>Qual e il risultato del check SPF per questa mail?</p></li>
<li><p>Qual e il risultato del check DKIM?</p></li>
<li><p>Come si comporta DMARC? La mail viene consegnata?</p></li>
<li><p>Cosa significa pct=50 nel record DMARC?</p></li>
<li><p>Suggerisci come rafforzare la configurazione per la massima protezione.</p></li>
</ul></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Soluzione</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p><strong>1. SPF:</strong> L'IP 45.33.32.156 non e tra quelli autorizzati (203.0.113.5 o quelli di sendgrid.net). Il qualificatore ~all produce un risultato 'softfail': il messaggio non viene rifiutato automaticamente da SPF, ma viene marcato come sospetto.</p>
<p><strong>2. DKIM:</strong> Non essendoci firma DKIM, il risultato e 'none'. Non c'e alcuna firma da verificare. Il check DKIM fallisce per assenza.</p>
<p><strong>3. DMARC:</strong> Entrambi i check di autenticazione (SPF softfail, DKIM none) non producono un 'pass' allineato con il dominio company.com nel campo From:. DMARC fallisce. Con p=quarantine e pct=50, il 50% dei messaggi falliti viene messo in quarantena (spam); il rimanente 50% viene consegnato normalmente. La mail potrebbe essere consegnata se rientra nel 50% non soggetto alla policy in questo deployment parziale.</p>
<p><strong>4. pct=50:</strong> Il parametro pct indica la percentuale di messaggi NON autenticati a cui applicare la policy DMARC. Con pct=50, la meta dei messaggi che falliscono DMARC vengono messi in quarantena; l'altra meta viene consegnata normalmente. Serve per un rollout graduale che riduce il rischio di bloccare mail legittime.</p>
<p><strong>5. Rafforzamento consigliato:</strong></p>
<ul>
<li><p>Cambiare ~all in -all nel record SPF: da softfail a fail esplicito, segnalando chiaramente che gli IP non autorizzati non devono essere accettati.</p></li>
<li><p>Configurare la firma DKIM su tutti i mail server che inviano per company.com (incluso SendGrid con signing domain alignment).</p></li>
<li><p>Raccogliere e analizzare i report DMARC (rua=) per verificare che tutta la posta legittima superi i check.</p></li>
<li><p>Aumentare pct a 100 dopo aver verificato i report.</p></li>
<li><p>Cambiare p=quarantine in p=reject per la massima protezione, una volta accertato che non ci sono mail legittime bloccate.</p></li>
<li><p>Considerare MTA-STS (RFC 8461) per forzare TLS sulle connessioni SMTP in ingresso e prevenire attacchi di downgrade su STARTTLS.</p></li>
</ul></td>
</tr>
</tbody>
</table>

# 8. Servizi di Threat Intelligence e Reputation

La threat intelligence e l'insieme di informazioni strutturate su attori malevoli, tecniche di attacco, indicatori di compromissione (IoC) e infrastrutture ostili. I servizi di reputation valutano l'affidabilita di IP, domini, URL e file.

## 8.1 Spamhaus

Spamhaus e un'organizzazione non-profit che gestisce alcune delle principali blocklist (DNS Blacklists, DNSBL) usate dai mail server globali. Le principali liste:

- SBL (Spamhaus Block List): IP di spam sources noti.

- XBL (Exploits Block List): IP di sistemi compromessi (botnet, proxy aperti).

- PBL (Policy Block List): range IP non autorizzati a inviare posta direttamente (es. IP residenziali degli ISP).

- DBL (Domain Block List): domini usati in spam e phishing.

- ZEN: lista combinata SBL+XBL+PBL.

L'interrogazione avviene tramite DNS: il mail server verifica se l'IP sorgente compare nella DNSBL prima di accettare la connessione SMTP.

## 8.2 VirusTotal

VirusTotal (acquisito da Google/Alphabet nel 2012, ora parte di Chronicle/Google Cloud) e una piattaforma che analizza file, URL, domini e indirizzi IP usando oltre 70 motori antivirus e scanner di sicurezza.

Funzionalita principali:

- Analisi di file: carica un file e ottieni i risultati di 70+ AV engine. Il file viene memorizzato permanentemente - attenzione ai documenti confidenziali.

- Analisi di URL: verifica se un URL e classificato come malevolo da diversi servizi.

- Ricerca per hash (MD5, SHA1, SHA256): verifica se un file con quel hash e gia stato analizzato senza caricarlo nuovamente.

- Analisi di dominio e IP: storico delle analisi, WHOIS, record DNS, certificati SSL, relazioni con altri IoC.

- API: integrazione in SOC, SIEM, script automatizzati.

E uno strumento centrale nelle attivita SOC (Security Operations Center) e OSINT (Open Source Intelligence).

## 8.3 Cisco Talos

Cisco Talos e uno dei team di threat intelligence piu grandi al mondo, che fornisce dati di reputazione e intelligence integrati nei prodotti Cisco (Firepower, Email Security Appliance, Umbrella) e accessibili pubblicamente tramite talosintelligence.com.

Funzionalita dello strumento pubblico:

- Reputation lookup per IP e domini: punteggio da 'Trusted' a 'Untrusted' con storico.

- Volume e-mail: grafico del traffico di posta associato a un dominio o IP negli ultimi 30 giorni. Un picco improvviso puo indicare una campagna di phishing o spam.

- WHOIS e informazioni di registrazione.

- File hash lookup.

## 8.4 Have I Been Pwned

Have I Been Pwned (HIBP), creato da Troy Hunt nel 2013, e un servizio che permette di verificare se un indirizzo e-mail o un numero di telefono compare in database di credenziali trafugate (data breach) resi pubblici o scoperti online.

Se un account compare in un breach:

- Cambiare immediatamente la password su quel servizio.

- Cambiare la stessa password su tutti gli altri servizi dove e riutilizzata (credential stuffing).

- Attivare MFA/2FA (Multi-Factor Authentication) ovunque disponibile.

- Verificare se ci sono sessioni attive non autorizzate.

HIBP espone anche un'API pubblica usata da browser (Firefox Monitor, Chrome) per avvisare automaticamente gli utenti. Il servizio 'Pwned Passwords' permette di verificare se una password specifica compare nei database di breach senza trasmettere la password in chiaro (usa un protocollo k-anonymity con SHA-1 prefisso).

## 8.5 AlienVault OTX

AlienVault OTX (Open Threat Exchange), ora parte di AT&T Cybersecurity, e una piattaforma collaborativa di threat intelligence in cui ricercatori e organizzazioni condividono indicatori di compromissione (IoC) sotto forma di 'pulse' (collezioni di IoC correlati). Ogni pulse puo contenere IP, domini, hash, URL e CVE correlati a una campagna o a un malware specifico.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Esercizio 4 - Analisi header e threat intelligence</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p>Uno studente riceve una mail sospetta con i seguenti header (estratto):</p>
<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><p>Received: from mail-eu1.suspicious-domain.ru (mail-eu1.suspicious-domain.ru [185.220.101.45])</p>
<p>by mx.university.edu with ESMTP;</p>
<p>Mon, 9 May 2026 08:14:22 +0000</p>
<p>From: IT Support &lt;itsupport@university.edu&gt;</p>
<p>Reply-To: helpdesk-reset@gmail.com</p>
<p>Subject: Urgent: Your account will be suspended</p>
<p>Authentication-Results: mx.university.edu;</p>
<p>spf=fail (sender IP 185.220.101.45 is not permitted)</p>
<p>dkim=none</p>
<p>dmarc=fail action=quarantine</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
<p>Domande:</p>
<ul>
<li><p>Elenca almeno quattro indicatori che segnalano questa mail come phishing.</p></li>
<li><p>Quali strumenti di threat intelligence useresti per approfondire l'analisi sull'IP 185.220.101.45 e sul dominio suspicious-domain.ru?</p></li>
<li><p>L'universita ha DMARC con p=quarantine. Perche la mail e comunque arrivata nella casella dell'utente?</p></li>
<li><p>Cosa suggerisce il campo Reply-To: helpdesk-reset@gmail.com?</p></li>
<li><p>Se l'utente clicca sul link nel corpo della mail, quali rischi incontra?</p></li>
</ul></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Soluzione</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p><strong>1. Indicatori di phishing:</strong></p>
<ul>
<li><p>SPF fail: il server 185.220.101.45 non e autorizzato a inviare per university.edu.</p></li>
<li><p>DKIM none: nessuna firma DKIM - un server legittimo di una grande universita avrebbe quasi certamente DKIM.</p></li>
<li><p>DMARC fail: entrambi i check falliscono, confermando che il mittente non e il dominio university.edu.</p></li>
<li><p>Reply-To anomalo: le risposte andrebbero a helpdesk-reset@gmail.com, non a un indirizzo istituzionale. Questo e un classico indicatore di phishing con harvest delle credenziali.</p></li>
<li><p>Dominio mittente .ru: il TLD .ru non ha correlazione con una universita (presumibilmente non russa).</p></li>
<li><p>Oggetto urgente ('Urgent', 'suspended'): tecnica di social engineering per indurre azione immediata impulsiva.</p></li>
<li><p>IP nel range Tor exit node: 185.220.101.0/24 e un range noto per exit node Tor, usato per anonimizzare attivita malevole.</p></li>
</ul>
<p><strong>2. Strumenti di threat intelligence:</strong></p>
<ul>
<li><p>VirusTotal: analisi dell'IP 185.220.101.45 e del dominio suspicious-domain.ru per vedere le detection degli AV e lo storico.</p></li>
<li><p>Cisco Talos: reputation lookup dell'IP e del dominio, volume e-mail, storico.</p></li>
<li><p>Spamhaus ZEN: verifica se l'IP e in blocklist (probabilmente si trovera in SBL o XBL).</p></li>
<li><p>AbuseIPDB: database collaborativo di segnalazioni di abuso per IP.</p></li>
<li><p>WHOIS: informazioni su registrazione del dominio, data di creazione (dominio giovane = sospetto).</p></li>
<li><p>Shodan: servizi esposti sull'IP 185.220.101.45.</p></li>
</ul>
<p><strong>3. Perche la mail e arrivata con p=quarantine:</strong> DMARC con p=quarantine non rifiuta il messaggio a livello SMTP: lo consegna nella cartella spam/quarantena. Se il client mail o il sistema di posta non separa correttamente la quarantena dalla posta normale, l'utente potrebbe vederla nella inbox. Inoltre, con pct&lt;100 una percentuale di messaggi potrebbe bypassare la policy. Solo p=reject blocca definitivamente il messaggio a livello SMTP prima della consegna.</p>
<p><strong>4. Reply-To: helpdesk-reset@gmail.com:</strong> Il campo Reply-To: reindirizza le risposte dell'utente verso un indirizzo controllato dall'attaccante (helpdesk-reset@gmail.com), non verso l'istituzione impersonata. Questo permette all'attaccante di raccogliere le risposte degli utenti che rispondono senza cliccare sul link. E una tecnica comune per phishing tramite risposta (es. CEO fraud, 'rispondimi con le tue credenziali').</p>
<p><strong>5. Rischi se si clicca il link:</strong></p>
<ul>
<li><p>Furto di credenziali: landing page che imita il portale di autenticazione dell'universita.</p></li>
<li><p>Download di malware (drive-by download) tramite exploit del browser o plugin.</p></li>
<li><p>Esecuzione di script JavaScript malevoli (XSS stored su sito compromesso).</p></li>
<li><p>Fingerprinting del browser: raccolta di informazioni sul sistema dell'utente.</p></li>
<li><p>Redirect a catena verso URL sempre piu nascosti per eludere i filtri.</p></li>
</ul></td>
</tr>
</tbody>
</table>

# 9. Architettura stratificata delle reti

## 9.1 Principio della stratificazione

Le comunicazioni di rete sono organizzate a livelli (layer). L'idea fondamentale e che ogni livello offre servizi al livello superiore e usa i servizi del livello inferiore, comunicando con il livello equivalente del sistema remoto tramite un protocollo di pari livello. Questo approccio garantisce:

- Modularita: ogni livello puo essere modificato o sostituito senza impattare gli altri, purche l'interfaccia rimanga la stessa.

- Interoperabilita: prodotti di vendor diversi possono comunicare se rispettano gli stessi standard di livello.

- Astrazione: il livello superiore non deve conoscere i dettagli implementativi del livello inferiore.

- Riusabilita: lo stesso protocollo di livello puo essere usato da applicazioni diverse.

## 9.2 Modello OSI a 7 livelli

Il modello OSI (Open Systems Interconnection), sviluppato dall'ISO nel 1984 (ISO 7498), descrive l'architettura di rete in 7 livelli. E un modello di riferimento concettuale, non direttamente implementato nella pratica (Internet usa TCP/IP), ma fondamentale per descrivere e discutere i protocolli.

| **Livello** | **Nome** | **Funzione** | **Protocolli/tecnologie esempi** | **PDU** |
|----|----|----|----|----|
| 7 | Applicazione | Interfaccia con le applicazioni utente | HTTP, SMTP, DNS, FTP, SSH | Messaggio/Dati |
| 6 | Presentazione | Formato dei dati, codifica, cifratura | TLS/SSL, MIME, ASCII, UTF-8, JPEG | Dati |
| 5 | Sessione | Gestione delle sessioni di comunicazione | NetBIOS, RPC, TLS (handshake) | Dati |
| 4 | Trasporto | Consegna end-to-end, affidabilita, controllo flusso | TCP, UDP, SCTP | Segmento (TCP) / Datagramma (UDP) |
| 3 | Rete | Indirizzamento logico e instradamento | IP (IPv4, IPv6), ICMP, BGP, OSPF | Pacchetto |
| 2 | Collegamento dati | Trasferimento affidabile su link fisico, indirizzamento MAC | Ethernet, Wi-Fi (802.11), PPP, VLAN (802.1Q) | Frame |
| 1 | Fisico | Trasmissione bit sul mezzo fisico | Cavo UTP, fibra ottica, onde radio, segnali elettrici | Bit |

**Nota pratica:** *Nella pratica professionale il modello OSI e usato per la comunicazione e il troubleshooting ('il problema e a livello 2' vs 'a livello 3') piu che come specifica implementativa. Il modello TCP/IP reale usa 4 livelli (accesso alla rete, Internet, trasporto, applicazione) che collassano i livelli OSI 1-2, 5-7.*

## 9.3 Incapsulamento e decapsulamento

Quando un'applicazione invia dati, ogni livello aggiunge una propria intestazione (header) incapsulando il PDU del livello superiore. Al livello fisico vengono trasmessi i bit. Il ricevente esegue il processo inverso (decapsulamento), rimuovendo ogni header man mano che risale i livelli.

Esempio: una richiesta HTTP:

- L7 (HTTP): dati applicativi + header HTTP (GET /index.html HTTP/1.1).

- L4 (TCP): aggiunge header TCP (porte sorgente e destinazione, sequence number, flags).

- L3 (IP): aggiunge header IP (indirizzi IP sorgente e destinazione, TTL).

- L2 (Ethernet): aggiunge header Ethernet (indirizzi MAC sorgente e destinazione) e FCS (checksum).

- L1 (Fisico): converte i bit in segnali elettrici, ottici o radio.

# 10. RFC - Request for Comments

Le RFC (Request for Comments) sono documenti tecnici pubblicati dall'IETF (Internet Engineering Task Force) che definiscono i protocolli, le procedure, i formati e le best practice di Internet. Il nome storico e fuorviante: le RFC pubblicate dopo il processo di revisione sono standard definitivi, non bozze aperte a commenti.

Categorie principali di RFC:

- Standards Track: protocolli in via di standardizzazione (Proposed Standard -\> Internet Standard).

- Best Current Practice (BCP): raccomandazioni operative e di configurazione.

- Informational: documenti informativi, descrittivi o storici.

- Experimental: protocolli sperimentali non ancora standardizzati.

- Historic: documenti obsoleti o superati.

RFC rilevanti per questa lezione:

| **RFC**        | **Titolo (abbreviato)**                        | **Anno**    |
|----------------|------------------------------------------------|-------------|
| RFC 821 / 5321 | SMTP - Simple Mail Transfer Protocol           | 1982 / 2008 |
| RFC 1939       | POP3 - Post Office Protocol version 3          | 1996        |
| RFC 3501       | IMAP4rev1 - Internet Message Access Protocol   | 2003        |
| RFC 2045-2049  | MIME - Multipurpose Internet Mail Extensions   | 1996        |
| RFC 4648       | Base64 e codifiche correlate                   | 2006        |
| RFC 7208       | SPF - Sender Policy Framework                  | 2014        |
| RFC 6376       | DKIM - DomainKeys Identified Mail              | 2011        |
| RFC 7489       | DMARC - Domain-based Message Authentication    | 2015        |
| RFC 8314       | Uso di TLS per e-mail (porta 465 reintrodotta) | 2018        |
| RFC 3207       | STARTTLS per SMTP                              | 2002        |
| RFC 9051       | IMAP4rev2 (aggiornamento)                      | 2021        |

Le RFC sono accessibili gratuitamente su https://www.rfc-editor.org. Ogni RFC ha un numero progressivo univoco; i numeri non vengono riutilizzati. Quando uno standard viene aggiornato, viene pubblicata una nuova RFC che 'obsoletes' la precedente.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Esercizio 5 - Analisi completa di un'e-mail sospetta</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p>Un utente riceve la seguente e-mail. Analizzare la struttura tecnica e identificare tutti gli elementi rilevanti per la sicurezza.</p>
<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><p>Received: from smtp.paypaI-security.com (smtp.paypaI-security.com [91.108.4.200])</p>
<p>by mx.victim.com with ESMTP;</p>
<p>Mon, 9 May 2026 14:33:01 +0000</p>
<p>MIME-Version: 1.0</p>
<p>Content-Type: multipart/alternative; boundary="boundary123"</p>
<p>From: PayPal Security &lt;security@paypal.com&gt;</p>
<p>Reply-To: security@paypaI-security.com</p>
<p>Subject: Your account has been limited</p>
<p>Authentication-Results: mx.victim.com;</p>
<p>spf=none dkim=none dmarc=none</p>
<p>Message-ID: &lt;abc@paypaI-security.com&gt;</p>
<p>--boundary123</p>
<p>Content-Type: text/plain</p>
<p>Dear Customer, your account has been limited. Click here: http://paypaI-security.com/verify</p>
<p>--boundary123</p>
<p>Content-Type: text/html</p>
<p>&lt;html&gt;&lt;body&gt;Dear Customer, your account has been &lt;b&gt;limited&lt;/b&gt;.</p>
<p>&lt;a href='http://paypaI-security.com/verify'&gt;Click here to verify&lt;/a&gt;&lt;/body&gt;&lt;/html&gt;</p>
<p>--boundary123--</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
<p>Domande:</p>
<ul>
<li><p>Identifica il typosquatting nel dominio del mittente e nell'URL.</p></li>
<li><p>Cosa indicano i risultati 'spf=none dkim=none dmarc=none'?</p></li>
<li><p>Il messaggio ha struttura multipart/alternative. Qual e la ragione tecnica di questa scelta da parte degli attaccanti?</p></li>
<li><p>Quali altri IoC (Indicators of Compromise) sono presenti?</p></li>
<li><p>Come verificheresti l'URL http://paypaI-security.com/verify prima di aprirlo?</p></li>
</ul></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr>
<th><strong>Soluzione</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><p><strong>1. Typosquatting:</strong> Il dominio paypaI-security.com usa la lettera maiuscola 'I' (i maiuscola) al posto della lettera 'l' (elle minuscola) nella parola 'Paypal'. Molti font non distinguono visivamente 'I' da 'l'. Il dominio e completamente diverso da paypal.com ma visivamente quasi identico. La stessa sostituzione appare nel Reply-To e nell'URL del link. Questa tecnica si chiama IDN homograph attack o lookalike domain.</p>
<p><strong>2. spf=none dkim=none dmarc=none:</strong> Tutti e tre i valori 'none' indicano che il dominio paypal.com (o paypaI-security.com) non ha pubblicato record SPF, DKIM ne DMARC. In realta paypal.com ha tutti e tre configurati con p=reject: questi risultati 'none' si riferiscono al dominio falsificato (paypaI-security.com) che probabilmente non ha record DNS di autenticazione configurati, o il server ha verificato il dominio sbagliato. In ogni caso, l'assenza di autenticazione positiva per un'email da un grande provider finanziario e un forte segnale di allarme.</p>
<p><strong>3. Struttura multipart/alternative:</strong> Gli attaccanti usano multipart/alternative (text/plain + text/html) per due motivi: alcuni filtri antispam analizzano il testo plain e potrebbero non processare l'HTML con la stessa attenzione; alcuni client mail potrebbero visualizzare solo il testo plain e non mostrare il link HTML malevolo; avere entrambe le versioni aumenta la probabilita di passare i filtri di contenuto e di essere visualizzato correttamente su piu client.</p>
<p><strong>4. Altri IoC:</strong></p>
<ul>
<li><p>Dominio giovane / sconosciuto: paypaI-security.com e un dominio di phishing registrato di recente.</p></li>
<li><p>IP 91.108.4.200: range noto per hosting di infrastrutture malevole (verificabile su VirusTotal/Talos).</p></li>
<li><p>Reply-To diverso dal From: le risposte vanno a security@paypaI-security.com, non a paypal.com.</p></li>
<li><p>Oggetto urgente: 'limited', 'suspended' sono leve classiche di social engineering.</p></li>
<li><p>URL non HTTPS: http:// (non cifrato) per una pagina di 'verifica' finanziaria e anomalo.</p></li>
<li><p>Message-ID dal dominio phishing: &lt;abc@paypaI-security.com&gt; - normalmente un Message-ID legittimo di PayPal avrebbe @paypal.com.</p></li>
</ul>
<p><strong>5. Verifica sicura dell'URL:</strong></p>
<ul>
<li><p>NON aprire direttamente nel browser: rischio drive-by download.</p></li>
<li><p>VirusTotal: incollare l'URL su virustotal.com per verifica con 70+ scanner.</p></li>
<li><p>Cisco Talos / URLhaus: verifica reputazione del dominio.</p></li>
<li><p>WHOIS: data di registrazione del dominio (se registrato di recente = sospetto).</p></li>
<li><p>Browser sandboxing: aprire in una VM isolata o usare any.run / urlscan.io per analisi dinamica.</p></li>
<li><p>Ispezione manuale: analizzare il codice sorgente della pagina senza eseguire JavaScript.</p></li>
</ul></td>
</tr>
</tbody>
</table>

# Bibliografia

Le opere sono organizzate per argomento e citate in formato APA 7a edizione.

## Protocolli e-mail - RFC fondamentali

Klensin, J. (2008). *Simple Mail Transfer Protocol* (RFC 5321). IETF. https://www.rfc-editor.org/rfc/rfc5321

Myers, J., & Rose, M. (1996). *Post Office Protocol - Version 3* (RFC 1939). IETF. https://www.rfc-editor.org/rfc/rfc1939

Crispin, M. (2003). *Internet Message Access Protocol - Version 4rev1* (RFC 3501). IETF. https://www.rfc-editor.org/rfc/rfc3501

Melnikov, A., & Leiba, B. (2021). *Internet Message Access Protocol (IMAP) - Version 4rev2* (RFC 9051). IETF. https://www.rfc-editor.org/rfc/rfc9051

Freed, N., & Borenstein, N. (1996). *Multipurpose Internet Mail Extensions (MIME) Part One: Format of Internet Message Bodies* (RFC 2045). IETF. https://www.rfc-editor.org/rfc/rfc2045

Josefsson, S. (2006). *The Base16, Base32, and Base64 Data Encodings* (RFC 4648). IETF. https://www.rfc-editor.org/rfc/rfc4648

## Autenticazione e-mail

Kitterman, S. (2014). *Sender Policy Framework (SPF) for Authorizing Use of Domains in Email, Version 1* (RFC 7208). IETF. https://www.rfc-editor.org/rfc/rfc7208

Crocker, D., Hansen, T., & Kucherawy, M. (2011). *DomainKeys Identified Mail (DKIM) Signatures* (RFC 6376). IETF. https://www.rfc-editor.org/rfc/rfc6376

Kucherawy, M., & Zwicky, E. (2015). *Domain-based Message Authentication, Reporting, and Conformance (DMARC)* (RFC 7489). IETF. https://www.rfc-editor.org/rfc/rfc7489

Margolis, D., Risher, M., Ramakrishnan, B., Brotman, A., & Jones, J. (2018). *SMTP MTA Strict Transport Security (MTA-STS)* (RFC 8461). IETF. https://www.rfc-editor.org/rfc/rfc8461

Levine, J., & Delany, M. (2018). *Cleartext Considered Obsolete: Use of Transport Layer Security (TLS) for Email Submission and Access (RFC 8314)*. IETF. https://www.rfc-editor.org/rfc/rfc8314

## Sicurezza e-mail e phishing

Anti-Phishing Working Group. (2024). *Phishing activity trends report*. APWG. https://apwg.org/trendsreports/

Jakobsson, M., & Myers, S. (Eds.). (2007). *Phishing and countermeasures: Understanding the increasing problem of electronic identity theft*. Wiley.

Stone-Gross, B., Cova, M., Cavallaro, L., Gilbert, B., Szydlowski, M., Kemmerer, R., Kruegel, C., & Vigna, G. (2009). Your botnet is my botnet: Analysis of a botnet takeover. *Proceedings of the 16th ACM Conference on Computer and Communications Security (CCS '09)*, 635-647. https://doi.org/10.1145/1653662.1653738

## Architettura di rete e modello OSI

Tanenbaum, A. S., & Wetherall, D. J. (2011). *Computer networks* (5th ed.). Prentice Hall.

Kurose, J. F., & Ross, K. W. (2021). *Computer networking: A top-down approach* (8th ed.). Pearson.

ISO/IEC 7498-1:1994. (1994). *Information technology - Open Systems Interconnection - Basic Reference Model: The basic model*. ISO. https://www.iso.org/standard/20269.html

## Threat intelligence e strumenti

Spamhaus Project. (2024). *Spamhaus blocklist documentation*. https://www.spamhaus.org/faq/

VirusTotal. (2024). *VirusTotal documentation*. Google/Chronicle. https://docs.virustotal.com/

Cisco Systems. (2024). *Talos intelligence - IP and domain reputation*. https://talosintelligence.com/

Hunt, T. (2013). *Have I Been Pwned: FAQ and methodology*. https://haveibeenpwned.com/FAQs

**Nota:** *Tutte le RFC citate sono accessibili gratuitamente su https://www.rfc-editor.org e costituiscono la fonte normativa primaria per i protocolli descritti. I riferimenti a strumenti online (VirusTotal, Talos, HIBP) rimandano alle rispettive documentazioni ufficiali, che possono subire aggiornamenti nel tempo.*
