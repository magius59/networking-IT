**Lezione 9**

Virtualizzazione, macchine virtuali e automazione degli ambienti di rete

# 1. Origine ed evoluzione della virtualizzazione

La virtualizzazione consente di eseguire più ambienti logici separati sulla stessa macchina fisica. L'idea non nasce con i personal computer moderni: già sui sistemi mainframe IBM degli anni Sessanta erano presenti forme di esecuzione virtualizzata. Il sistema CP-67, introdotto nel 1967, e il suo successore VM/370 (1972) permettevano a un mainframe di eseguire più ambienti indipendenti con piena virtualizzazione dell'hardware. Quella tradizione rimase confinata al mondo mainframe e non si propagò direttamente ai PC.

Con l'evoluzione dell'informatica si è passati dai mainframe ai minicomputer, poi ai personal computer e infine ai server x86. In origine, molte organizzazioni utilizzavano server fisici dedicati a singole funzioni: un server per il dominio, uno per il DNS, uno per il file server, uno per il web server, uno per il database e così via.

Questo modello aveva diversi limiti:

- basso utilizzo medio delle risorse (un server dedicato al DNS impiegava spesso meno del 5-10% della CPU);

- consumo elevato di energia;

- costi hardware superiori;

- gestione complessa;

- difficoltà nel ripristino rapido dei sistemi.

Il revival della virtualizzazione su architettura x86 avvenne negli anni Novanta con VMware (fondata nel 1998), che risolse con tecniche software — in particolare la binary translation — i limiti dell'architettura x86, che non era originariamente progettata per essere virtualizzata. Una svolta decisiva arrivò nel 2005-2006 con l'introduzione delle estensioni hardware Intel VT-x e AMD-V, che permisero all'hypervisor di intercettare le istruzioni privilegiate direttamente in hardware, rendendo la virtualizzazione molto più efficiente e portando a un'adozione di massa.

# 2. Concetto di macchina virtuale

Una macchina virtuale è un ambiente software che si comporta come un computer indipendente. Al suo interno possono essere installati un sistema operativo, applicazioni, servizi di rete e strumenti di amministrazione.

È importante distinguere tra emulazione e virtualizzazione. Un emulatore (come QEMU in modalità software pura) ricrea l'hardware traducendo ogni singola istruzione del processore guest: offre grande flessibilità ma a costo di prestazioni significativamente ridotte. Un hypervisor con supporto hardware, invece, esegue direttamente la maggior parte delle istruzioni della CPU guest sull'hardware fisico, intercettando solo quelle privilegiate. Questa distinzione spiega perché la virtualizzazione hardware sia incomparabilmente più veloce dell'emulazione pura.

La macchina virtuale vede risorse virtualizzate, tra cui:

- CPU virtuali (vCPU);

- memoria RAM assegnata;

- dischi virtuali;

- schede di rete virtuali;

- controller e periferiche emulate.

In questa architettura si usano due termini fondamentali: il sistema operativo installato all'interno della VM è chiamato guest OS; il sistema su cui opera l'hypervisor è chiamato host. Il componente che gestisce l'astrazione tra guest e hardware fisico si chiama hypervisor (o Virtual Machine Monitor, VMM).

