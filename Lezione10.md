**LEZIONE 10**

Cloud Computing: Architettura, Sicurezza e Governance

# 1. Introduzione al Cloud Computing

Quando oggi si parla di "cloud", si pensa immediatamente a piattaforme come Amazon Web Services (AWS), Microsoft Azure o Google Cloud. In realtà, l'idea alla base del cloud computing è molto più antica di quanto si possa immaginare.

## 1.1 Origini storiche: il time-sharing e l'utility computing

Le radici concettuali del cloud risalgono agli anni Sessanta, quando i computer erano enormi, costosi e accessibili solo a grandi organizzazioni. In quel contesto nacque il concetto di time-sharing: più utenti potevano condividere la stessa macchina centrale utilizzandola da terminali remoti.

Nel 1961, in un discorso tenuto per il centenario del MIT, John McCarthy — noto principalmente per aver coniato il termine "intelligenza artificiale" — ipotizzò pubblicamente che la potenza computazionale potesse un giorno essere distribuita come un servizio pubblico, analogamente all'energia elettrica o al sistema telefonico. Questa visione, denominata utility computing, rappresenta la prima formulazione esplicita del modello cloud. McCarthy affermò testualmente: "Computing may someday be organized as a public utility just as the telephone system is a public utility."

*Nota: il discorso di McCarthy al MIT Centennial del 1961 è documentato e verificato. Il concetto di time-sharing era tuttavia già in sviluppo prima di quella data, e McCarthy stesso fu coinvolto nello sviluppo del Compatible Time-Sharing System (CTSS) al MIT.*

J. C. R. Licklider, uno dei principali promotori di ARPANET (la rete precursore di Internet), contribuì in modo indipendente allo sviluppo dell'idea di elaborazione distribuita e accesso remoto alle risorse, concependo nel 1963 la visione di un "Intergalactic Computer Network" in cui macchine connesse avrebbero condiviso risorse e informazioni globalmente.

Negli anni Settanta e Ottanta si diffusero i sistemi mainframe IBM e le architetture centralizzate: molte grandi aziende operavano già secondo un modello simile a un "cloud privato", con terminali leggeri collegati a potenti sistemi centrali.

## 1.2 La nascita del cloud moderno

Il cloud moderno prese forma tra la fine degli anni Novanta e i primi anni Duemila, grazie a tre elementi fondamentali: la diffusione globale di Internet, la virtualizzazione e l'automazione dell'infrastruttura.

La virtualizzazione ebbe un ruolo decisivo. VMware, fondata nel 1998, dimostrò su larga scala che un singolo server fisico poteva ospitare molte macchine virtuali indipendenti, rendendo l'utilizzo dell'hardware molto più efficiente. Invece di dedicare un intero server fisico a una singola applicazione, le risorse potevano essere condivise dinamicamente.

Il momento comunemente considerato la vera nascita commerciale del cloud moderno arrivò nel 2006, quando Amazon lanciò Amazon S3 (Simple Storage Service) a marzo e Amazon EC2 (Elastic Compute Cloud) ad agosto in beta version, nell'ambito di AWS. Amazon aveva costruito enormi infrastrutture interne per il proprio e-commerce e si rese conto di poter rendere disponibili le risorse in eccesso ad altre aziende.

*Timeline: EC2 fu lanciato in beta pubblica limitata il 25 agosto 2006. Divenne disponibile al pubblico generale solo nell'ottobre 2008. La sequenza esatta è: SQS (novembre 2004, primo servizio AWS pubblico), S3 (marzo 2006), EC2 beta (agosto 2006), EC2 GA (ottobre 2008). Il testo originale, indicando solo il 2006 come data di lancio EC2, è corretto ma incompleto.*

Questo modello cambiò radicalmente l'industria IT. Prima del cloud, realizzare un nuovo servizio informatico richiedeva l'acquisto di server fisici, la loro installazione fisica, il cablaggio, la configurazione di rete: processi con tempi di attesa di settimane o mesi. Con il cloud, molte di queste attività divennero operazioni eseguibili da interfaccia web o API in pochi minuti.

Negli anni successivi anche altri grandi attori entrarono nel settore: Microsoft con Azure (2010), Google con Google Cloud Platform, IBM con i propri servizi enterprise, Oracle nel settore database e infrastrutture enterprise.

## 1.3 Definizione formale (NIST)

NIST (National Institute of Standards and Technology) fornisce una definizione autorevole e internazionalmente adottata, pubblicata nella Special Publication 800-145 (2011):

