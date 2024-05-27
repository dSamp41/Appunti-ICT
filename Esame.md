Links:
https://tor.stackexchange.com/questions/423/what-are-good-explanations-for-relay-flags
https://2019.www.torproject.org/about/overview.html.en
https://2019.www.torproject.org/about/overview.html.en

# Tor: introduzione
La rete TOR è formata da circa 8000 relay. Un client Tor seleziona (almeno) 3 relay per formare un *circuito*, un insieme di stream TCP per comunicare con l'host destinatario.

TOR misura la larghezza di banda che ogni relay fornisce per assegnargli un peso. I pesi sono usati nella selezione per distribuire il carico verso relay con più banda disponibile.

La *directory authority* assegna una status flag ai relay. La flag ==GUARD== è assegnata ai relay il cui uptime è pari o superiore alla mediana dei relay familiari (A node is 'familiar’ if 1/8 of all active nodes have appeared more recently than it, OR it has been around for a few weeks) e la loro banda è pari o superiore a min(250KiB/s, median_relay_bandwidth).

I client selezionano 3 GUARD attive e le usano come relay di ingresso per tutti i loro circuiti. Queste vengono ruotata randomicamente ogni 30-60 giorni.

Un relay ottiene la flag ==STABLE== se il suo mean time between failure pesato è pari o superiore alla mediana dei relay attivi.

La flag ==EXIT== è assegnata a quei relay che possono comunicare con host esterni. Questi relay specificano una exit policy: range di indirizzi e porte con cui sono disponibili a connettersi.

I client usano queste policy per determinare il relay da scegliere alla fine di un circuito.

Tra le altre regole che un client *enforces*: non scegliere due relay con stesso prefisso di rete a 16 bit, o due relay appartenenti alla stessa famiglia (set of relays that mutually indicate that they belong to a group together). In generale fissato un circuito, questo viene usato per 10 minuti.

# Adversary model and metrics
### Adversary model
Un avversario realistico è in grado di osservare, ritardare, alterare (aggiungere o rimuovere parti di) una comunicazione. 

Tuttavia in questa analisi ci si limita ad un avversario passivo, end-to-end correlating. Tale avversario apprende la sorgente o destinazione di una comunicazione quando è in posizione di osservare uno o entrambi di questi. Inoltre è in grado di collegare le osservazioni di questo flusso di comunicazione ovunque esse siano nel loro percorso.

Tale avversario può aggiungere risorse di rete o corrompere quelle esistenti; non può però alterare in alcun modo il traffico di rete.


Le risorse che l'avversario possiede sono i relay o il server di destinazione => in senso più astratto l'avversario controlla una certa larghezza di banda già esistente o che può aggiungere alla rete TOR.

### Resource endowment
[IXP, AS ???]
Un avversario è visto come l'insieme di risorse che controlla. Dato che il protocollo di selezione dei percorsi di TOR da un peso relativo alla bandwidth, questa è una misura accurata delle risorse avversarie. 


### Adversary goal
L'obbiettivo principale (e più generale) dell'avversario è quello di deanonimizzare (collegare sorgente e destinazione) quanti più circuiti possibile. 

È possibile che i goal siano più specifici come identificare le destinazioni di specifiche classi di utenti, o compromettere circuiti in modo da individuare chi si connette a specifiche destinazioni.

Dei circa 3000 relay, 1/3 sono usati come guard relay mentre 1/3 sono usati da exit relay.

### Security metrics
In generale tali metriche sono definite rispetto ad uno specifico tipo di avversario, sono usate per determinare la sicurezza (su tempi umani) e devono permettere la stima di probabilità per eventi rilevanti.

Per questa analisi sono usate le seguenti metriche:
- distribuzione di probabilità sul numero di path compromessi per un dato utente
- distribuzione di probabilità sul tempo fino al primo path compromesso.

# Metodologia
Per valutare la sicurezza di TOR rispetto ad averrsario e metriche proposte, si usa il metodo Monte Carlo per campionare il flusso di traffico sulla rete durante varie tipi di attività degli utenti.

Per ogni campione si usa un modello della rete, si simula il comportamento degli utenti e si simula il risultato.

TorPS è un simulatore di selezione dei path, costruito a partire dai dati storici della rete per ricreare le condizioni sotto le quali i client hanno operato in passato. TorPs include un modello dei client, relay e dei loro stati.

### Tor Network Model
TorPS usa i dati di Tor Metrics per modellare gli stati passati della rete TOR. Tali dati permettono di determinare lo stato dei relay nel tempo (flags, exit policies). I relay senza descrittori o che non appaiono in un *consensus* (documento che la directory authority invia ai client contenente informazioni su tutti i relay), risultano come inattivi


### User Model
I ricercatori hanno sviluppato 5 modelli utente, corrispondenti a tipi di uso diversi del client TOR. Ogni modello consiste di una sequenza di stream Tor e l'orario. Uno stream include chiamate DNS e connessioni TCP.

Tre di questi modelli sono costruiti tracciando l'uso di alcune applicazioni. Ogni traccia corrisponde a 20 minuti di utilizzo. 

- ==Typical==: rappresenta un uso medio di Tor. Usa 4 tracce: Gmail+Google Chat alle 9.00, Google Calendar + Docs alle 12.00, Facebook alle 15.00, and navigazione web a parite dalle 18.00. 
- ==IRC==: rappresenta l'uso di Tor allo scopo esclusivo di accedere a chat IRC. Usa la traccia di una singola sessione IRC, ripetuta dalle 8.00 alle 17.00 da lunedì a venerdì.
- ==BitTorrent==: rappresenta l'uso di BitTorrent su Tor. Consiste nel download di un singolo file. Il modello ripete la traccia dalle 12.00 alle 18:00, sabato e domenica.

- WorstPort. modifica il modello Typical rimpiazzando la porta con la 6523. Tale porta è la penultima in capacità in uscita. [TODO?]
- ==BestPort==: modifica il modello Typical rimpiazzando la porta con la 443 (HTTPS).


| Model      | Streams/wee | # IPs | # Ports |
| ---------- | ----------- | ----- | ------- |
| Typical    | 2632        | 205   | 2       |
| IRC        | 135         | 1     | 1       |
| BitTorrent | 6768        | 171   | 118     |
| WorstPort  | 2632        | 205   | 1       |
| BestPort   | 2635        | 205   | 1       |

### Tor Client Model
TorPS simula il client nella creazione di circuiti. Prende in considerazione caratteristiche quali peso dei relay, selezione e rotazione delle guardie, exit policy, famiglia e prefisso di rete.

Tuttavia essendo una simulazione, ogni costruzione di circuito ha successo. Non viene valutato il caso in cui questa fallisca o il circuito cada durante l'uso.

# Internet Map 
?????


# SECURITY ANALYSIS
Si considerano due tipi di avversari: uno in grado di fornire/controllare relay su TOR, un altro è un ISP in grado osservare porzioni di rete TOR.


# Relay Adversary
Avversari in grado di eseguire/controllare relay è la minaccia più plausibile per gli utenti TOR, dato che chiunque può fornire un relay alla rete senza alcuna restrizione.

I client scelgono un relay in proporzione alla sua larghezza di banda, pertanto il traffico che un avversario può deanonimizzare è limitata solo dalla sua banda. 

Pertanto l'avversario in considerazione ha un banda significante (ma ragionevole) di 100MiB/s. 

Interessante notare che la banda non deve essere fornita da un singolo relay, per cui un avversario può fornirla tramite una botnet.

La banda ha un costo, per cui arrivare a controllare una grossa parte di TOR è estremamente difficile. Tuttavia la velocità continua ad aumentare, per cui il costo che un avversario affronta per arrivare a minacciare la rete continuerà a diminuire. 

### Adversary Resource Allocation
L'avversario deve determinare come allocare la sua banda per massimizzare la probabilità di compromettere circuiti.

Dato che un realy non può essere scelto due volte nelllo stesso circuiti, deve controllare almeno 2 relay per eseguire una correlazione.

Si suppone che l'avversario possieda un guard ed un exit relay con uscita verso ogni indirizzo e porta. Entrambi hanno abbastanza banda da ottenere la flag ==fast== 

L'allocazione della banda come rapporto tra relay guard su relay exit che massimizza la probabilità di compromettere una comunicazione è di 5:1.

![[Schermata del 2024-05-24 14-29-41.png]]


Allocare più banda per le guardie è giustificato dal fatto che per un avversario è più strategico ottenere una guardia dato che un utente la cambia molto meno frequentemente di quanto si usi una nuova exit.

### Analisi
I vari modelli di utente sono esposti a diversi problemi di sicurezza.

Ad esempio inviare molti stream su TOR, implica un rate maggiore di creazione di circuiti; ciò aumenta le possibilità che venga scelto un relay malevolo.

Una destinazione non raggiungibile da molti exit relays, invece aumenta la probabilità che venga scelto un exit relay malevolo.

Fissato un avversario che controlla un guard relay con banda 83.3 MiB/s ed un exit relay con banda 16.7 MiB/s, i ricercatori hanno usato TorPS per simulare sui vari modelli utente.

Per tutti i modelli, emerge che esiste una probabilità maggiore dell'80% di deanonimizzazione entro 6 mesi. Il tempo mediano è inferiore a 70 giorni. 

![[Schermata del 2024-05-24 14-41-24.png]]


Emerge anche che il tempo per scegliere una guardia malevola (50-60 giorni) è molto maggiore del tempo per scegliere una exit malevola (2.5 giorni)
![[Schermata del 2024-05-24 14-43-27.png]]

Questo implica che un avversario che prova la deanonimizzazione degli utenti avrà come priorità il far scegliere una guardia malevola.



Il rate di compromissione completa (visto come prodotto tra rate di compromissione di guard ed exit) mediano è tra 0.25-1.5%.

I rate di compromissione delle guard è pressocchè invariato per i vari utenti, dato che Ip:porta delle destinazioni non è considerato nella scelta iniziale.

Le exit sono scelte indipendentemente per ogni circuito, per cui la distribuzione del rate di compromissione è una normale con media pari alla frazione di exit_bandwidth fornita. 

### Valutazioni
Il modello BitTorrent crea 2.5 volte gli streams creati dal Typical, ed oltre 50 volte rispetto ad IRC. Inoltre fra le 171 porte usate, diverse sono rifiutate dalla exit policy default di TOR. 

Questo permette ad un avversario di fornire una percentuale di banda maggiore: ciò si riflette in un temo mediano di exit compromise di meno di 6 ore, e un rate mediano del 12%.

Anche i modelli IRC e WorstPort soffrono di questo probelma dato che entrambi si connettono a porte poco "permesse".

I modelli Typical e BestPort risultano i più sicuri: le porte 80 e 443 sono praticamente supportate da tutti gli exit relay.

![[Schermata del 2024-05-24 15-06-25.png]]




Un effetto notevole sul tempo per compromettere è dato anche dalla quantità totale di bandwidth fornita.

Sperimentando con velocità tra 10-200 MiB/s emerge che raddoppiare la banda porta a dimezzare i tempi di compromissione. Con una velocità di 200 MiB/s, l'avversario compromette un utente entro 30 giorni con una probabilità del 50%.

![[Schermata del 2024-05-24 15-10-23.png]]

Per confronto si suppone che un avversario non controlli una guardia/exit fino al secondo mese di simulazione, quando l'utente ha già scelto e rotato le sue guardie.

Nei restanti 4 mesi, l'avversario riesce a deanonimizzare l'utente nel 70% dei casi.
Tale probabilità è simile a la probabilità di compromissione entro 4 mesi nel caso in cui controllasse i relay fin dall'inizio.



# Network Attack
A differenza di un relay adversary, un network adversary non controlla relay nella speranza che un client ne scelga uno; sfrutta la sua posizione di carrier del traffico per correlare il traffico che eventualmente attraverserà la sua infrastruttura fra guard ed exit relay di un circuito.

##### Client Behavior
Si considerano solo i modelli Typical, BitTorrent ed IRC

Un caso particolare è quello di un client la cui comunicazione inizia e termina nella stessa AS => l'avversario può deanonimizzare l'intero traffico, 

Dalla analisi quindi sono esclusi i casi in cui l'AS contiene il client o la destinazione di un dato client.

##### Client Location
I client sono dislocati fra i 5 più popolari AS (4 tedeschi ed 1 italiano)

### Metodologia
Sono analizzati i percorsi client-guard per 5 volte, uno per ogni possibile locazione del client.

### Network Adversaries
Si considerano 3 tipi di network adversaries: AS, IXP e IXP organizations.

Gli IXP sono punti in cui più AS si interconnettono, perciò controllare uno o più IXP garantisce una significativa abilità di deanonimizzare il traffico TOR.

Una stessa organizzazione può controllare diversi IXP: 19 organizzazioni amministrano 90 IXP


### Analisi
Tutti gli stream generati da un client per una data posizione vengono aggregati e viene contato il numero di streams in cui un avversario esiste lungo il percorso (sia in ingresso che in uscita). Dopodichè viene selezionata l'entità che compromette il maggior numero di streams.

### AS adversary
Contro un AS avversario, una compromissione entro un giorno ha probabilità 45.9% per Typical, 64.9% per IRC, e 76.4% per BitTorrent nel worst-case.

Almeno una stream viene compromessa entro 3 mesi nel 98% dei casi.

Nel best-case, gli utenti IRC sono compromessi entro 44 giorno (mediana); entro 90 giorni il 44% dei Typical e il 38% dei BitTorrent sono compromessi

![[Schermata del 2024-05-25 11-02-20.png]]


### IXP adversary
Risultati simili ad un AS adversary per il worst case.

Rappresentano una minaccia minore nel best case. Meno del 20% dei client usano una stream che può essere compromessa entro 3 mesi.

Questa discrepanza è spiegata dal fatto che l'80% delle connessione non attraversa un IXP.

![[Schermata del 2024-05-25 11-03-01.png]]

Emerge come una organizzazione ha più potere rispetto ad un IXP singolo. Nei primi 30 giorni un singolo IXP compromette il 3.7% dei samples, contro il 12.4% di una organizzazione.


### Discussione
A differenza del Relay Adversary, emerge che il comportamento degli utenti con poca diversità di destinazioni, comporta una maggiore probabilità di compromissione.

Un singolo AS-adversary può compromettere il 50% dei client IRC entro 44 giorni, cifra simile a quella dei Typical user compromessi durante <u>tutto</u> il periodo.

Questo è causato da semplice probabilità: man mano che l'inseme i destinazioni si restringe, è più probabile che il client scelga un percorso malevolo.

Per aumentare la sicurezza quindi è bene che un utente diversifichi le proprie destinazioni.


Emerge anche un picco iniziale di client compromessi, seguito da un declino del rate di compromissione.

Questo perché se una guard compromessa è scelta, basta solo che un exit malevolo sia dall'altro lato del circuito.
Tuttavia se questo non accade, non sarà possibile compromettere alcun circuito fino a quando non verranno scelte nuove guardie.


Per quanto gli IXP abbiano una minore probabilità di compromettere il traffico, va notato come sia meno complesso eseguire una correlazione del traffico.,data la loro concentrazione geografica, a differenza dei più sparsi AS.



# Sniper Attack

### Detour su protocollo TOR
[cell == pacchetto TCP]
I relay di un circuito usano TCP. 
Ogni edge node (client, exit) ha una ==package window== inizializzata a 1000, decrementata per ogni cell inviata, ed una ==delivery windows== (inizializzata a 100), decrementata per ogni cell ricevuta.

Il packaging edge di un circuito smetterà di inviare pacchetti quando la sua package windows raggiunge 0; il delivery edge invece invierà un segnale (SENDME) al PE quando una delivery window lungo il circuito raggiunge 0.

SENDME spinge il PE ad aumentare la propria packaging window ed il DE ad aumentare la propria delivery window. 
SENDME segnala la disponibilità a ricevere altri pacchetti. (ricarica il credit)

### Attacco: versioni base
L'attacco si basa su un fatto: un DE che smette di leggere, causa al prossimo nodo il riempimento del buffer con un pacchetto pari ad una package window completa (1000 cells).

Il riempimento del buffer causerà la terminazione del processo Tor da parte dell'OS => relay fuori dalla rete TOR.

![[Schermata del 2024-05-27 11-29-32.png]]




### Attacco: versione efficiente
Nella versione base è necessario che un avversario possa caricare/generare dati.
Una versione efficiente elimina tale necessità: sfrutta i segnali di controllo di flusso offerto da TOR.

Il segnale SENDME implica che il DE abbia ricevuto le cells, ma il DE può inviare tali segnali anche senza aver effettivamente ricevuto i dati.

La versione efficiente combina l'invio di SENDME con il meccanismo di stop della lettura usato nelle versioni base.

L'avversario deve controlla un singolo client malevolo. Il client costruisce un circuito selezionando l'obbiettivo come entry. Dopodichè inizia il download di alcuni file di grandi dimensioni (per assicurarsi che la package window dell'exit venga svuotata).


Il client smetterà di leggere dalla connessione verso l'entry relay, ma invierà SENDME verso l'exit, per assicurarsi che la sua package window non venga azzerata, quindi che possa continuamente inviare pacchetti sul circuito.

Questi pacchetti verranno bufferizzati nell'entry relay, consumandone la memoria fin quando il processo TOR non verrà terminato dall'OS.

![[Schermata del 2024-05-27 11-41-52.png]]