Il meccanismo tecnico sottostante si basa sui ring di protezione della CPU. In architettura x86, il ring 0 è il livello di privilegio massimo (kernel), mentre il ring 3 è quello degli applicativi utente. L'hypervisor sfrutta le estensioni hardware (VMX root mode per Intel, SVM per AMD) per intercettare in modo sicuro le operazioni privilegiate eseguite dal guest, senza che questo possa accedere direttamente all'hardware.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><strong>Ring di protezione, tipi di istruzione e modalità del sistema operativo</strong></td>
</tr>
<tr>
<td><strong>Ring di protezione della CPU (architettura x86)</strong></td>
</tr>
<tr>
<td><p>L'architettura x86 definisce quattro livelli di privilegio numerati da 0 (massimo) a 3 (minimo). I sistemi operativi moderni usano in pratica solo due di essi:</p>
<ul>
<li><p><strong>Ring 0 (kernel mode):</strong> accesso illimitato all'hardware. Qui opera il kernel del sistema operativo. Può eseguire qualsiasi istruzione, modificare le tabelle di pagina, gestire gli interrupt, accedere a qualsiasi indirizzo fisico.</p></li>
<li><p><strong>Ring 1 e 2:</strong> livelli intermedi, originariamente pensati per driver e sistemi operativi ospitati. Rimasti quasi inutilizzati nella pratica.</p></li>
<li><p><strong>Ring 3 (user mode):</strong> livello in cui operano tutte le applicazioni utente (browser, editor, script). Nessun accesso diretto all'hardware. Ogni tentativo di eseguire un'istruzione privilegiata genera un'eccezione che il kernel intercetta.</p></li>
</ul>
<p><strong>Nota:</strong> <em>esiste anche un Ring −2 (System Management Mode, SMM), riservato al firmware BIOS/UEFI. È invisibile al sistema operativo e all'hypervisor, e viene attivato da interrupt hardware speciali (SMI). È potenzialmente vettore di attacchi firmware molto persistenti (SMM rootkit).</em></p></td>
</tr>
<tr>
<td><strong>Tipi di istruzione del processore</strong></td>
</tr>
<tr>
<td><p>Le istruzioni macchina si classificano in base a chi può eseguirle e a come si comportano in un contesto virtualizzato.</p>
<ul>
<li><p><strong>Istruzioni non privilegiate (normali):</strong> aritmetiche, logiche, di confronto, di salto, accesso alla memoria utente. Eseguibili da qualunque ring. In una VM con supporto hardware, vengono eseguite direttamente sulla CPU fisica senza alcuna intercettazione — è questo il motivo per cui la virtualizzazione hardware è molto più veloce dell'emulazione software. Esempi: ADD, MOV, JMP, CMP, PUSH, POP.</p></li>
<li><p><strong>Istruzioni privilegiate:</strong> modificano lo stato globale della macchina (tabelle di pagina, registri di controllo, interrupt, I/O diretto). Riservate al Ring 0. Se un'applicazione in Ring 3 tenta di eseguirle, la CPU genera un'eccezione di protezione. In una VM, l'hypervisor le intercetta tramite un meccanismo chiamato VM exit, le emula in modo sicuro, e restituisce il controllo alla VM (VM entry). Esempi: CLI/STI (interrupt), LGDT, MOV CR0, IN/OUT, HLT.</p></li>
<li><p><strong>Istruzioni sensibili (il problema storico di x86):</strong> istruzioni che leggono o modificano stato privilegiato, ma che non generano alcuna eccezione se eseguite da Ring 3 — si comportano silenziosamente in modo diverso a seconda del ring in cui vengono eseguite. Questo era il difetto fondamentale dell'architettura x86 che rendeva impossibile la virtualizzazione "classica". Popek e Goldberg (1974) dimostrarono che, per essere virtualizzabile, ogni istruzione sensibile di un'architettura deve essere anche privilegiata — requisito che x86 originariamente non soddisfaceva. Esempi storici: POPF, PUSHF, SGDT, SMSW.</p></li>
<li><p><strong>Istruzioni di virtualizzazione hardware (Intel VT-x / AMD-V, 2005–2006):</strong> risolvono il problema delle istruzioni sensibili introducendo due nuove modalità di esecuzione — VMX root (per l'hypervisor) e VMX non-root (per il guest) — e istruzioni dedicate per gestire le transizioni in modo efficiente senza binary translation. Esempi: VMXON, VMXOFF, VMLAUNCH, VMRESUME.</p></li>
</ul>
<p><strong>Impatto sulle prestazioni:</strong> con la virtualizzazione hardware attiva, il 90–99% delle istruzioni della VM vengono eseguite direttamente sulla CPU fisica. Solo le istruzioni privilegiate causano un VM exit verso l'hypervisor, con un costo di circa 1.000–10.000 cicli di clock ciascuno. QEMU senza KVM deve invece intercettare e tradurre via software ogni istruzione sensibile, con degrado di prestazioni molto più marcato.</p></td>
</tr>
<tr>
<td><strong>Modalità operative di un sistema operativo moderno</strong></td>
</tr>
<tr>
<td><ul>
<li><p><strong>Kernel mode (Ring 0):</strong> il kernel opera con accesso illimitato. Qui girano scheduler, driver, gestione della memoria, filesystem, stack di rete. Una corruzione di memoria in kernel mode può causare il blocco dell'intero sistema (kernel panic su Linux, BSOD su Windows). In una VM, il kernel del guest opera in VMX non-root Ring 0: crede di avere accesso pieno all'hardware, ma l'hypervisor intercetta le operazioni privilegiate.</p></li>
<li><p><strong>User mode (Ring 3):</strong> ogni applicazione utente opera in Ring 3 in uno spazio di indirizzamento virtuale isolato. Non può accedere direttamente all'hardware né alla memoria di altri processi. Se tenta un'operazione non consentita, il kernel termina il processo (Segmentation Fault su Linux, Access Violation su Windows). Questo confina i danni di un programma difettoso o malevolo.</p></li>
<li><p><strong>Transizione Ring 3 → Ring 0 (system call):</strong> l'unico meccanismo legittimo per cui un'applicazione può richiedere al kernel di eseguire operazioni privilegiate per suo conto. L'applicazione invoca una funzione di libreria (es. read(), write(), socket()) che esegue l'istruzione SYSCALL (x86-64) o INT 0x80 (x86 legacy). La CPU cambia automaticamente il ring a 0, salta all'entry point del kernel, esegue l'operazione validando ogni parametro, e restituisce il controllo all'applicazione in Ring 3.</p></li>
<li><p><strong>System Management Mode — SMM (Ring −2):</strong> modalità speciale del processore riservata al firmware BIOS/UEFI. Attivata da un interrupt hardware speciale (SMI). Completamente invisibile al sistema operativo e all'hypervisor. Ha accesso a tutta la memoria fisica, incluse le regioni nascoste al SO. Usata per gestione dell'energia e aggiornamenti del firmware; potenziale vettore di attacchi di tipo SMM rootkit, molto difficili da rilevare.</p></li>
</ul></td>
</tr>
</tbody>
</table>

# 3. Hypervisor di tipo 1 e tipo 2

La classificazione degli hypervisor in tipo 1 e tipo 2 fu formalizzata da Gerald Popek e Robert Goldberg nel 1974, nel saggio *Formal requirements for virtualizable third generation architectures*. È una distinzione concettuale importante, anche se nella pratica moderna alcuni prodotti si collocano in una zona intermedia.

## Hypervisor di tipo 1 (bare-metal)

Un hypervisor di tipo 1 viene installato direttamente sull'hardware fisico. Non richiede un sistema operativo tradizionale sottostante: è esso stesso lo strato che gestisce CPU, memoria, storage e networking. Questa architettura offre prestazioni superiori, maggiore isolamento e minore superficie di attacco rispetto al tipo 2.

Esempi tipici sono:

- VMware ESXi: hypervisor enterprise di riferimento per data center;

- Microsoft Hyper-V: hypervisor di tipo 1 indipendentemente dal contesto di installazione (vedi nota sotto);

- Xen: hypervisor open source, usato storicamente da AWS, che introduce il concetto di paravirtualizzazione;

- KVM (Kernel-based Virtual Machine): modulo del kernel Linux che trasforma il kernel stesso in un hypervisor con accesso diretto all'hardware (vedi nota sotto).

**Nota su Hyper-V:** Hyper-V è un hypervisor di tipo 1. Quando si installa il ruolo Hyper-V su Windows Server o su Windows 10/11 Pro, l'hypervisor viene posizionato sotto il sistema operativo esistente, che diventa la root partition — una VM privilegiata.

**Nota su KVM:** KVM è classificato prevalentemente come hypervisor di tipo 1 perché, una volta caricato il modulo kvm nel kernel Linux, il kernel stesso acquisisce le capacità di un VMM e accede all'hardware tramite le estensioni di virtualizzazione (VT-x/AMD-V). Ogni VM è gestita come processo Linux. Tuttavia, poiché KVM si appoggia all'infrastruttura del kernel host (scheduling, driver), alcuni autori lo collocano in una categoria ibrida. Per la gestione dell'I/O virtuale, KVM si abbina tipicamente a QEMU. Proxmox VE, spesso citato come hypervisor, è in realtà una piattaforma di gestione che usa KVM per le macchine virtuali e LXC per i container.

## 

## Hypervisor di tipo 2 (hosted)

Un hypervisor di tipo 2 viene installato sopra un sistema operativo già esistente, come Windows, Linux o macOS, e funziona come un'applicazione. L'hypervisor negozia l'accesso alle risorse con il sistema operativo host, con conseguente overhead aggiuntivo.

Esempi comuni sono:

- Oracle VirtualBox (cross-platform, open source);

- VMware Workstation (Windows/Linux);

- VMware Fusion (macOS);

- Parallels Desktop (macOS).

Queste soluzioni sono molto utili per laboratori, didattica, testing, cybersecurity training e sviluppo software. Presentano però maggiore superficie di attacco: una vulnerabilità nel sistema operativo host può compromettere anche le VM ospitate.

# 4. Vantaggi della virtualizzazione

La virtualizzazione offre diversi vantaggi tecnici e operativi.

Il primo è il migliore utilizzo delle risorse. Un server fisico moderno può ospitare più macchine virtuali, ciascuna dedicata a una funzione diversa, riducendo il numero di server fisici necessari e il consumo energetico.

Il secondo vantaggio è l'isolamento. Ogni macchina virtuale opera come ambiente separato. Un problema su una VM non dovrebbe compromettere direttamente le altre, salvo errori di configurazione, vulnerabilità dell'hypervisor, o attacchi di tipo VM escape — categoria nella quale un attaccante sfrutta una vulnerabilità dell'hypervisor o dei driver virtuali per uscire dal contesto della VM e accedere all'host o ad altre VM (fenomeno documentato da vulnerabilità reali su VMware, KVM e VirtualBox nel corso degli anni).

Il terzo vantaggio è la gestione degli snapshot. Uno snapshot registra lo stato di una macchina virtuale in un dato momento: disco, RAM e configurazione. Se dopo un aggiornamento, un test o un'analisi malware il sistema diventa instabile, è possibile tornare allo stato precedente in pochi secondi. Questa funzione è particolarmente utile in:

- malware analysis;

- digital forensics;

- penetration testing;

- laboratori universitari;

- test di aggiornamenti;

- simulazioni di ambienti aziendali.

Il quarto vantaggio è la replicabilità. Una macchina virtuale può essere copiata, clonata, esportata e importata su qualsiasi sistema compatibile con lo stesso formato.

Il quinto vantaggio, fondamentale in ambienti enterprise, è la migrazione live (live migration). Una VM in esecuzione può essere spostata da un host fisico a un altro senza interruzione del servizio — tecnologia nota come vMotion in VMware. Questo permette di effettuare manutenzioni hardware senza downtime applicativo ed è uno dei pilastri dell'alta disponibilità e del cloud computing.

Gli hypervisor possono inoltre applicare tecniche di overcommitting delle risorse: assegnare alle VM più RAM o CPU di quante ne esistano fisicamente sull'host, contando sul fatto che non tutte saranno utilizzate al massimo contemporaneamente. Questa pratica, utile per ottimizzare l'uso dell'hardware, introduce rischi di degrado delle prestazioni in caso di picchi di carico e deve essere gestita con attenzione.

# 5. Virtualizzazione e sicurezza

Dal punto di vista della sicurezza informatica, la virtualizzazione è uno strumento fondamentale.

In un laboratorio di malware analysis, il codice malevolo viene eseguito in una macchina virtuale isolata. L'hypervisor può monitorare l'esecuzione, registrare le chiamate di sistema, analizzare il traffico di rete generato dal malware e ripristinare il sistema allo stato pulito tramite snapshot. Strumenti come Cuckoo Sandbox sfruttano questo approccio in modo automatizzato.

Un elemento critico in questo contesto sono le anti-VM techniques: tecniche con le quali il malware rileva di essere eseguito in un ambiente virtualizzato e modifica il proprio comportamento, rimanendo inattivo o alterando le azioni.\
\
I metodi di rilevamento usati dal malware includono: l'istruzione CPUID (che in una VM restituisce valori caratteristici), timing attacks (misurazioni di latenza anomale rispetto all'hardware fisico), la presenza di driver tipici degli hypervisor (vmxnet, VBoxGuest) e artefatti nel registro di sistema di Windows. Chi opera in malware analysis deve conoscere queste tecniche e configurare gli ambienti per minimizzarne la rilevabilità.