<table style="width:96%;">
<colgroup>
<col style="width: 96%" />
</colgroup>
<tbody>
<tr>
<td><p><strong>Definizione NIST di Cloud Computing (SP 800-145, 2011)</strong></p>
<p>"Cloud computing è un modello per abilitare un accesso di rete ubiquo, conveniente e on-demand a un pool condiviso di risorse computazionali configurabili (es. reti, server, storage, applicazioni e servizi) che possono essere rapidamente fornite e rilasciate con il minimo sforzo gestionale o interazione con il provider." Il modello include: cinque caratteristiche essenziali (on-demand self-service, broad network access, resource pooling, rapid elasticity, measured service); tre modelli di servizio (SaaS, PaaS, IaaS); quattro modelli di deployment (private, community, public, hybrid cloud).</p></td>
</tr>
</tbody>
</table>

# 2. Modelli di Servizio Cloud

Il NIST definisce tre modelli di servizio principali. È importante comprendere come la responsabilità tecnica e operativa si distribuisca tra provider e cliente in ciascuno di essi.

## 2.1 IaaS — Infrastructure as a Service

Nel modello IaaS il provider fornisce l'infrastruttura virtualizzata: macchine virtuali, rete virtuale, storage, firewall virtuali e bilanciatori di carico. Il cliente gestisce il sistema operativo, le applicazioni, la configurazione di sicurezza, gli utenti e gli aggiornamenti software.

IaaS offre il massimo grado di controllo e flessibilità, ma comporta anche la maggiore responsabilità operativa per il cliente. Esempi principali: Amazon EC2, Azure Virtual Machines, Google Compute Engine.

## 2.2 PaaS — Platform as a Service

Nel modello PaaS il provider gestisce anche il sistema operativo, il middleware e il runtime applicativo. Il cliente si concentra principalmente sulle applicazioni. PaaS riduce il carico operativo rispetto a IaaS, ma riduce anche il grado di personalizzazione dell'infrastruttura. Esempi: Azure App Service, Google App Engine, AWS Elastic Beanstalk.

## 2.3 SaaS — Software as a Service

Nel modello SaaS il servizio completo è erogato dal provider; l'utente utilizza direttamente l'applicazione tramite browser o client, senza dover gestire nessun livello infrastrutturale. Il cliente mantiene responsabilità principalmente su utenti, accessi e dati caricati. Esempi: Microsoft 365, Google Workspace, Salesforce.

## 2.4 Modelli aggiuntivi: FaaS e DBaaS

Oltre ai tre modelli fondamentali NIST, il settore ha sviluppato ulteriori specializzazioni.

**FaaS / Serverless:** il codice viene eseguito in risposta a eventi, senza che il cliente gestisca server o runtime. Il provider scala automaticamente le risorse. Esempi: AWS Lambda (2014), Azure Functions. Il termine "serverless" è improprio dal punto di vista tecnico: i server esistono, ma sono completamente astratti dal cliente.

**DBaaS — Database as a Service:** il provider gestisce direttamente il database, inclusi patching, backup e alta disponibilità. Esempi: Amazon RDS, Amazon Aurora, Azure SQL Database, Google Cloud Spanner.

# 3. Caratteristiche di un Cloud Service Provider (CSP)

Scegliere un Cloud Service Provider significa, in ambito enterprise, delegare una parte importante dell'infrastruttura IT a una terza parte. Questo richiede una valutazione approfondita di aspetti tecnici, organizzativi e operativi, non limitata al prezzo o alla notorietà commerciale.

## 3.1 Elevata disponibilità

La disponibilità indica la percentuale di tempo in cui un sistema rimane operativo su base annua. I grandi CSP progettano le proprie infrastrutture per minimizzare interruzioni, downtime, singoli punti di guasto e manutenzioni invasive, distribuendo i servizi su cluster, più host fisici, più reti e più sistemi di storage.

| **Disponibilità** | **Downtime annuo teorico** | **Definizione comune** |
|-------------------|----------------------------|------------------------|
| 99%               | ~3,65 giorni               | Due nines              |
| 99,9%             | ~8,76 ore                  | Three nines            |
| 99,99%            | ~52,6 minuti               | Four nines             |
| 99,999%           | ~5,26 minuti               | Five nines             |

In ambito cloud enterprise si mira generalmente ad almeno "four nines" (99,99%). Da notare che questa disponibilità riguarda il servizio cloud specifico come dichiarato dallo SLA, non necessariamente l'intera applicazione del cliente (che dipende da molteplici componenti).

## 3.2 Ridondanza geografica

I principali CSP possiedono data center distribuiti in numerose regioni del mondo, consentendo di ridurre la latenza, aumentare la resilienza, garantire continuità operativa e limitare gli effetti di guasti regionali. Se una region diventasse indisponibile (guasti elettrici, incendi, eventi naturali, errori operativi), il traffico può essere reindirizzato verso un'altra area geografica.

## 3.3 Sicurezza logica e multi-tenancy

