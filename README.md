**PdS 2023 - Laboratorio OS161 - 4**

***Per affrontare questo laboratorio è necessario:*** 

- ***aver svolto (e capito) i laboratori 2 e 3*** 
- ***il laboratorio 2 serve per capite system call e (soprattutto) la chiusura di un processo*** 
- ***il laboratorio 3 per poter realizzare la sincronizzazione.*** 

**Realizzare la system call waitpid (attesa fine processo)**

Si vuole realizzare il supporto per la system call waitpid (in Unix/Linux esiste anche la wait, che attende un qualunque processo child), che permette a un processo di attendere il cambio di stato di un altro processo, di cui sia noto l’identificatore (pid).  

Si vedano ad esempio le documentazioni di waitpid su[ https://linux.die.net/man/2/waitpid ](https://linux.die.net/man/2/waitpid)o [https://www.freebsd.org/cgi/man.cgi?query=waitpid ](https://www.freebsd.org/cgi/man.cgi?query=waitpid)

Per semplicità, si chiede di gestire unicamente il cambiamento di stato a processo “terminato” (sarebbe necessario gestirne altri, quali wait/resume connessi a un signal). In sintesi, dopo thread\_exit (dell’ultimo/solo thread di un dato processo) un processo resta in stato “zombie” fintanto che un altro processo non fa una wait o waitpid (in OS161 solo waitpid), e ne ottiene quindi lo stato di uscita.  

Il laboratorio può essere suddiviso in parti, che si consiglia di realizzare un pezzo alla volta, passando al successivo dopo aver messo a punto (compresa esecuzione/debug)  il precedente: 

- Attesa della terminazione di un processo user, ritornandone lo stato di uscita 
- Distruzione della struttura dati di un processo  (struct proc) 
- Assegnazione di pid a processo, gestione della tabella dei processi e waitpid 
- (opzionale) realizzare le system call getpid e fork  

**Attesa di terminazione processo** 

Si consiglia di realizzare dapprima una funzione int proc\_wait(struct proc \*p), che gestisca *(senza bisogno di gestire pid e di supportare la system call waitpid)*, mediante semaforo o condition variable (aggiunti come campo alla struct proc), l’attesa della fine (con chiamata alla syscall \_exit())  di un altro processo, di cui si ha il puntatore alla relativa struct proc. 

Si tratta quindi di una funzione del kernel, utilizzabile solo all’interno di questo (perché sfrutta il puntatore a struct proc). Tale funzione potrebbe essere realizzata in kern/proc/proc.c e provata (chiamata) all’interno di common\_prog, dopo che questa ha chiamato thread\_fork con successo, per aspettare la fine del processo attivato (e non tornare immediatamente al menu che richiederebbe subito un altro comando). La common\_prog potrebbe quindi aspettare la fine del processo child mediante  

exit\_code = proc\_wait(proc); 

e stampare su console il codice di ritorno (quello ricevuto dalla \_exit - sys\_\_exit  e salvato nella struct proc) prima di ritornare al programma chiamante. La figura rappresenta lo schema delle chiamate e della sincronizzazione. Si noti che le chiamate a wait() e signal() vanno sostituite da opportune funzioni, che dipendono dalle scelte implementative fatte (semaforo, condition variable, funzione wrapper o chiamata diretta) 

Kernel main thread  cmd\_progthread thread  (menu) ![](Aspose.Words.b30fc37c-fef2-4d26-92af-0db7bd5b376e.001.png)![ref1]![ref2] becomes 

userprocess (palin) 



|<p>common\_prog(...) { ![](Aspose.Words.b30fc37c-fef2-4d26-92af-0db7bd5b376e.004.png)</p><p>`  `proc =[ proc_create_runprogram ](../../df/d03/proc_8h.html#aae835d06c28841caf3b0690c46012c28)(...); </p><p>`  `result =[ thread_fork(](../../d8/da9/include_2thread_8h.html#afcf81f11876cc2f4556b20c691c583ab)...,  </p><p>`    `proc,[ cmd_progthread,](../../d2/d0a/menu_8c.html#a3499a7a133dc64bf4abc25d460519436) ...); </p><p>`  `exit\_code = proc\_wait(proc);   ... </p><p>} </p>|||
| - | :- | :- |
|<p>proc\_wait(...) { </p><p>` `**wait(...);** /\* sem or cv \*/  /\* get exit status  </p><p>`    `and destroy userprocess    \*/</p>|||

|<p>/\* KERNEL MODE \*/ ![](Aspose.Words.b30fc37c-fef2-4d26-92af-0db7bd5b376e.005.png)cmd\_progthread(...) { </p><p>`  `result = runprogram(“testbin/palin”); }  </p><p>runprogram(...) { </p><p>`  `load\_elf(); </p><p>`  `enter\_new\_process(…); } </p>|||
| :- | :- | :- |
|<p>/\* USER MODE: proces main(...) { </p><p>`  `exit(0);</p>|s palin \*/ ||
|<p>/\* KERNEL MODE: system call \*/ ![](Aspose.Words.b30fc37c-fef2-4d26-92af-0db7bd5b376e.006.png)syscall(...) { </p><p>`  `switch (callno) { </p><p>`    `sys\_\_exit(status);   } </p><p>` `}</p>|||
| :- | :- | :- |
|sys\_\_exit(...) { |||
**Distruzione della struct proc**  ![](Aspose.Words.b30fc37c-fef2-4d26-92af-0db7bd5b376e.007.png)![](Aspose.Words.b30fc37c-fef2-4d26-92af-0db7bd5b376e.008.png)

*ATTENZIONE: si ribadisce il consiglio di andare avanti un passo alla volta, realizzando la distruzione della struct proc solo dopo aver realizzato e verificato, eventualmente con debug, la correttezza della proc\_wait().* 

La struct proc di un processo non può essere distrutta (durante la \_exit) finchè un altro processo che chiami wait/waitpid non ne riceva la segnalazione (con lo stato di uscita). Tra le varie soluzioni possibili per liberare quindi una struct proc mediante la proc\_destroy, si consiglia di chiamare quest’ultima all’interno della proc\_wait, dopo l’attesa su semaforo o condition variable (cioè la struct non viene distrutta quando termina il processo ma all’interno di una proc\_wait, fatta da un altro processo, eventualmente il kernel).  

Questo implica, probabilmente, una modifica alla realizzazione precedente della sys\_\_exit, che ora non necessiterebbe più di rilasciare l’address space, ma unicamente di segnalare (su semaforo o condition variable) la fine del processo, prima di chiamare thread\_exit. In altri termini, la sys\_\_exit termina il thread, non distrugge la struttura dati del processo, ma ne segnala semplicemente la terminazione. La distruzione completa di un processo viene effettuata dalla proc\_wait dopo che ne ha ricevuto la segnalazione di fine. La proc\_wait gestisce inoltre il ritorno dello stato di uscita del processo. 

*ATTENZIONE: la funzione proc\_destroy() richiede che il processo che si sta distruggendo (struttura dati) non abbia più thread attivi (si veda il codice della proc destroy, che contiene  l’asserzione[ KASSERT(](../../de/d14/lib_8h.html#a8f0725c87f662db8a0f3ca04379da56f)proc-[>p_numthreads ](../../de/d48/structproc.html#a006b8c2e8e12e7d4d346bfb9a7cdb2dd)== 0);). Siccome la sys\_\_exit() segnala la fine del processo prima di chiamare thread\_exit(), è possibile che il kernel in attesa (nella common\_prog()) venga svegliato e chiami la proc\_destroy() prima che* 

*thread\_exit() “stacchi” il thread dal processo (si veda il codice della thread\_exit(), in particolare la chiamata a proc\_remthread()). La soluzione per evitare questa corsa critica (race) non è univoca. Si suggerisce, come possibile opzione, di chiamare la proc\_remthread() in modo esplicito nella sys\_\_exit(), “prima” di segnalare la fine del processo, e modificare la thread\_exit() in modo che accetti un thread già staccato dal processo (attenzione però a non OBBLIGARE la thread\_exit() a vedere SEMPRE un thread “staccato”: la thread\_exit() viene chiamata anche in altri contesti, fuori dalla sys\_\_exit()).* 

**Assegnare pid** 

La proc\_wait non realizza completamente il lavoro richiesto alla waitpid, in quanto parte da un puntatore a processo (invece che da un pid, un valore intero). 

Per l’attribuzione di un pid (process id) a un processo, occorre tener conto che si tratta di un intero unico (tipo pid\_t), di valore compreso tra PID\_MIN e PID\_MAX (kern/include/limits.h), definiti in base a \_\_PID\_MIN e \_\_PID\_MAX (kern/include/kern/limits.h). Per l’attribuzione del pid e i passaggi da processo (puntatore a struct proc) a pid e viceversa, occorre realizzare una tabella. Per semplicità, si consiglia di realizzare un vettore di puntatori a struct proc, in cui l’indice corrisponda al pid (in tal caso si consiglia di utilizzare come pid massimo un numero accettabile, ad es. 100), oppure un vettore di coppie (pid, puntatore a processo). Un opportuno campo pid nella struct proc) può invece consentire reperire il pid a partire dal puntatore. Ogni nuovo processo creato va aggiunto alla tabella, generandone il pid, ogni processo distrutto va rimosso dalla tabella (una volta completata una wait/waitpid). La tabella può, ad esempio, essere realizzata come variabile globale in kern/proc/proc.c. Occorre poi realizzare una eventuale funzione sys\_waitpid da chiamare in syscall() (si consiglia di utilizzare lo stesso file, es. proc\_syscalls.c, già usato per la sys\_\_exit). 

**Tabella dei processi e waitpid** 

La common\_prog, funzione interna al kernel, non ha bisogno di gestire il pid di un processo creato (ne ha già il puntatore), quindi non necessita, per attendere la fine del processo creato, del supporto per la waitpid (che richiede di gestire la tabella dei processi). In sostanza, la fine di un processo con \_exit (e system call sys\_\_exit) non necessita, se ad aspettare è il kernel che ne ha il puntatore, della waitpid (con processo identificato da pid), ma è sufficiente la proc\_wait (processo identificato da puntatore).  

La waitpid invece è necessaria per gestione di processi da parte di programmi user. Ad esempio testbin/forktest premetterebbe di verificare il funzionamento di waitpid. Tuttavia, per il funzionamento di tale programma di test occorre realizzare la getpid, che ottiene il pid del processo corrente (puntato da curproc), e la fork, che permette di generare un processo child a livello user. *Questa parte (realizzare getpid e fork) può essere considerata opzionale (si consiglia di provare a realizzarla solo una volta completato con successo il resto del laboratorio): realizzare la getpid è semplice, la fork meno, in quanto si tratta di clonare (duplicare, l’intero address space di un processo (il child è una copia del padre) e di farlo partire correttamente).* 

Un modo semplice per testare (senza bisogno della fork), non direttamente la waitpid, ma l’eventuale sys\_waitpid chiamata in syscall per supportarla, è di ottenere in common\_prog il pid del processo child (mediante sys\_getpid() o altra strategia), e successivamente attendere con sys\_waitpid  anziché proc\_wait.  

Si rappresenta, nella figura che segue, un possibile scenario per verificare la correttezza della catena getpid/waitpid e gestione della tabella dei processi, non direttamente, ma indirettamente tramite le funzioni sys\_getpid() e sys\_waitpid(). 

Kernel main thread  cmd\_progthread thread  (menu) ![](Aspose.Words.b30fc37c-fef2-4d26-92af-0db7bd5b376e.009.png)![ref3]![ref2] becomes 

userprocess (palin) 



|<p>common\_prog(...) { ![](Aspose.Words.b30fc37c-fef2-4d26-92af-0db7bd5b376e.011.png)</p><p>`  `proc =[ proc_create_runprogram ](../../df/d03/proc_8h.html#aae835d06c28841caf3b0690c46012c28)(...); </p><p>`  `result =[ thread_fork(](../../d8/da9/include_2thread_8h.html#afcf81f11876cc2f4556b20c691c583ab)...,  </p><p>`    `proc,[ cmd_progthread,](../../d2/d0a/menu_8c.html#a3499a7a133dc64bf4abc25d460519436) ...); </p><p>`  `pid = sys\_getpid(proc);   </p><p>`  `exit\_code = sys\_waitpid(pid);   ... </p><p>} </p>|||
| - | :- | :- |
|<p>sys\_waitpid(...) { </p><p> ...  </p><p>` `return proc\_wait(...); </p><p>proc\_wait(...) { </p><p>` `wait(...); /\* sem or cv \*/ </p>|||

`  `/\* USER PROCESS \*/ ![](Aspose.Words.b30fc37c-fef2-4d26-92af-0db7bd5b376e.012.png)

`  `exit(...); 



|<p>/\* KERNEL MODE: system call \*/ ![](Aspose.Words.b30fc37c-fef2-4d26-92af-0db7bd5b376e.013.png)syscall(...) { </p><p>}  </p>|
| :- |
|<p>sys\_\_exit(...) { ![](Aspose.Words.b30fc37c-fef2-4d26-92af-0db7bd5b376e.014.png)</p><p>` `signal(...); /\* sem or cv \*/  ... </p><p>` `thread\_exit(); </p><p>} </p>|

[ref1]: Aspose.Words.b30fc37c-fef2-4d26-92af-0db7bd5b376e.002.png
[ref2]: Aspose.Words.b30fc37c-fef2-4d26-92af-0db7bd5b376e.003.png
[ref3]: Aspose.Words.b30fc37c-fef2-4d26-92af-0db7bd5b376e.010.png