In un penetration test, è buona pratica usare ambienti virtualizzati dedicati, per ridurre il rischio di conservare accidentalmente dati, strumenti, credenziali o artefatti relativi a test precedenti. Lo snapshot permette di ripristinare rapidamente le macchine all'inizio del test.

Anche nella digital forensics la virtualizzazione può essere utile, purché l'ambiente sia controllato e documentato. È fondamentale evitare contaminazioni tra analisi diverse e mantenere chiara la separazione tra sistemi, immagini e dati esaminati.

# 6. Networking nelle macchine virtuali

Le macchine virtuali usano schede di rete virtuali (vNIC) collegate a switch virtuali. Uno switch virtuale è un componente software che simula il comportamento di uno switch fisico, operando a livello 2 del modello OSI.

Le modalità di rete principali sono le seguenti. La terminologia può variare tra i diversi hypervisor (VirtualBox, VMware, Hyper-V, KVM/libvirt), ma i concetti sottostanti sono equivalenti.

## Bridge (o Bridged)

La VM si collega alla stessa rete fisica dell'host, riceve un indirizzo IP della stessa subnet e si comporta come un dispositivo indipendente sulla LAN. È utile quando si vuole che la macchina virtuale sia raggiungibile dagli altri sistemi della rete.

## NAT