Nel cloud la sicurezza non riguarda soltanto gli edifici fisici. I CSP implementano normalmente segmentazione delle reti, isolamento tra tenant, sistemi IDS/IPS, firewall distribuiti, sistemi anti-DDoS, controllo accessi IAM, cifratura dei dati, logging centralizzato e autenticazione multifattore.

Un aspetto critico è la multi-tenancy: nel cloud pubblico più clienti condividono la stessa infrastruttura fisica. Il provider garantisce che i dati di ciascun cliente siano isolati, le reti separate e le macchine virtuali non interferiscano tra loro. L'isolamento avviene principalmente a livello di hypervisor e di rete virtuale.

## 3.4 Logging e monitoraggio

I grandi CSP investono enormemente nel monitoraggio continuo. I log registrano accessi, errori, modifiche configurative, eventi di rete, tentativi di autenticazione e traffico anomalo. Questi sistemi abilitano rilevazione di incidenti, troubleshooting, auditing, analisi forense e compliance normativa. Esempi di servizi dedicati: AWS CloudTrail (audit API), Azure Monitor, Google Cloud Logging.

## 3.5 Compliance normativa

Molte organizzazioni operano in settori regolamentati (sanitario, bancario, governativo, pagamenti elettronici). Il provider cloud deve dimostrare conformità a standard specifici. Tra i principali:

> **•** GDPR (Regolamento UE 2016/679): obbligatorio per il trattamento di dati personali di cittadini UE, applicabile a qualsiasi organizzazione che processi tali dati indipendentemente dalla localizzazione.
>
> **•** PCI DSS: standard obbligatorio per organizzazioni che gestiscono dati di carte di pagamento.
>
> **•** HIPAA: normativa statunitense per la protezione dei dati sanitari.
>
> **•** DORA (Digital Operational Resilience Act, Reg. UE 2022/2554): entrata in vigore nel gennaio 2025, obbligatoria per il settore finanziario europeo, include requisiti specifici per i fornitori cloud critici (ICT third-party risk management).

*Importante: la compliance del provider non trasferisce automaticamente la compliance al cliente. Il cliente rimane responsabile della corretta configurazione dei servizi e delle proprie pratiche di trattamento dei dati.*

## 3.6 Disaster Recovery

Per disaster recovery si intende la capacità di ripristinare servizi e dati dopo eventi gravi (guasti catastrofici, ransomware, errori umani, incendi, perdita di data center). I CSP forniscono strumenti per replica geografica, snapshot, backup automatici, failover e ripristino rapido. Tuttavia il disaster recovery deve essere progettato anche dal cliente: molti incidenti cloud derivano da cancellazioni accidentali, configurazioni errate, errori IAM o assenza di backup indipendenti.

## 3.7 SLA — Service Level Agreement

Uno SLA è un contratto tecnico-operativo che definisce il livello di servizio garantito dal provider. Gli SLA descrivono disponibilità minima, tempi di risposta (RTO - Recovery Time Objective), obiettivi di punto di ripristino (RPO - Recovery Point Objective), responsabilità ed eventuali compensazioni economiche (crediti di servizio). Gli SLA variano significativamente tra servizi diversi dello stesso provider: ad esempio AWS EC2 garantisce 99,99% per distribuzioni multi-AZ, mentre altri servizi possono avere SLA differenti.

# 4. Tier dei Data Center (Uptime Institute)

I data center vengono classificati secondo livelli chiamati Tier, sistema introdotto dall'Uptime Institute negli anni Novanta e divenuto lo standard internazionale de facto per la valutazione della resilienza infrastrutturale. I Tier sono progressivi: ogni livello superiore include tutti i requisiti di quelli inferiori. La denominazione ufficiale utilizza numeri romani (Tier I, II, III, IV).

| **Tier** | **Denominazione ufficiale** | **Uptime garantito** | **Downtime max/anno** | **Caratteristica chiave** |
|----|----|----|----|----|
| I | Basic Capacity | 99,671% | ~28,8 ore | Singolo percorso di distribuzione, nessuna ridondanza |
| II | Redundant Capacity | 99,741% | ~22 ore | Componenti ridondati (UPS, generatori), singolo percorso |
| III | Concurrently Maintainable | 99,982% | ~1,6 ore | Manutenzione senza downtime; N+1 ridondanza |
| IV | Fault Tolerant | 99,995% | ~26,3 minuti | Guasto di componenti non interrompe i servizi; 2N ridondanza |

Una distinzione importante: il Tier III garantisce la manutenzione concorrente (nessun downtime pianificato), ma non è completamente fault tolerant: eventi non pianificati possono ancora causare interruzioni. Il Tier IV aggiunge la fault tolerance completa, richiedendo infrastruttura doppiamente ridondante (2N). I costi operativi di un Tier IV sono tipicamente il doppio rispetto a un Tier III.

# 5. Certificazioni dei Cloud Provider

