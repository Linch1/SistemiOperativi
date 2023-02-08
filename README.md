
<h1 align="center">Sistemi Operativi</h1>

>Domande di teoria  degli esami di sistemi operativi. 
>Aiutano per fare un recap generale o per fare un ripasso degli argomenti pricipale

[File System](#file-system)
[Files](#files)
[Processi](#processi)
[Sys Call](#sys-call)
[Memory](#memory)
[Legge di Little](#legge-di-little)
[IPC](#ipc)
[Allocators (SLUB/BUDDY)](#custom-allocators-slubbuddy)

### File System

**In cosa consiste l’operazione di mount di un file system?** *(2018/01/23)*
Tramite tale operazione il sistema operativo viene informato che un nuovo file system pronto
per essere usato. L’operazione, quindi, provvederà ad associarlo con un dato mount-point, ovvero la
posizione all’interno della gerarchia del file system del sistema dove il nuovo file system verrà caricato.
Prima di efferruare questa operazione di attach, ovviamente bisognerà controllare la tipologia e l’integrità
del file system. Una volta fatto ciò, il nuovo file system sarà a disposizione del sistema (e dell’utente)

**Si consideri l’implementazione di un file system che gestisce i files mediante lista concatenata di blocchi
(vedi progetto), ed un file system che invece utilizza un albero di indici memorizzati negli inode.
Illustrare brevemente i vantaggi dell’uno e dell’altro nell’eseguire le seguenti operazioni:
• accesso sequenziale
• accesso indicizzato
• operazioni su file di testo** *(2018/01/23)*

1. Accesso Sequenziale: in questo caso, il file system che usa la lista concatenata sarà favorito,
garantendo una maggiore velocità dell’operazione. Ciò poiché non bisogna effettuare alcuna ricerca
per trovare il blocco successivo, poiché esso sarà semplicemente il blocco next nella lista.
2. Accesso Indicizzato: questa operazione - contrariamente alla precedente - risulta essere molto
onerosa per il file system che usa la linked list. Infatti, per ogni accesso, bisognerà scorrere tutta
la lista finchè non viene trovato il blocco desiderato. La ricerca tramite inode risulterà molto più
efficiente.
3. Accesso su file di testo: per la natura del tipo di file (testo), la linked list risulterà più efficiente
ancora una volta. Questo poiché i file di testo sono memorizzati in maniera sequenziale sul disco,
riportandoci al caso 1.

**Descrivere con un breve esempio un File System con allocazione concatenata (Linked Allocation).**(2018/03/26)

Una delle problematiche a cui un File System deve sopperire e’ come allocare lo spazio
necessario ad ogni file che andra’ a storare sul disco. Nel caso della Linked Allocation, ogni file sara’
una lista concatenata di blocchi, dove ogni blocco contiene il puntatore al successivo. Cio’ permette
quindi di allocare i blocchi in maniera scattered sul disco (non devono essere per forza contigui). Il file
finisce quindi con un puntatore a NULL. Ovviamente, in questo caso, localizzare un blocco puo’ richiedere
numerosi cicli di I/O. Inoltre i puntatori utilizzati influiranno negativamente sullo spazio disponibile per
contenere i dati: supponendo che i blocchi siano da 512 byte e che un puntatore sia 4 byte, lo 0.78% del
disco sara’ occupato da puntatori.
Figure 4: Esempio di File System con linked allocation. Il file jeep iniziera’ nel blocco 9 ed avra’ al suo
interno il puntatore al successivo - il 16 - e cosi’ via, finche’ nel blocco 25 next → NULL.
Un altro problema di tale implementazione e’ la sua affidabilita’. Infatti basta un solo puntatore
corrotto per danneggiare uno o piu’ files.
Un esempio visivo di tale implementazione e’ riportato in Figura 4.

**Descrivere con un breve esempio un File System con allocazione a indice.**(2018/06/19)

Soluzione Un File System ha tra i suoi scopi l’allocazione e la gestione dello spazio sul disco per tutti i
file in esso contenuti. Nel caso di Indexed Allocation i blocchi di un file saranno posti in maniera scattered
sul disco (e non contigua), analogamente a quanto avviene nella allocazione a lista (Linked Allocation).
In questo caso pero’ ogni file conterra’ un index block, ovvero un blocco contenente i puntatori a tutti gli
altri blocchi componenti il file.
Quando un file viene creato, tutti i puntatori dell’index block sono settati a null; quando un nuovo
blocco viene richiesto e scritto, il puntatore a tale blocco entrera’ nell’index block.
Tale tipo di allocazione permette di guardagnare velocita’ rispetto ad una implementazione tramite
linked list nel caso in cui si effettuino molti accessi scattered ai blocchi (non bisogna scorrersi tutta la
lista, ma basta fare una ricerca). Il costo da pagare e’ lo spazio necessario a contenere l’index block
stesso.
Un esempio di tale implementazione e’ riportato nella Figura 3.
Figure 3: Esempio di allocazione indicizzata: nel primo blocco del file jeep si trova l’index block contente
i puntatori a tutti gli altri blocchi del file.

**Cos’e’ un Virtual File System (VFS)? Fornisci un esempio del loro utilizzo.**(2019/01/22)
Soluzione Con VFS viene indicato tutto il livello di astrazione costruito su un File System concreto.
Il VFS rende possibile ad un client di accedere a diversi tipi di File System in maniera uniforme e
consistente. Un esempio di VFS e’ la directory /proc in Linux. I file contenuti in detta directory sono
in realta’ runtime information del sistema - e.g. memoria di sistema, configurazione hardware - le quali
vengono rese accessibili come un file tramite l’astrazione del VFS.

**Cos’e’ la directory /proc e cosa contiene al suo interno?**(2018/09/11)

Soluzione La directory proc un virtual filesystem, poich non contiene file reali, bens runtime system
information - e.g. memoria di sistema, configurazione hardware, etc. Molte delle utilities di sistema,
infatti, si riferiscono ai file contenuti in questa cartella - e.g. lsmod ≡ cat /proc/modules.


**Sia dato un OS che operi su un disco primario /dev/sda1, quest’ultimo gestito da un File System (FS)
ext4. Supponendo di collegare un secondo disco /dev/sda2 formattato in ext4, spiegare in dettaglio
cosa succede quando viene eseguito il comando
sudo mount /dev/sda2 /home/linus/torvalds
Si schematizzi, inoltre, l’albero delle directory prima e dopo l’esecuzione di tale comando. Cosa sarebbe
cambiato se il disco /dev/sda2 fosse stato formattato in ExFAT?**(2019/02/15)

Tramite il comando mount l’OS viene informato che un nuovo FS e’ pronto per essere usato.
L’operazione, quindi, provvedera’ ad associarlo con un dato mount-point, ovvero la posizione all’interno
della gerarchia del FS del sistema dove il nuovo FS verra’ caricato. Prima di efferruare questa operazione
di attach, ovviamente bisognera’ controllare la tipologia e l’integrita’ del FS. Una volta fatto cio’, il nuovo
FS sara’ a disposizione del sistema (e dell’utente). In Figura 3 viene schematizzato tale processo.

Figure 3: Schematizzazione dei FS del sistema prima e dopo l’operazione di mount. La directory /
rappresenta il mount-point del disco principale /dev/sda1, mentre il disco secondario /dev/sda2 avra’
come root la directory /home/linus/torvalds.
Ovviamente se il disco /dev/sda2 fosse formattato in ExFAT non sarebbe cambiato niente per l’utente.
Cio’ poiche’ tramite il layer denominato Virtual File System (VFS), diversi FS vengono uniformati sotto
un’unica interfaccia. Cio’ significa che qualunque tipo di FS supportato dal sistema verra’ ”visto” come
una serie di file e directory, anche se organizzato in maniera diversa internamente.


### Files

**Cos’e’ un File Control Block (FCB)? Cosa contiene al suo interno?**(2018/03/26)

Il FCB e’ una struttura dati che contiene tutte le informazioni relative al file a cui e’
associato. Esempi di informazioni possono essere: permessi, dimensione, data di creazione, numero di
inode (se esiste), ecc. . Inoltre il FCB contiene informazione su la locazione sul disco dei dati del file
- ad esempio in un FS con allocazione concatenata il puntatore al primo blocco del file. In Figura 5 e’
riportata una illustrazione di tale struttura.

Figure 5: Esempio di FCB.La struttura contiene tutti gli attributi del file compresa la locazione dei dati -
qui rappresentato dal puntatore al primo blocco della lista contente i dati, supponendo un FS con linked
allocation.

**Che relazione c’e’ tra un File Descriptor ed una entry nella tabella globale dei file aperti del file system?**(2018/03/26)

Un File Descriptor consiste in un file handler che viene restituito ad un processo in seguito
ad una chiamata alla syscall open(). In seguito a tale chiamata, il sistema scandisce il FS in cerca del
file e, una volta trovato, il FCB e’ copiato nella tabella globale dei file aperti. Per ogni singolo file aperto,
anche se da piu’ processi esiste una sola entry nella tabella globale dei file aperti.
Viene, quindi, creata una entry all’interno della tabella dei file aperti detenuta dal processo, la quale
puntera’ alla relativa entry nella tabella globale, insieme ad altre informazione - e.g. permessi, locazione
del cursore all’interno del file, ecc. La syscall open() restituisce per l’appunto l’entry all’interno della
tabella del processo - il File Descriptor. Piu’ open() su uno stesso file da parte di uno stesso processo
generano descrittori diversi.

### Processi

**Cosa contiene il Process Control Block (PCB) di un processo? Illustrare inoltre il meccanismo di Context**(2019/02/15)
Il PCB di un processo contiene tutte le informazioni relative al processo a cui e’ associato.
Esempi di informazioni contenute nel PCB sono:
– stato del processo (running, waiting, zombie ...)
– Program Counter (PC), ovvero il registro contente la prossima istruzione da eseguire
– registri della CPU
– informazioni sulla memoria allocata al processo
– informazioni sull’I/O relativo al processo.
Il PCB e’ fondamentale durante il context switch, permettendo di salvare ”lo stato” del processo (PC,
registri ecc.) e di ripristinare l’esecuzione da dove la si era interrotta.

Una illustrazione di tale struttura dati e’ riportata in Figura 2a. (VEDILAAAA)
Supponendo di avere due processi P0 e P1 e che il primo sia in running. Per mettere in esecuzione P1,
l’Operating System (OS) deve salvare lo stato corrente di P0 in modo da poter rirpistinare l’esecuzione
dello stesso in un secondo momento. Quindi, il context switch può essere riassunto visivamente nella
Figura 2b. E’ bene notare che il context switch e’ fonte di overhead a causa delle varie operazioni di
preambolo e postambolo necessarie allo switch - e.g. salvare lo stato, blocco e riattivazione della pipeline
di calcolo, svuotamento e ripopolamento della cache.


**Quali risorse vengono usate per creare un thread? In cosa differiscono da quelle usate per la creazione
di un processo?**(2018/09/11)

Creare un thread - sia esso kernel o user - richiede l’allocazione di una data structure
contenente il register set, lo stack e altre informazioni quali la priorita’, come riportato in Figura 3.
Figure 3: Processo single thread vs. multi-thread.
Creare un nuovo processo invece, richiede l’allocazione di un nuovo PCB (una data structure molto
piu’ pesante). Il PCB contiene tutte le informazioni del processo, quali pid, stato del processo, infor-
mazioni sull’I/O, il Program Counter e la lista delle risorse aperte dal processo. Inoltre il PCB include
anche informazioni sulla memoria allocata dal processo (tabella delle pagine, regioni mmapped etc.).
Tale operazione e’ relativamente costosa.
In definitiva, quindi, la creazione di un thread e’ piu’ leggera rispetto a quella di un nuovo processo.

- Domanda 2018/09/11
Che cos’e’ una syscall ? Come e’ possibile aggiungere (in linea di massima) una nuova syscall in un OS
previsto di tale meccanismo?
Soluzione Una syscall e’ una chiamata diretta al sistema operativo da parte di un processo user-level
- e.g. quando viene invocata la funzione printf.
Nel caso si volesse aggiungere una nuova syscall, essa, una volta definita, va registrata nel sistema e
aggiunta al vettore delle syscall del sistema operativo, specificandone il numero di argomenti (ed il loro
preciso ordine).

### Sys Call

**Quali sono i passi necessari per aggiungere una qualsiasi syscall in un sistema operativo
provvisto di tale meccanismo?** *(2018/01/23)*
Una syscall e’ una chiamata diretta al sistema operativo da parte di un processo user-level
- e.g. quando viene invocata la funzione printf.
Una volta definita la syscall, va registrata nel sistema e aggiunta al vettore delle syscall del
sistema operativo, specificando il numero di argomenti (ed il loro ordine) nell’altro vettore di sistema
apposito.

**Cos’e’ la Syscall Table (ST)? Come e’ possibile installare una nuova syscall in un Sistema Operativo che
supporta tale meccanismo? Infine, in cosa differiscono la ST e l’Interrupt Vector (IV)?**(2019/01/22)
Una syscall e una chiamata diretta al sistema operativo da parte di un processo user-level
- e.g. quando viene fatta una richiesta di IO.
L’ IV e un vettore di puntatori a funzioni; queste ultime saranno le Interrupt Service Routine (ISR)
che gestiranno i vari interrupt. Analogamente, la ST conterra in ogni locazione il puntatore a funzione che
gestisce quella determinata syscall. Alla tabella verra associata anche un vettore contenente il numero e
lordine di parametri che detta syscall richiede.
Per registrare una nuova syscall, essa va registrata nel sistema ed aggiunta al vettore delle syscall del
sistema operativo, specificando il numero di argomenti (ed il loro ordine) nell’altro vettore di sistema
apposito.

**Spiegare brevemente la differenza tra open(...) e fopen(...) ?**  *(2018/01/23)*
Soluzione fopen(...) una funzione di alto livello che ritorna una stream, mentre open(...) una
syscall di basso livello che ritorna un file descriptor. fopen(...), infatti, nasconde una chiamata alla
syscall open(...)

**Cosa succede durante la system call fork? Illustrare in dettaglio i passi necessari alla sua
esecuzione, evidenziando come vengono modificate le strutture nel kernel.** (2018/02/16)
La syscall fork e’ usata per creare un nuovo processo - children - a partire da un processo
parent. Il processo creato, avra’ una copia dell’adress space del parent, consentendo di comunicare
facilmente tra loro. I processi, in questo caso, continueranno l’esecuzione concorrentemente. Il child,
inerita anche i privilegi e gli attributi nel parent, nonche’ alcune risorse (quali i file aperti). Una volta
creato il processo, se non viene invocata l’istruzione exec() il child sara’ una mera copia del parent (con
una propria copia dei dati); in caso contrario sara’ possibile eseguire un diveso comando.

Figure 3: Questa immagine schematizza cio’ che avviene durante la chiamata a fork(). In Figura 3a,
il processo P P esegue una fork. Il processo creato P F viene portato in coda di ready - o comunque
schedulato secondo l’algoritmo utilizzato - come riportato in Figura 3b. P F eredita, oltre ad una copia
dello stack di P P , le sue risorse.
Il parent aspetta che il child completi il suo task tramite l’istruzione wait(); quando il child finisce la sua
esecuzione (o avviene una chiamata exit()), il parent riprende il suo flusso, come riportato in Figura 4.

**Cos’e’ la tabella delle syscall? Cos’e’ invece l’interrupt vector?**(2018/02/16)
L’interrupt vector e’ un vettore di puntatori a funzioni; queste ultime saranno le Interrupt
Service Routine (ISR) che gestiranno i vari interrupt.
Analogamente, la tabella delle syscall conterra’ in ogni locazione il puntatore a funzione che gestisce
quella determinata syscall. Alla tabella verra’ associata anche un vettore contenente il numero e l’ordine
di parametri che detta syscall richiede. Per registrare una nuova syscall, essa va registrata nel sistema
ed aggiunta al vettore delle syscall del sistema operativo, specificando il numero di argomenti (ed il loro
ordine) nell’altro vettore di sistema apposito.

**A cosa serve la syscall ioctl? Come si puo’ configurare la comunicazione con devices a caratteri e seriali
in Linux?**(2019/02/15)

La syscall ioctl permette di interagire con il driver di un device generico - e.g. una webcam.
Tramite essa sara’ possibile ricavare e settare i parametri di tale device - e.g. ricavare la risoluzione della
webcam o settarne la tipologia di acquisizione dati.
Per configurare devices seriali a caratteri - e.g. terminali - e’ possibile usare le API racchiuse nella
interfaccia termios. Tramite di essa, avremo accesso a tutte le informazioni relative al device - e.g.
baudrate, echo, ... .

**Che relazione c’e’ tra una syscall, un generico interrupt e una trap? Sono la stessa cosa?**(2019/03/26)
Che relazione c’e’ tra una syscall, un generico interrupt e una trap? Sono la stessa cosa?
Soluzione Ovviamente le tre cose non sono la stessa cosa.
Una syscall e una chiamata diretta al sistema operativo da parte di un processo user-level - e.g.
quando viene fatta una richiesta di IO.
Un interrupt invece e’ un segnale asincrono proveniente da hardware o software per richiedere il
processo immediato di un evento. Gli interrupt software sono definiti trap. A differenza delle syscall, gli
interrupt esistono anche in sistemi privi di Sistema Operativo - e.g. in un microcontrollore. Quando una
syscall viene chiamata, una trap verra’ generata (un interrupt software), in modo da poter richiamare
l’opportuna funzione associata a tale syscall (attraverso la Syscall Table (ST)).

### Memory

**Formule**
`T_EAT = p_hit (T_TLB + T_RAM ) + (1 − p_hit ) · 2(T_TLB + T_RAM )`

**Cos’e’ una Shared Memory? Fornire un breve esempio del suo utilizzo.** (2018/02/16)
La shared memory e’ un meccanismo usato per permettere a processi diversi di comunicare
tra loro (Interprocess Communication - IPC). In questo caso, viene riservata una porzione di memoria
condivisa tra i vari processi, i quali potranno scambiarsi informazioni semplicemente scrivendo e leggendo
in tale porzione di memoria. I processi sceglieranno la locazione di memoria ed la tipologia di dati;
essi dovranno anche sincronizzarsi in modo da non operare contemporaneamente sugli stessi dati. La
shared memory, per esempio, e’ molto utile nel caso di problemi producer/consumer - o analogamente
client/server. In questo caso, infatti, sara’ necessario istanziare un buffer condiviso da entrambi i processi,
in modo che il produttore possa rendere disponibile ai consumatori cio’ che ha prodotto.
Figure 5: Diversi metodi di IPC: l’immagine a raffigura un IPC message-based; l’immagine b invece, una
comunicazione basata su shared memory.
Un altro metodo di IPC prevede l’uso di messaggi (Message Passing). In questo caso, i processi
comunicheranno inviandosi dei messaggi che saranno gestiti tramite opportune syscall - creando quindi
overhead. Questi ultimi sono da favorire nel caso in cui i dati da veicolare abbiano una dimensione
ridotta o nel caso di architetture fortemente multicore - per evitare problemi di coerenza delle cache.
Entrageditmbi i metodi sono riportati nella Figura 5.


**Cosa significa Copy On Write (COW) nella gestione della memoria? Illustrare con un semplice esempio
il suo funzionamento.** (2018/03/26)

Creando un nuovo processo figlio a partire da un padre implica una copia della memoria
usata da quest’ultimo. E’ bene notare che molto probabilmente il figlio chiamera’ una exec(), percio’
tale copia puo’ risultare inutile. A tal porposito e’ possibile usare la tecnica di Copy-On-Write. In questo
caso, inizialmente entrambi i processi - padre e figlio - condividono le stesse pagine di memoria - marcate
come copy-on-write pages - e quando uno dei due vuole modificare una delle pagine questa verra’ copiata.
Il processo e’ illustrato visivamente nella Figura 2.

Figure 2: Illustrazione del funzionamento del processo Copy-On-Write: i due processi condividono le
stesse pagine di memoria finche’ una delle due non necessita di modificarne qualcuna. In quel caso, la
pagina interessata verra’ copiata.


**Come si puo’ implementare l’algoritmo di sostituzione delle pagine ”Second Chance”?**(2018/03/26)
L’algoritmo Second Chance - o altresi’ noto come clock algorithm - e’ un algoritmo FIFO
per la sostituzione delle pagine che usa un reference bit (implementato in hardware) per capire quale
pagine bisogna rimpiazzare. In particolare:
– se il reference bit di una pagina e’ 0 allora essa viene rimpiazzata;
– se il reference bit e’ 1 il bit viene settato a 0 e la pagina viene lasciata in memoria, quindi si passa
alla pagina successiva - usando le stesse regole.
Un modo per implementare tale algoritmo e’ attraverso una coda circolare con un puntatore alla
pagina da rimpiazzare - che scorre nella coda finche’ non trova una pagina con reference bit pari a 0.
Trovata una pagina che soddisfa le richieste, viene eliminata dalla coda e rimpiazzata con la nuova.
L’algoritmo e’ illustrato in Figura 3.
Figure 3: L’algoritmo di sostituzione delle pagine Second Chance. Il puntatore scorre finche’ una pagina
con reference bit 0 non viene trovata; quindi la nuova pagina prendera’ il posto di quest’ultima nella
coda circolare.

**In relazione alla gestione della memoria, evidenziare le differenze tra segmenti e pagine.**(2019/02/15)

Gli indirizzi virtuali sono resi necessari per indicizzare un address space piu’ grande del
numero di registri disponibili. La memoria quindi viene organizzata in segmenti o pagine - o una
combinazione di entrambe. Di conseguenza, i primi bit dall’indirizzo virtuale faranno riferimento ad uno
di essi, mentre la restante parte indichera’ lo spiazzamento all’interno del segmento o pagina selezionata.
I segmenti sono stati il primo approccio a tale problema. Ogni segmento ha dimensione variabile
e percio’ e’ identificato da indirizzo di base e limite. Tali informazioni sono contenute in una tabella
apposita (Segment Table). Ovviamente il sistema puo’ proteggere alcuni segmenti non rendendoli ac-
cessibili tramite un bit di controllo. I contro della segmentazione sono principalmente frammentazione
esterna e la necessita’ di essere ricompattati.
Per far fronte alle pecche della segmentazione, architetture moderne usano la paginazione per vir-
tualizzare l’address space. La memoria viene quindi divisa in frames (pagine) di dimensione uguale
(prestabilita). La parte alta dell’indirizzo virtuale verra’ usata come key per la tabella delle pagine
(Page Table), andando ad evidenziare la pagina corrente. La rimanente parte dell’indirizzo verra’ us-
ata come offset all’interno della pagina. La paginazione permette anche di proteggere zone di memoria
(tramite bit di protezione) e puo’ essere implementata anche in maniera gerarchica (con n livelli di
indirezione).
E’ bene notare che entrambi gli approcci necessitano di strutture hardware dedicate per funzionare
efficientemente.

**Con riferimento agli algoritmi di Page Replacement, enumerare i 4 principali algoritmi usati per tale
scopo, ordinandoli in base al loro page-fault rate (dal piu’ alto al piu’ basso). Si evidenzino anche gli
algoritmi che soffrono dell’anomalia di Belady.**(2019/03/26)

Partendo dall’algoritmo con le peggiori performances in termini di page-fault rate, avremo:

| Algorithm |Belady  |
|--|--|
| FIFO | si’ |
| Second Chance | si’ |
| LRU | no |
| Optimal | no |

L’anomalia di Belady e’ un fenomeno che si presenta in alcuni algoritmi di rimpiazzamento delle pagine
di memoria per cui la frequenza dei page-fault puo’ aumentare con il numero di pagine disponibili (con
alcuni pattern di accesso alle pagine).


**Spiegare brevemente cos’e’ il Direct Memory Access (DMA) e quando viene usato.**
Alcuni device necessitano di trasferire grandi quantita’ di dati a frequenze elevate. Effet-
tuare tali trasferimenti tramite il protocollo genericamente usato per altri tipi di periferiche richiederebbe
l’intervento della CPU per trasferire un byte alla volta i dati - tramite un processo chiamato Programmed
I/O (PIO). Cio’ risulterebbe in un overhead ingestibile per l’intera macchina, consumando inutilmente
la CPU. Per consentire il corretto funzionamento di tali device evitando gli svantaggi del PIO, tali
periferiche possono avvalersi di controllori dedicati che effettuano DMA, cioe’ andando a scrivere diret-
tamente sul bus di memoria. La CPU sara’ incaricata soltanto di ”validare” tale trasferimento e poi
sara’ di nuovo libera di eseguire altri task.
Questo tipo di periferiche sono molto comuni ai giorni nostri e sono usate nella maggior parte dei
dispositivi elettronici - pc, smartphones, servers, . . . . Esempi di periferica che si avvalgono di controller
DMA sono videocamere, dischi, schede video, schede audio, ecc.


**Quali sono le principali differenze tra un indirizzo logico ed un indirizzo fisico? Da chi vengono generati?**(2019/07/16)
Un indirizzo logico non si riferisce ad un indirizzo realmente esistente in memoria. Esso e’
in realta’ un indirizzo astratto generato dalla CPU, che verra’ poi tradotto in un indirizzo fisico tramite
la Memory Management Unit (MMU). L’indirizzo fisico, quindi, si riferisce ad una locazione esistente
della memoria e non e’ generato dalla CPU, bensi’ dalla MMU.

### Legge di Little

**Cos’e’ la legge di Little (Little’s Law). (hint: teoria delle code, produttori/consumatori).** (2018/02/16)
La legge di Little e’ usata per valutare i vari algoritmi di scheduling, mettendo in relazione
la dimensione media della coda n con il tempo di attesa medio W e frequenza media di arrivo dei processi
nella coda λ. In particolare, essa e’ descritta dalla relazione:
*n = λ · W*
Tale legge si inserisce nello studio degli algoritmi di scheduling mediante Queuing Models. A differenza
di una valutazione analitica in cui bisogna conoscere il workload, un’analisi tramite Queuing Models
assume che quest’ultimo sia sconosciuto. In tal caso, si effettuera’ una stima probabilistica basata sulla
distribuzione di CPU ed I/O bursts e sulla distribuzione dei tempi di arrivo dei processi.


### IPC
**Cos’e’ una coda di messaggi IPC (mailbox)? Fornire un breve esempio del suo utilizzo.**(2018/06/19)

Tra i metodi di IPC (Inter-Process Communication) troviamo lo scambio di messaggi tra
piu’ processi. Rispetto all’implementazione tramite shared memory, si avranno numerosi vantaggi tra i
quali:
– maggiore flassibilita’ - e.g. e’ possibile implementare una comunicazione remota tra processi eseguiti
su macchine diverse
– semplicita’ di implementazione
– sicurezza nella comunicazione - e.g. nessuna lettura parziale dei dati poiche’ processati messaggio
per messaggio
Il costo da pagare, invece, sara’ in termini di performances:
– maggiore overhead poiche’ la comunicazione dovra’ essere gestita dal sistema operativo
– copia dei dati nella coda
– l’accesso ai dati e’ in generale piu’ lento perche’ limitato ad un messaggio per volta (la dimensione
di un messaggio e’ generalmente limitata e relativamente piccola).
La comunicazione e’ basata sulle operazioni di send e receive. Tali funzioni (syscall) possono essere
sviluppati in base alle specifiche esigenze dell’utente (comunicazione sincrona/asincrona, diretta/indi-
retta, limitata/illimitata ...).
Alla luce di quanto detto finora, una coda di messaggi e’ un oggetto gestito dal sistema operativo che
implementa una mailbox. I processi quindi potranno creare una coda di messaggi oppure linkarsi ad una
mailbox esistente a partire da un identificatore della coda. Le altre operazioni fondamentali sono:
- check dello status della coda
- post di un messaggio
- attesa di un messaggio (bloccante o non bloccante).



### Custom Allocators (SLUB/BUDDY)
**Illustrare brevemente SLAB e Buddy allocator, sottolineandone le differenze ed i casi di utilizzo.
Soluzione**(2019/02/15)
Tra le piu’ annoverate implementazioni di Memory Allocator traviamo: slab e buddy sys-
I. Slab Allocator : usato per allocare oggetti di dimensione fissa, puo’ allocarne fino ad un numero
massimo fissato. Il buffer viene quindi diviso in chunk di dimensioni item size. Per poter organiz-
zare il buffer viene quindi usata una struttura ausiliaria che tenga l’indice dei blocchi ancora liberi.
Una array list soddisfa tale richiesta.
II. Buddy Allocator : usato per allocare oggetti di dimensione variabile. Il buffer viene partizionato
ricorsivamente in 2 - creando di fatto un albero binario. La foglia piu’ piccola che soddisfa la
richiesta di memoria sara’ ritornata al processo. Il buddy associato ad una foglia sara’ l’altra
regione ottenuta dalla divisione del parent. Ovviamente, se un oggetto e’ piu’ piccolo della minima
foglia che lo contiene, il restante spazio verra’ sprecato. Quando un blocco viene rilasciato, esso
verra’ ricompattato con il suo buddy (se libero), risalendo fino al livello piu’ grande non occupato.




A luglio 2022 è capitato:
Una domanda su ioctl (Non mi ricordo di preciso cosa chiedesse, mi pare come si può interagire con un dispositivo input/output)
Come funziona lo slab allocator
Che funzione ha il kernel stack nel context switch 
Descrivere in breve i vari algoritmi di sostituzione delle pagine