La VM usa l'indirizzo dell'host per uscire verso Internet o verso la rete esterna. Può navigare, aggiornarsi e stabilire connessioni in uscita, ma di norma non è direttamente raggiungibile dall'esterno. È una modalità comoda e sicura per molte attività di laboratorio.

## NAT Network

Variante del NAT in cui più VM condividono lo stesso circuito NAT e possono comunicare tra loro, rimanendo però isolate dalla rete fisica dell'host. Utile per scenari multi-macchina che richiedono accesso a Internet ma isolamento dall'esterno.

## Host-only

La VM comunica solo con l'host e con eventuali altre VM collegate alla stessa rete virtuale. Non ha accesso diretto alla rete esterna. Utile per laboratori locali isolati.

## Rete interna (Internal Network)

Collega più macchine virtuali tra loro senza esporle alla rete fisica né all'host. L'host non è raggiungibile dalla VM. È la modalità più isolata e viene usata per simulare segmenti di rete aziendali, reti con dominio Active Directory, firewall interni e ambienti vulnerabili.

In ambienti enterprise (VMware vSphere, Proxmox, KVM con Open vSwitch), la gestione di rete è più articolata e include virtual distributed switch, VLAN, trunk port e VLAN tagging — concetti che permettono di riprodurre fedelmente architetture di rete aziendali complesse.

# 7. Dimensionamento delle risorse

Quando si costruisce un laboratorio virtuale, la risorsa che si esaurisce più rapidamente è spesso la RAM. Ogni macchina virtuale richiede una quantità di memoria dedicata. Molti hypervisor riservano inoltre spazio su disco per le strutture interne di ogni VM (ad esempio VMware crea un file .vswp delle dimensioni della RAM assegnata alla VM).

La CPU è importante ma spesso diventa un limite solo quando molte VM eseguono carichi intensivi contemporaneamente. È fondamentale verificare nel BIOS/UEFI che le estensioni di virtualizzazione hardware (Intel VT-x o AMD-V) siano abilitate, poiché non sempre lo sono di default. Per ambienti su server multi-socket è rilevante anche la topologia NUMA (Non-Uniform Memory Access): allocare una VM su core appartenenti a socket diversi può causare degrado significativo delle prestazioni di accesso alla memoria.

Lo storage è altrettanto rilevante. Dischi SSD, e in particolare NVMe, migliorano sensibilmente le prestazioni rispetto ai dischi meccanici, specialmente quando più VM accedono contemporaneamente ai propri dischi virtuali.