Le certificazioni dimostrano che il provider ha implementato controlli organizzativi e tecnici verificati da enti indipendenti accreditati. Sono fondamentali nella valutazione di un CSP, in particolare in contesti enterprise e regolamentati.

| **Certificazione** | **Ente emittente** | **Ambito** |
|----|----|----|
| ISO/IEC 27001 | ISO/IEC | Information Security Management System (ISMS): gestione strutturata della sicurezza delle informazioni |
| ISO 22301 | ISO | Business Continuity Management: capacità di continuare a operare durante crisi |
| ISO/IEC 27017 | ISO/IEC | Estensione di 27001 al contesto cloud: controlli di sicurezza specifici per i servizi cloud |
| ISO/IEC 27018 | ISO/IEC | Protezione dei dati personali nel cloud pubblico; rilevante per conformità GDPR |
| SOC 2 Type II | AICPA | Verifica continuativa (minimo 6 mesi) di sicurezza, disponibilità, integrità, riservatezza e privacy |
| SOC 1 | AICPA | Controlli rilevanti per il reporting finanziario del cliente |
| PCI DSS | PCI SSC | Standard per il trattamento di dati di carte di pagamento |
| CSA STAR | Cloud Security Alliance | Framework specifico per la sicurezza cloud, disponibile in livelli (self-assessment, attestation, certification) |
| FedRAMP | US Government | Autorizzazione federale USA per servizi cloud usati da agenzie governative |

# 6. Sicurezza Fisica del Cloud

Senza sicurezza fisica, nessuna misura di sicurezza logica sarebbe pienamente efficace. I grandi data center cloud sono tra le infrastrutture fisiche più protette al mondo. Molti CSP non divulgano l'esatta posizione dei propri siti critici.

Gli elementi principali della sicurezza fisica includono:

