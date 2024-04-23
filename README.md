# LAB1

## Build di OS161

1. Assicurati di essere nella directory principale del progetto OS161.

2. Esegui il comando `sys161 kernel` per avviare la build del kernel.

3. Verifica che la build sia stata eseguita correttamente controllando la presenza dei file.

4. In caso di errori durante la build, correggili e ripeti il processo.

## Debugging

1. Per avviare OS161 in modalità debug, utilizza il comando da VSC `Run Task` dalla sezione Terminale.

2. Durante il debug, utilizza gli shortcut da tastiera o i comandi `run state` oppure `F5` per avanzare nell'esecuzione.

3. Assicurati che il debugger sia funzionante impostando un breakpoint sulla funzione chiamata `kmain` e utilizzando il comando `go` per continuare l'esecuzione.

4. Utilizza la finestra del debugger per monitorare variabili, stack e breakpoint.

## Debugging Avanzato

1. Se desideri avviare OS161 in modalità debug avanzata, esegui il comando `start debug`.

2. Durante il debug avanzato, potresti notare che, consultando main, troverai già indicazioni dettagliate sotto il titolo "step by step good".

3. Nel processo di esecuzione, potresti incontrare un prompt che ti chiede conferma dopo aver eseguito determinate azioni.

## Abilitazione e Disabilitazione delle Funzionalità

1. Per rendere facilmente disabilitabili alcune funzionalità nel codice, evita di utilizzare strategie basate su commenti e condizioni complesse.

2. Invece, usa gli switch di compilazione come `#if` per abilitare o disabilitare parti del codice in base a determinati simboli.

3. Assicurati di verificare che sia possibile disabilitare correttamente le funzionalità, testando il codice in diverse configurazioni.

## Risoluzione dei Problemi

1. Durante lo sviluppo, potresti incontrare problemi come errori di accesso a disco.

2. Se incontri questo tipo di errore, potrebbe essere dovuto al fatto che hai avviato più istanze di OS161 contemporaneamente nella stessa directory.

3. Assicurati di non avere più di una sessione di OS161 attiva contemporaneamente nella stessa cartella per evitare conflitti e errori.

4. Presta attenzione ai simboli non definiti nel codice, poiché il compilatore potrebbe assumere automaticamente che siano falsi, disabilitando involontariamente alcune parti del codice.

## Configurazione Avanzata

1. Per abilitare o disabilitare simboli nel codice, evitando modifiche dirette ai file sorgente, utilizza il file di configurazione "conf.kern".

2. Aggiungi o rimuovi simboli definendo o commentando le relative direttive `#define` nel file di configurazione. Esempio:

    ```makefile
    # NEW PDS WORK LAB 1
    defoption hello
    optfile   hello    main/hello.c 
    ```

3. Assicurati di includere il file di configurazione corretto nei tuoi sorgenti per attivare o disattivare le funzionalità desiderate.

# TEST

# Guida ai Test di OS161

## Introduzione

In questa guida, esploreremo i test disponibili per OS161, un sistema operativo educativo. I test OS161 consentono di verificare il corretto funzionamento del sistema operativo e delle sue funzionalità.

## Tipi di Test

I test di OS161 sono suddivisi in diverse categorie:

### Test del Kernel

I test del kernel eseguono funzioni direttamente nel kernel di OS161, testando le sue operazioni di base. Ecco alcuni esempi di test del kernel:

- `test1`: tt1
- `test2`: tt2
- `test3`: tt3
- `test7`: tt7

### Test delle Operazioni del Kernel

I test delle operazioni del kernel eseguono operazioni specifiche nel kernel e valutano il loro comportamento. Ad esempio:

- `opTest1`: Esegue operazioni nel kernel.
- `opTest5`: Testa le operazioni di I/O.

### Test dei Processi Utente

I test dei processi utente eseguono programmi utente su OS161 e ne valutano il comportamento. Ad esempio:

- `userTest1`: Testa un programma utente di base.
- `userTest55`: Testa un programma utente che produce output su console.

## Esecuzione dei Test

Per eseguire un test in OS161, seguire questi passaggi:

1. Accedere alla directory del progetto OS161.
2. Utilizzare il comando appropriato per eseguire il test desiderato. Ad esempio, per eseguire il test 3, utilizzare il comando `tt3`.

## Analisi dei Risultati

Dopo aver eseguito un test, analizzare attentamente i risultati per identificare eventuali errori o comportamenti anomali. Utilizzare gli strumenti di debug forniti da OS161 per individuare e risolvere i problemi.

## Ulteriori Informazioni sui Test

Per una comprensione più approfondita dei test di OS161, ecco alcune informazioni aggiuntive:

- Quando si debugga un thread, è possibile mettere un breakpoint su di esso per intercettare il suo comportamento. Tuttavia, è importante disabilitare eventuali altri breakpoint per concentrarsi sul thread specifico.
- Durante la programmazione concorrente, i thread possono alternarsi rapidamente, causando un ordine non deterministico delle stampe. Utilizzare gli strumenti di debug per analizzare il comportamento dei thread e identificare eventuali problemi.


# ALTERNATIVA ALL'UTILIZZO DI RUN TASK SU TERMINAL DI VSC - LINEA DI COMANDO

## Configurazione e Compilazione del Kernel OS161

### Passaggio 1: Navigare alla Cartella di Configurazione

1. Utilizzare il terminale o l'interprete dei comandi per accedere alla directory del kernel OS161.
   ```bash
   cd src/kern/conf
   ```

### Passaggio 2: Configurare il Kernel

1. Eseguire il comando `./config` seguito dal nome desiderato per la versione del kernel. Ad esempio, se si desidera configurare una versione chiamata "HELLO", il comando sarà:
   ```bash
   ./config HELLO
   ```

### Passaggio 3: Compilare il Kernel

1. Navigare alla directory relativa alla versione del kernel configurata. Questa directory apparirà in `src/kern/compile`.
   ```bash
   cd ../../compile/HELLO
   ```

2. Eseguire il comando `bmake depend` per gestire le dipendenze.
   ```bash
   bmake depend
   ```

3. Successivamente, eseguire il comando `bmake` per compilare il kernel.
   ```bash
   bmake
   ```

4. Infine, eseguire il comando `bmake install` per installare il kernel compilato.
   ```bash
   bmake install
   ```

### Passaggio 4: Eseguire il Kernel

1. Risalire fino alla cartella principale del progetto OS161, ed entrare in `/root`.
   ```bash
   cd ../../../..
   ```

2. Una volta nella cartella principale, lanciare il kernel OS161 utilizzando il comando `sys161 kernel`.
   ```bash
   sys161 kernel
   ```