Per un laboratorio personale sono quindi consigliabili:

- molta RAM (almeno 16 GB, idealmente 32 GB o più per laboratori multi-macchina);

- disco SSD o NVMe come storage primario;

- CPU multicore con supporto a VT-x/AMD-V;

- buona organizzazione delle reti virtuali.

# 8. Immagini virtuali e formati

Una macchina virtuale può essere distribuita come immagine già pronta, evitando l'installazione manuale del sistema operativo da ISO. I formati principali sono:

- OVF (Open Virtualization Format): standard aperto che descrive la configurazione di una VM in un file XML. OVA (Open Virtualization Appliance) è un archivio tar che contiene il descrittore OVF e uno o più dischi virtuali. I due termini non sono sinonimi: OVF è il formato, OVA è il container. Sono supportati da VMware, VirtualBox e altri hypervisor.

- VMDK (Virtual Machine Disk): formato del disco virtuale usato da VMware. La configurazione della VM è separata, nel file .vmx.

- VDI (Virtual Disk Image): formato usato da VirtualBox.

- VHD/VHDX: formati usati da Microsoft Hyper-V. VHDX supporta dimensioni fino a 64 TB e maggiore resilienza rispetto al VHD.

- QCOW2 (QEMU Copy On Write v2): formato ampiamente usato in ambienti KVM/QEMU e cloud (OpenStack, libvirt). Supporta nativamente snapshot, thin provisioning e cifratura del disco. È il formato di fatto negli ambienti Linux enterprise.

Per esempio, molte distribuzioni orientate alla sicurezza, come Kali Linux, offrono immagini già pronte per diversi hypervisor. L'utente scarica l'immagine, la importa e avvia la macchina senza alcuna installazione manuale.

# 

# 9. Automazione con Vagrant

Vagrant è uno strumento a riga di comando sviluppato da HashiCorp che consente di creare, configurare e gestire macchine virtuali in modo ripetibile e codificato come infrastruttura.

L'idea centrale è il Vagrantfile, un file di testo (sintassi Ruby) che descrive:

- quale immagine (box) usare;

- quale provider usare (VirtualBox, Hyper-V, KVM/libvirt, VMware — vedi nota sotto);

- quanta RAM assegnare;

- quante CPU usare;

- quale rete configurare;

- quali comandi o strumenti di provisioning eseguire al primo avvio.

**Nota sui provider:** Vagrant include il supporto nativo per VirtualBox, Hyper-V e Docker. Il supporto per VMware (Workstation, Fusion, Desktop) è disponibile come plugin aggiuntivo. KVM/libvirt richiede il plugin vagrant-libvirt. La box scaricata è specifica per ogni provider: una box VirtualBox non è compatibile con VMware.

Con Vagrant è possibile scaricare box predefinite dal catalogo pubblico Vagrant Cloud (oggi Vagrant Registry). Un flusso tipico è:

- scelta della box (ad es. ubuntu/jammy64);

- definizione del Vagrantfile;

- comando vagrant up — Vagrant scarica la box, crea la VM e avvia il provisioning;

- accesso con vagrant ssh;

- sospensione con vagrant halt, distruzione con vagrant destroy.

Questo approccio riduce molto il lavoro manuale e permette di ricostruire ambienti identici su qualsiasi macchina in pochi minuti. Vagrant si inserisce nell'ecosistema HashiCorp insieme a Terraform (provisioning dell'infrastruttura cloud) e Packer (creazione di immagini golden).

# 10. Provisioning con Ansible

Ansible è uno strumento di automazione open source usato per configuration management, deployment applicativo e orchestrazione. Permette di descrivere lo stato desiderato di una macchina tramite file YAML chiamati playbook.

Ansible è agentless per architettura: non richiede l'installazione di alcun agente permanente sulla macchina gestita. Questa non è una scelta configurabile, ma la filosofia progettuale che distingue Ansible da strumenti agent-based come Puppet o Chef. Ansible comunica tramite SSH su sistemi Linux/Unix (Python deve essere presente sul nodo gestito) e tramite WinRM (Windows Remote Management) su sistemi Windows.

Un principio fondamentale di Ansible è l'idempotenza: un playbook eseguito più volte produce sempre lo stesso stato finale, senza effetti collaterali indesiderati. Se un pacchetto è già installato, Ansible non lo reinstalla; se un file ha già il contenuto corretto, non lo sovrascrive.

La struttura di un playbook Ansible si organizza in:

- plays: definiscono su quali host agire;

- tasks: le operazioni singole (installare un pacchetto, creare un file, avviare un servizio);

- handlers: azioni eseguite in risposta a cambiamenti (es. riavviare un servizio dopo una modifica alla configurazione);

- roles: strutture riusabili che raggruppano task, handler, variabili e template per una funzione specifica.

Un playbook può, per esempio: installare pacchetti, creare utenti, configurare servizi, impostare hostname, distribuire file, configurare policy, preparare server applicativi e configurare ambienti Active Directory. Ansible Galaxy è il repository pubblico di role predefiniti e condivisi dalla community.