> **•** Controllo degli accessi: badge, smart card, biometria (impronte digitali, scanner dell'iride), con più livelli di autenticazione consecutivi e registrazione di ogni accesso.
>
> **•** Videosorveglianza e presidio: monitoraggio 24/7 tramite telecamere, sensori di movimento e sistemi anti-intrusione. In alcune strutture è presente personale di sicurezza armato.
>
> **•** Continuità elettrica: sistemi UPS (Uninterruptible Power Supply), batterie industriali, generatori diesel con riserve di carburante, linee elettriche multiple e alimentazione da fonti indipendenti.
>
> **•** Controllo ambientale: sistemi di raffreddamento avanzati (CRAC, raffreddamento a liquido nei data center moderni), controllo dell'umidità (tipicamente 40-60% di umidità relativa), monitoraggio termico continuo e gestione dei flussi d'aria (hot aisle / cold aisle containment).
>
> **•** Distruzione sicura dei supporti: quando un disco si guasta o viene dismesso, procedure di distruzione fisica (triturazione), degaussing o sovrascrittura certificata garantiscono che i dati non siano recuperabili. I principali CSP documentano queste procedure nelle proprie policy di sicurezza.

# 7. Shared Responsibility Model (SRM)

Uno dei concetti più importanti del cloud computing è lo Shared Responsibility Model, formalizzato e pubblicato da tutti i principali CSP (AWS, Microsoft Azure, Google Cloud).

<table style="width:96%;">
<colgroup>
<col style="width: 96%" />
</colgroup>
<tbody>
<tr>
<td><p><strong>Principio fondamentale del SRM</strong></p>
<p>Il provider protegge la sicurezza del cloud (security OF the cloud): data center fisici, rete hardware, hypervisor, alimentazione, backbone infrastrutturale. Il cliente protegge la sicurezza nel cloud (security IN the cloud): dati, applicazioni, configurazioni, identità e accessi, patching del sistema operativo (in IaaS), cifratura applicativa, backup. Questa distinzione è formalmente stabilita da AWS, Microsoft e Google nelle proprie documentazioni ufficiali.</p></td>
</tr>
</tbody>
</table>

La distribuzione delle responsabilità varia in funzione del modello di servizio adottato:

| **Responsabilità** | **On-Premise** | **IaaS** | **PaaS** | **SaaS** |
|----|----|----|----|----|
| Dati e classificazione | Cliente | Cliente | Cliente | Cliente |
| Identità e accessi (IAM) | Cliente | Cliente | Condivisa | Condivisa |
| Applicazioni | Cliente | Cliente | Cliente/Condivisa | Provider |
| Sistema Operativo | Cliente | Cliente | Provider | Provider |
| Middleware / Runtime | Cliente | Cliente | Provider | Provider |
| Virtualizzazione / Hypervisor | Cliente | Provider | Provider | Provider |
| Storage fisico | Cliente | Provider | Provider | Provider |
| Rete fisica | Cliente | Provider | Provider | Provider |
| Data center fisico | Cliente | Provider | Provider | Provider |

Molti incidenti cloud reali derivano da errori del cliente, non da vulnerabilità del provider. Esempi comuni: bucket storage esposti accidentalmente in modalità pubblica, password deboli o riutilizzate, firewall troppo permissivi, assenza di MFA, chiavi API o credenziali hardcoded nel codice sorgente.

# 8. Criteri di Scelta di un Servizio Cloud

La scelta di un Cloud Service Provider richiede una valutazione multidimensionale che coinvolge aspetti tecnici, normativi, economici e strategici. In ambito enterprise questa scelta ha implicazioni a lungo termine difficilmente reversibili.

## 8.1 Requisiti di sicurezza e compliance

Il livello di protezione offerto deve essere valutato in base al profilo di rischio dell'organizzazione: cifratura dei dati (at rest e in transit), gestione IAM, protezione DDoS, segregazione dei tenant, sicurezza API. La compliance normativa (GDPR, DORA, HIPAA, PCI DSS) deve essere verificata formalmente attraverso certificazioni terze e audit.

## 8.2 Modello economico e FinOps

Il cloud introduce il modello pay-as-you-go, eliminando grandi investimenti iniziali in hardware. Tuttavia può generare costi imprevisti: macchine virtuali dimenticate attive, storage non gestito, traffico in uscita (egress cost), backup accumulati, scaling automatico mal configurato. La gestione economica dell'infrastruttura cloud è oggi una disciplina specifica denominata FinOps (Financial Operations), con pratiche formalizzate dalla FinOps Foundation.

## 8.3 Latenza e distribuzione geografica

Applicazioni real-time (videoconferenze, trading finanziario, sistemi industriali, gaming) sono sensibili alla latenza. È importante scegliere region cloud vicine agli utenti principali. In alcuni casi si adottano architetture ibride (parte on-premise, parte cloud) o architetture edge computing per ridurre latenza e traffico.

## 8.4 Vendor lock-in

L'adozione massiva di servizi proprietari specifici di un provider (database proprietari, API specifiche, sistemi serverless non standard) può rendere molto difficile la migrazione verso altri provider o verso ambienti on-premise. Strategie di mitigazione: uso preferenziale di standard aperti, architetture basate su container (Kubernetes), Infrastructure as Code portabile.

## 8.5 Scalabilità e right-sizing

Il cloud consente di aumentare o ridurre le risorse rapidamente. Un errore comune è il sovradimensionamento: acquistare risorse in eccesso per sicurezza, pratica tipica del mondo on-premise, nel cloud può generare costi elevati. Il right-sizing consiste nel dimensionare correttamente le risorse e nel monitorare il comportamento reale per ottimizzare nel tempo.

# 9. Region, Availability Zone e Ridondanza

La distribuzione geografica dell'infrastruttura è uno degli aspetti più innovativi e distintivi del cloud moderno.

## 9.1 Region

Una region rappresenta un'area geografica cloud separata, dotata di infrastrutture indipendenti. Esempi: eu-south-1 (Milano), eu-central-1 (Francoforte), ap-northeast-1 (Tokyo), us-east-1 (N. Virginia). La scelta della region influenza latenza, compliance (localizzazione dei dati), costi e resilienza. Non tutti i servizi cloud sono disponibili in tutte le region.

## 9.2 Availability Zone (AZ)

All'interno di ogni region esistono più Availability Zone, ciascuna costituita da uno o più data center fisicamente separati con alimentazione indipendente, networking indipendente e raffreddamento separato. Le AZ sono progettate per isolare i guasti (fault isolation): un incendio, un blackout o un problema di raffreddamento in una AZ non dovrebbe propagarsi alle altre.

Le AZ all'interno di una stessa region sono collegate da reti a bassa latenza (generalmente \< 2 ms), consentendo la replica sincrona dei dati. La distanza fisica tra AZ è intenzionalmente sufficiente per prevenire la propagazione di eventi fisici (es. AWS mantiene distanze di decine di km tra AZ della stessa region).

## 9.3 Architetture multi-AZ e multi-region

Le applicazioni enterprise vengono tipicamente distribuite su più AZ per garantire alta disponibilità e failover automatico. Le organizzazioni con requisiti di resilienza più elevati adottano distribuzioni multi-region, che proteggono contro guasti regionali completi, problemi geopolitici e interruzioni massive. Questo approccio richiede però una progettazione attenta, in particolare per la coerenza dei dati.

# 10. Continuità Operativa nel Cloud

La continuità operativa (Business Continuity) nel cloud non riguarda solo il backup dei dati, ma la capacità complessiva dell'organizzazione di continuare a operare durante incidenti e guasti. Due metriche fondamentali:

> **•** RTO (Recovery Time Objective): il tempo massimo accettabile per ripristinare un servizio dopo un'interruzione.
>
> **•** RPO (Recovery Point Objective): la quantità massima accettabile di dati che possono andare perduti, espressa in tempo (es. «non più di 1 ora di dati»).

Gli strumenti principali includono: replica geografica dei dati tra AZ o region, backup periodici (con attenzione: nel cloud il backup non è automatico per tutti i servizi), failover automatico verso sistemi alternativi, load balancing per distribuire il traffico tra istanze multiple, e deployment multi-region per le organizzazioni con requisiti più stringenti.

*Errore frequente: molti utenti credono erroneamente che l'essere nel cloud implichi automaticamente il backup dei dati. In realtà, la presenza e la configurazione di backup dipendono dal servizio specifico e dalla configurazione scelta. Il cliente deve progettare esplicitamente la strategia di backup.*

# 11. Rischi nell'Adozione del Cloud

Nonostante i numerosi vantaggi, il cloud introduce rischi specifici che devono essere compresi e gestiti attivamente.

| **Categoria di rischio** | **Descrizione** | **Mitigazione tipica** |
|----|----|----|
| Multi-tenancy | Più clienti condividono la stessa infrastruttura fisica; teoricamente vulnerabilità dell'hypervisor o errori di segregazione potrebbero esporre dati tra tenant. | Verifica dell'isolamento del provider, cloud privato o dedicato per carichi critici |
| Vendor lock-in | Dipendenza da API, database e servizi proprietari specifici del provider. | Standard aperti, container, IaC portabile, multi-cloud |
| Rischio economico | Costi imprevisti da risorse non gestite, egress traffic, scaling mal configurato. | FinOps, budget alert, cost governance |
| Misconfigurazione | La principale causa di incidenti cloud: bucket S3 pubblici, permessi IAM eccessivi, firewall aperti. | IaC con review, CSPM tools, least privilege |
| Dipendenza dal provider | Downtime del CSP impatta tutti i clienti. Incidenti AWS (es. 2021), Azure (vari) hanno avuto impatti globali. | Multi-cloud, architetture ibride, SLA review |
| Rischio reputazionale | Data breach o downtime prolungati impattano la fiducia di clienti e partner. | DR planning, comunicazione trasparente, incident response |

# 12. Esempio Applicativo: Architettura AWS

A titolo illustrativo, consideriamo un'architettura cloud elementare su AWS composta da un server applicativo, un database relazionale gestito, un firewall virtuale e una rete privata virtuale.

## 12.1 VPC — Virtual Private Cloud

AWS utilizza il concetto di VPC (Virtual Private Cloud): una rete virtuale privata isolata logicamente all'interno dell'infrastruttura cloud. Il VPC consente di definire subnet, indirizzi IP (tipicamente in spazi RFC 1918: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16), tabelle di routing, gateway Internet, firewall virtuali e connessioni VPN. L'equivalente in Azure è la VNet (Virtual Network); in Google Cloud è la VPC (con semantica globale, non regionale come in AWS).

Una segmentazione tipica in subnet:

> **•** 10.0.1.0/24 → subnet pubblica (frontend, load balancer)
>
> **•** 10.0.2.0/24 → subnet privata applicativa (server applicativi)
>
> **•** 10.0.3.0/24 → subnet privata dati (database, storage)

## 12.2 Amazon EC2

EC2 (Elastic Compute Cloud) consente di creare macchine virtuali (istanze) in pochi minuti, scegliendo sistema operativo, tipo di istanza (configurazione CPU/RAM/rete), storage e configurazione di rete. Le istanze Linux vengono amministrate via SSH con autenticazione a chiave pubblica/privata. Lo storage persistente è fornito da EBS (Elastic Block Store), introdotto nel 2008, che si comporta come un disco virtuale indipendente dal ciclo di vita dell'istanza.

## 12.3 Amazon RDS

RDS (Relational Database Service) è un database managed, in cui il provider gestisce hardware, sistema operativo, patching di basso livello e infrastruttura di replica. L'utente interagisce con il motore di database (PostgreSQL, MySQL, MariaDB, SQL Server, Oracle, o Amazon Aurora) tramite le normali interfacce SQL. RDS supporta deployment multi-AZ con failover automatico e backup automatici configurabili. Il database deve essere posizionato in subnet private, senza IP pubblico, accessibile solo dal server applicativo.

## 12.4 Security Groups e Network ACL

I Security Groups sono firewall stateful associati alle istanze (o ad altri servizi). Consentono di definire regole in ingresso (inbound) e in uscita (outbound) per traffico specifico. A differenza di un firewall tradizionale, le regole dei Security Groups sono permissive: tutto il traffico non esplicitamente consentito è negato. Le Network ACL operano invece a livello subnet, sono stateless (le risposte devono essere esplicitamente autorizzate) e consentono anche regole di negazione esplicita.

| **Porta** | **Protocollo** | **Uso** | **Accesso consigliato** |
|----|----|----|----|
| 22 | TCP | SSH | Solo IP amministrativi (non 0.0.0.0/0) |
| 80 | TCP | HTTP | Pubblico (o redirect a HTTPS) |
| 443 | TCP | HTTPS | Pubblico |
| 3306 | TCP | MySQL/MariaDB | Solo dal server applicativo |
| 5432 | TCP | PostgreSQL | Solo dal server applicativo |

## 12.5 IP pubblici ed Elastic IP

Ogni istanza EC2 può avere un IP privato (utilizzato per comunicazioni interne al VPC, es. 10.0.1.10) e, opzionalmente, un IP pubblico. Gli IP pubblici dinamici cambiano a ogni riavvio dell'istanza; gli Elastic IP sono IP pubblici statici permanenti che facilitano DNS e servizi pubblici. Una buona pratica: il database non dovrebbe mai avere IP pubblico; SSH dovrebbe essere limitato a IP specifici; le porte inutilizzate devono rimanere chiuse.

# 13. Storage Cloud: Tier e Archiviazione di Lungo Termine

Nel cloud moderno lo storage è organizzato in livelli (tier), ciascuno rappresenta un compromesso tra costo, velocità di accesso e frequenza d'uso prevista.

| **Tier** | **Uso tipico** | **Latenza accesso** | **Costo relativo** | **Esempi AWS** |
|----|----|----|----|----|
| Hot (Standard) | Dati frequentemente accessibili: applicazioni live, siti web, database | Millisecondi | Elevato | S3 Standard, EBS gp3 |
| Cool / Infrequent Access | Dati acceduti occasionalmente: backup recenti, documentazione | Millisecondi | Medio | S3 Standard-IA, S3 One Zone-IA |
| Archive (Glacier) | Dati raramente accessibili: archivi storici, log a lungo termine, retention legale | Minuti–ore | Molto basso | S3 Glacier, S3 Glacier Deep Archive |

Amazon S3 Glacier e S3 Glacier Deep Archive sono progettati per archiviazione a lungo termine con costi molto ridotti, in cambio di tempi di recupero più lunghi (da minuti per Glacier Instant Retrieval a 12-48 ore per Glacier Deep Archive). AWS dichiara una durabilità di 99,999999999% (11 nines) per S3, ottenuta tramite replica su più facility.

Le lifecycle policies consentono di automatizzare il movimento dei dati tra tier in base all'età degli oggetti, ottimizzando i costi senza intervento manuale. Esempio: dopo 30 giorni → S3 Standard-IA; dopo 90 giorni → S3 Glacier; dopo 7 anni → eliminazione.

# 14. Documentazione di Sicurezza dei Principali CSP

I principali provider pubblicano documentazione tecnica e normativa accessibile pubblicamente, fondamentale per amministratori, auditor, consulenti e team di cybersecurity.

> **•** Microsoft Azure — Documentazione sicurezza: https://learn.microsoft.com/en-us/azure/security/ \| Trust Center: https://www.microsoft.com/trust-center
>
> **•** Amazon Web Services — Security Documentation: https://docs.aws.amazon.com/security/ \| Compliance Programs: https://aws.amazon.com/compliance/programs/
>
> **•** Google Cloud Platform — Security Overview: https://cloud.google.com/security \| Compliance: https://cloud.google.com/compliance

# 15. Considerazioni Finali

Il cloud computing rappresenta oggi una componente fondamentale dell'infrastruttura IT moderna. Comprenderne i modelli di servizio, le responsabilità condivise, i meccanismi di sicurezza, ridondanza e continuità operativa è essenziale per progettare sistemi affidabili e sicuri.

Il cloud non elimina i problemi di sicurezza: sposta alcune responsabilità sul provider, ma richiede comunque solide competenze tecniche e organizzative da parte del cliente. La principale causa di incidenti cloud documentati non sono le vulnerabilità dei provider, ma le misconfigurazione e gli errori operativi del cliente.

Le competenze fondamentali richieste includono: networking virtuale, IAM, sicurezza cloud, automazione (Infrastructure as Code), resilienza e disaster recovery, monitoraggio, gestione dei costi (FinOps) e compliance normativa.

# Bibliografia

Di seguito le fonti primarie e secondarie di riferimento per gli argomenti trattati in questa lezione. Le fonti normative e gli standard citati sono da considerarsi obbligatorie come letture di approfondimento.

## Standard e Documenti Normativi

**\[1\]** Mell, P. e Grance, T. (2011). The NIST Definition of Cloud Computing. NIST Special Publication 800-145. National Institute of Standards and Technology, Gaithersburg, MD. https://doi.org/10.6028/NIST.SP.800-145

**\[2\]** Simmon, E. (2018). Evaluation of Cloud Computing Services Based on NIST SP 800-145. NIST Special Publication 500-322. National Institute of Standards and Technology. https://doi.org/10.6028/NIST.SP.500-322

**\[3\]** Uptime Institute (2022). Tier Standard: Topology. Uptime Institute LLC, New York. Disponibile su: https://uptimeinstitute.com/tiers

**\[4\]** ISO/IEC 27001:2022. Information Security, Cybersecurity and Privacy Protection — Information Security Management Systems — Requirements. International Organization for Standardization, Geneva.

**\[5\]** ISO/IEC 27017:2015. Code of Practice for Information Security Controls Based on ISO/IEC 27002 for Cloud Services. International Organization for Standardization, Geneva.

**\[6\]** ISO/IEC 27018:2019. Code of Practice for Protection of Personally Identifiable Information (PII) in Public Clouds Acting as PII Processors. International Organization for Standardization, Geneva.

**\[7\]** ISO 22301:2019. Security and Resilience — Business Continuity Management Systems — Requirements. International Organization for Standardization, Geneva.

**\[8\]** Parlamento Europeo e Consiglio dell'UE (2022). Regolamento (UE) 2022/2554 relativo alla resilienza operativa digitale per il settore finanziario (DORA). Gazzetta Ufficiale dell'Unione Europea, L 333, 27 dicembre 2022.

**\[9\]** Parlamento Europeo e Consiglio dell'UE (2016). Regolamento (UE) 2016/679 (GDPR). Gazzetta Ufficiale dell'Unione Europea, L 119, 4 maggio 2016.

## Documentazione Ufficiale dei Provider

**\[10\]** Amazon Web Services (2023). AWS Shared Responsibility Model. https://aws.amazon.com/compliance/shared-responsibility-model/

**\[11\]** Microsoft Azure (2024). Shared Responsibility in the Cloud. Microsoft Learn. https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility

**\[12\]** Google Cloud (2024). Shared Responsibility and Shared Fate on Google Cloud. Google Cloud Architecture Center. https://cloud.google.com/architecture/framework/security/shared-responsibility-shared-fate

**\[13\]** Cloud Security Alliance (2017). Security Guidance for Critical Areas of Focus in Cloud Computing v4.0. Cloud Security Alliance, Seattle, WA. https://cloudsecurityalliance.org/research/guidance

## Testi di Riferimento

**\[14\]** Armbrust, M. et al. (2010). A View of Cloud Computing. Communications of the ACM, 53(4), pp. 50–58. https://doi.org/10.1145/1721654.1721672

**\[15\]** Mather, T., Kumaraswamy, S. e Latif, S. (2009). Cloud Security and Privacy: An Enterprise Perspective on Risks and Compliance. O'Reilly Media, Sebastopol, CA. ISBN: 978-0-596-80276-9.

**\[16\]** Velte, T., Velte, A. e Elsenpeter, R. (2010). Cloud Computing: A Practical Approach. McGraw-Hill, New York. ISBN: 978-0-07-162694-1.

**\[17\]** Fehling, C., Leymann, F., Retter, R., Schupeck, W. e Arbitter, P. (2014). Cloud Computing Patterns: Fundamentals to Design, Build, and Manage Cloud Applications. Springer, Vienna. ISBN: 978-3-7091-1567-1.

## Fonti Storiche e di Approfondimento

**\[18\]** McCarthy, J. (1961). Discorso al MIT Centennial, Massachusetts Institute of Technology, Cambridge, MA. Citato in: MIT Technology Review, ottobre 2011. https://www.technologyreview.com/2011/10/03/190237/the-cloud-imperative/

**\[19\]** Licklider, J.C.R. (1963). Memorandum for Members and Affiliates of the Intergalactic Computer Network. Advanced Research Projects Agency (ARPA), Washington DC.

**\[20\]** Miller, R. (2021). How Amazon EC2 grew from a notion into a foundational element of cloud computing. TechCrunch, 28 agosto 2021. https://techcrunch.com/2021/08/28/how-amazon-ec2-grew-from-a-notion-into-a-foundational-element-of-cloud-computing/

**\[21\]** Amazon Web Services (2021). Happy 15th Birthday Amazon EC2. AWS News Blog. https://aws.amazon.com/blogs/aws/happy-15th-birthday-amazon-ec2/

## Strumenti e Framework Operativi

**\[22\]** FinOps Foundation (2024). FinOps Framework. https://www.finops.org/framework/

**\[23\]** Cloud Security Alliance (2019). Cloud Controls Matrix (CCM) v4. https://cloudsecurityalliance.org/research/cloud-controls-matrix/

**\[24\]** ENISA — European Union Agency for Cybersecurity (2023). Cloud Cybersecurity Market Analysis. ENISA, Heraklion. https://www.enisa.europa.eu