L'uso combinato di Vagrant e Ansible consente di costruire laboratori complessi in modo completamente automatico: Vagrant crea le macchine virtuali, Ansible le configura. Il provisioner Ansible può essere invocato direttamente dal Vagrantfile, specificando il percorso del playbook.

# 11. Laboratori di sicurezza e Active Directory

Un esempio tipico di uso avanzato della virtualizzazione è la creazione di laboratori per studiare Active Directory, Kerberos, attacchi laterali, privilege escalation e tecniche difensive.

Active Directory è rilevante per la sicurezza offensiva perché gestisce l'autenticazione (Kerberos, NTLM), le Group Policy (GPO), la gestione delle identità e i trust tra domini. Attacchi come Pass-the-Hash, Pass-the-Ticket, Kerberoasting, AS-REP Roasting, Golden/Silver Ticket e DCSync emergono direttamente dalle caratteristiche architetturali di Active Directory.

In questi ambienti virtualizzati possono essere presenti:

- domain controller;

- server membri;

- workstation;

- firewall;

- macchine di attacco (es. Kali Linux);

- utenti e gruppi preconfigurati;

- vulnerabilità intenzionali;

- policy di dominio;

- subnet e routing simulati.

Progetti come GOAD (Game of Active Directory) automatizzano la creazione di questi laboratori tramite Vagrant e Ansible, riproducendo ambienti realistici con vulnerabilità intenzionali pronti per l'uso. Questi laboratori permettono di studiare scenari realistici senza intervenire su infrastrutture reali.

# 12. Dal laboratorio locale al cloud

La virtualizzazione è anche alla base del cloud computing. Quando si crea una macchina virtuale su un provider cloud come AWS, Azure o Google Cloud, si sta utilizzando un'infrastruttura virtualizzata gestita dal provider su scala massiva.

Dal punto di vista architetturale, il cloud si declina in modelli di servizio distinti:

- IaaS (Infrastructure as a Service): l'utente gestisce le VM — CPU, RAM, storage, OS. Esempio: AWS EC2, Azure Virtual Machines.

- PaaS (Platform as a Service): l'utente gestisce solo l'applicazione; l'infrastruttura sottostante è trasparente. Esempio: AWS Elastic Beanstalk, Google App Engine.

- SaaS (Software as a Service): l'utente usa solo il software come servizio finale. Esempio: Microsoft 365, Google Workspace.

Un elemento rilevante per la sicurezza è il concetto di multi-tenancy: più clienti condividono lo stesso hardware fisico del provider, separati a livello di hypervisor. La robustezza dell'isolamento dell'hypervisor è quindi critica. Meccanismi come Intel TXT e AMD SEV (Secure Encrypted Virtualization) offrono protezione crittografica della memoria delle VM anche nei confronti dell'hypervisor stesso, rilevanti in contesti cloud dove il provider è considerato un potenziale rischio.

Il cloud aggiunge ulteriori livelli di astrazione: cataloghi di immagini (AMI su AWS), istanze dimensionate, reti virtuali (VPC), security group, chiavi SSH, profili IAM, storage scalabile e servizi gestiti.

# 13. Appliance virtuali e firewall in cloud

Molti vendor distribuiscono appliance virtuali pronte per l'uso. Un firewall virtuale, per esempio, può essere installato in cloud e collegato a più reti virtuali. Esempi di firewall virtuali di uso comune includono pfSense/OPNsense (open source), Palo Alto VM-Series, Fortinet FortiGate VM e Cisco FTDv.

Uno scenario tipico prevede:

- una rete pubblica esposta a Internet (DMZ);

- un firewall virtuale con regole di filtraggio e logging;

- una rete interna protetta;

- server applicativi dietro il firewall.

Questa architettura replica nel cloud un modello classico di rete aziendale, ma con componenti software-defined. Le regole di filtraggio, il routing e le policy di sicurezza sono configurati via API o interfaccia web, senza alcun hardware fisico.

# 14. Reti virtuali: SDN e overlay network

La virtualizzazione moderna non riguarda solo server e sistemi operativi. Anche la rete può essere definita via software. È però importante distinguere due concetti spesso confusi:

## Software-Defined Networking (SDN)

SDN, nel senso tecnico definito dalla Open Networking Foundation, è un'architettura che separa il control plane (la logica di decisione del routing) dal data plane (l'effettivo inoltro dei pacchetti). Un controller centralizzato programma il comportamento dei dispositivi di rete tramite protocolli come OpenFlow. Soluzioni come ONOS, OpenDaylight e VMware NSX ne sono implementazioni. SDN è principalmente una tecnologia per data center e reti enterprise di grandi dimensioni.

## Overlay network e VPN mesh

Soluzioni come Tailscale, ZeroTier o WireGuard non sono SDN in senso stretto: sono reti overlay che creano un layer virtuale di connettività sopra la rete fisica esistente, permettendo a macchine distribuite geograficamente di comunicare come se fossero sulla stessa rete privata. Tailscale, in particolare, si basa sul protocollo WireGuard e su un piano di controllo centralizzato che gestisce chiavi e autorizzazioni. Questi strumenti sono molto utili per:

- amministrazione remota;

- laboratori distribuiti;

- accesso sicuro a server;

- collegamento tra ambienti locali e cloud;

- gestione di macchine sparse geograficamente.

# 15. Limiti e rischi della virtualizzazione

La virtualizzazione non elimina tutti i problemi. I principali limiti tecnici sono:

- consumo di RAM (risorsa critica nei laboratori);

- contesa sulla CPU in caso di carichi intensivi simultanei;

- I/O disco insufficiente, specialmente con storage meccanico;

- configurazioni di rete errate che compromettono l'isolamento;

- dipendenza dall'hypervisor — una vulnerabilità nell'hypervisor può impattare tutte le VM ospitate;

- gestione delle licenze: alcuni vendor (es. Oracle) calcolano le licenze software in base ai core fisici dell'host, indipendentemente dalle vCPU assegnate alla VM;

- VM sprawl: proliferazione incontrollata di VM non più utilizzate ma ancora in esecuzione, con consumo silenzioso di risorse e aumento della superficie di attacco.

Dal punto di vista della sicurezza, è essenziale configurare correttamente:

- reti virtuali e isolamento tra segmenti;

- permessi sulle cartelle condivise tra host e VM (un vettore di potenziale contaminazione in analisi malware);

- snapshot (evitare di lasciare snapshot con dati sensibili su VM condivise);

- accessi remoti all'hypervisor;

- aggiornamenti regolari dell'hypervisor (per mitigare vulnerabilità VM escape);

- separazione fisica o logica tra VM sensibili e VM esposte.

# 16. Container come evoluzione successiva

I container rappresentano un'evoluzione rispetto alle macchine virtuali. Una macchina virtuale include un intero sistema operativo guest. Un container, invece, isola un'applicazione e le sue dipendenze condividendo il kernel del sistema host.

I meccanismi tecnici sottostanti nei sistemi Linux sono due:

- namespace: isolano le risorse visibili al processo (PID, rete, filesystem, utenti, hostname);

- cgroups (control groups): limitano le risorse che il processo può consumare (CPU, memoria, I/O).

Questo riduce: consumo di risorse, tempo di avvio, peso delle immagini (non contengono un intero OS) e duplicazione dei sistemi operativi.

Una distinzione importante riguarda la portabilità: i container Linux condividono il kernel del sistema host. Su Linux questo avviene nativamente. Su Windows e macOS, Docker esegue un kernel Linux leggero in una VM (tramite WSL2 su Windows, o una VM dedicata su macOS): i container Linux non girano direttamente sul kernel Windows o macOS.

Docker è uno degli strumenti più diffusi per la gestione dei container. Permette di scaricare immagini applicative da registri pubblici (Docker Hub), eseguirle in ambienti isolati e collegarle tramite reti virtuali. Lo standard OCI (Open Container Initiative) definisce le specifiche dei container in modo indipendente dal vendor. Kubernetes è il sistema di orchestrazione di container più diffuso, usato per gestire applicazioni distribuite su cluster.

Dal punto di vista della sicurezza, l'isolamento dei container è minore rispetto alle VM: un container compromesso che sfrutta una vulnerabilità del kernel può potenzialmente compromettere l'host o altri container. Questo è documentato da vulnerabilità reali (es. CVE su runc). Per workload ad alto rischio è preferibile usare VM o soluzioni di container con isolamento rinforzato (come gVisor o Kata Containers).

La virtualizzazione tradizionale e la containerizzazione non si escludono: spesso convivono. Un container può girare dentro una macchina virtuale, e molte piattaforme cloud usano entrambe le tecnologie in modo complementare.

# 17. Sintesi finale

La virtualizzazione ha trasformato il modo in cui vengono costruiti, gestiti e protetti gli ambienti informatici. Ha permesso di passare da infrastrutture basate su molti server fisici dedicati a modelli più flessibili, nei quali le risorse sono astratte, isolate, replicate e automatizzate.

Nel networking e nella sicurezza informatica, la virtualizzazione è fondamentale per:

- costruire laboratori e simulare reti complesse;

- analizzare malware in ambienti isolati e ripristinabili;

- condurre penetration test;

- studiare Active Directory, Kerberos e attacchi laterali;

- testare firewall e architetture di rete;

- creare e gestire ambienti cloud;

- automatizzare il deployment con Vagrant e Ansible;

- ridurre i tempi di ripristino;

- migliorare l'efficienza operativa.

Comprendere macchine virtuali, hypervisor, reti virtuali, formati di immagine, Vagrant, Ansible, cloud e container è essenziale per chi studia reti moderne e cybersecurity.

# Bibliografia

Le opere sono organizzate per argomento e citate in formato APA 7ª edizione.

## Fondamenti di virtualizzazione e hypervisor

Popek, G. J., & Goldberg, R. P. (1974). Formal requirements for virtualizable third generation architectures. *Communications of the ACM, 17*(7), 412–421. https://doi.org/10.1145/361011.361073

Smith, J. E., & Nair, R. (2005). *Virtual machines: Versatile platforms for systems and processes*. Elsevier / Morgan Kaufmann.

Rosenblum, M., & Garfinkel, T. (2005). Virtual machine monitors: Current technology and future trends. *IEEE Computer, 38*(5), 39–47. https://doi.org/10.1109/MC.2005.176

Uhlig, R., Neiger, G., Rodgers, D., Santoni, A. L., Martins, F. C. M., Anderson, A. V., Bennett, S. M., Kagi, A., Leung, F. H., & Smith, L. (2005). Intel virtualization technology. *IEEE Computer, 38*(5), 48–56. https://doi.org/10.1109/MC.2005.163

## KVM e virtualizzazione Linux

Kivity, A., Kamay, Y., Laor, D., Lublin, U., & Liguori, A. (2007). KVM: The Linux virtual machine monitor. *Proceedings of the Linux Symposium, 1*, 225–230. https://www.linux-kvm.org/images/b/b0/KvmOls2007.pdf

Bellard, F. (2005). QEMU, a fast and portable dynamic translator. *Proceedings of the USENIX Annual Technical Conference (FREENIX Track)*, 41–46. https://www.usenix.org/legacy/event/usenix05/tech/freenix/bellard.html

## Xen e paravirtualizzazione

Barham, P., Dragovic, B., Fraser, K., Hand, S., Harris, T., Ho, A., Neugebauer, R., Pratt, I., & Warfield, A. (2003). Xen and the art of virtualization. *Proceedings of the 19th ACM Symposium on Operating Systems Principles (SOSP '03)*, 164–177. https://doi.org/10.1145/945445.945462

## Container e isolamento

Merkel, D. (2014). Docker: Lightweight Linux containers for consistent development and deployment. *Linux Journal, 2014*(239), articolo 2. https://dl.acm.org/doi/10.5555/2600239.2600241

Burns, B., Grant, B., Oppenheimer, D., Brewer, E., & Wilkes, J. (2016). Borg, Omega, and Kubernetes. *ACM Queue, 14*(1), 70–93. https://doi.org/10.1145/2898442.2898444

Price, D. (2004). Solaris zones: Operating system support for server consolidation. *Proceedings of the 18th USENIX Large Installation System Administration Conference (LISA '04)*, 43–54. https://www.usenix.org/legacy/event/lisa04/tech/full_papers/price/price.pdf

## Cloud computing e IaaS

Armbrust, M., Fox, A., Griffith, R., Joseph, A. D., Katz, R., Konwinski, A., Lee, G., Patterson, D., Rabkin, A., Stoica, I., & Zaharia, M. (2010). A view of cloud computing. *Communications of the ACM, 53*(4), 50–58. https://doi.org/10.1145/1721654.1721672

Mell, P., & Grance, T. (2011). *The NIST definition of cloud computing* (NIST Special Publication 800-145). National Institute of Standards and Technology. https://doi.org/10.6028/NIST.SP.800-145

## Software-Defined Networking

McKeown, N., Anderson, T., Balakrishnan, H., Parulkar, G., Peterson, L., Rexford, J., Shenker, S., & Turner, J. (2008). OpenFlow: Enabling innovation in campus networks. *ACM SIGCOMM Computer Communication Review, 38*(2), 69–74. https://doi.org/10.1145/1355734.1355746

## Sicurezza, VM escape e malware analysis

Garfinkel, T., & Rosenblum, M. (2003). A virtual machine introspection based architecture for intrusion detection. *Proceedings of the Network and Distributed Systems Security Symposium (NDSS '03)*. Internet Society. https://www.ndss-symposium.org/ndss2003/virtual-machine-introspection-based-architecture-intrusion-detection/

Raffetseder, T., Kruegel, C., & Kirda, E. (2007). Detecting system emulators. *Proceedings of the 10th International Conference on Information Security (ISC 2007)*, Lecture Notes in Computer Science, vol. 4779, 1–18. Springer. https://doi.org/10.1007/978-3-540-75496-1_1

Rutkowska, J. (2004). *Red Pill: Detecting VMM using (almost) one CPU instruction* \[Technical report\]. https://www.invisiblethings.org/papers/redpill.html

## Automazione e Infrastructure as Code

HashiCorp. (2024). *Vagrant documentation*. https://developer.hashicorp.com/vagrant/docs

Red Hat. (2024). *Ansible documentation*. https://docs.ansible.com/ansible/latest/index.html

Morris, K. (2016). *Infrastructure as code: Managing servers in the cloud*. O'Reilly Media.

## Active Directory e sicurezza

Metcalf, S. (2015). *攻撃と防御: Advanced Active Directory attack and defense techniques* \[Presentazione tecnica, DerbyCon 2015\]. https://adsecurity.org/wp-content/uploads/2015/09/2015-DerbyCon-Attack-and-Defense-ADSecurity.pdf

Microsoft. (2023). *Active Directory Domain Services overview*. Microsoft Learn. https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview

**Nota:** I riferimenti agli strumenti open source (KVM, QEMU, Docker, Vagrant, Ansible) includono sia la letteratura accademica originale sia la documentazione ufficiale aggiornata, in quanto per software in evoluzione attiva la documentazione ufficiale costituisce la fonte normativa primaria. Tutti gli URL sono stati verificati al momento della redazione del documento.
