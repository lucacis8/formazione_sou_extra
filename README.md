# Formazione Sourcesence - DevOps Academy - Track Extra

Questa repository contiene il materiale relativo agli esercizi svolti nella Track Extra dell'Academy. Gli esercizi proposti durante questa track hanno l'obiettivo di approfondire conoscenze e competenze in aree avanzate dell'amministrazione di sistemi, gestione di container, automazione tramite Ansible, e interazione con le API di Jenkins.

## Tecnologie e Metodologie Utilizzate

Durante la Track Extra, sono state utilizzate le seguenti tecnologie e metodologie:

### 1. **Scripting Bash**
   - **Obiettivo**: Automazione di operazioni amministrative su sistemi Linux attraverso l'utilizzo di script Bash.
   - **Tecnologie**: Bash, comandi Linux (`find`, `awk`, `nc`, ecc.), cron per la schedulazione automatica.
   - **Descrizione**: Gli esercizi Bash sono stati progettati per migliorare la gestione dei sistemi Linux tramite l'automazione di operazioni quotidiane, come la gestione dei file di log, l'esecuzione di scansioni di porte e la manipolazione di dati tramite espressioni regolari e AWK.

### 2. **Containerizzazione e Namespaces**
   - **Obiettivo**: Comprendere il funzionamento dei container Linux e l'isolamento tramite namespaces.
   - **Tecnologie**: Linux namespaces (PID, MNT, NET, USER, IPC, TIME), Docker.
   - **Descrizione**: In questi esercizi, è stato esplorato come i namespaces di Linux possano isolare i processi e risorse di sistema, una parte fondamentale per la gestione e sicurezza dei container. È stato inoltre esaminato come i cgroups (control groups) possano limitare l'accesso alle risorse (CPU, memoria, I/O) per migliorare l'efficienza e la gestione delle risorse nei container.

### 3. **Automazione con Ansible**
   - **Obiettivo**: Semplificare la gestione dei sistemi e delle configurazioni tramite Ansible, utilizzando playbook per automatizzare la gestione di pacchetti, utenti e file di configurazione.
   - **Tecnologie**: Ansible, Jinja2, YAML.
   - **Descrizione**: Gli esercizi su Ansible si sono concentrati sull'uso di strutture dati avanzate (liste e dizionari) per automatizzare operazioni di configurazione dei sistemi, come l'installazione di pacchetti e la creazione di utenti, e sull'uso dei template Jinja per generare file dinamici.

### 4. **REST API di Jenkins**
   - **Obiettivo**: Utilizzare le REST API di Jenkins per automatizzare la creazione e gestione di agenti su un server Jenkins.
   - **Tecnologie**: Jenkins, REST API, curl.
   - **Descrizione**: Un'importante parte del materiale riguarda l'automazione della creazione di nodi agenti Jenkins, utilizzando le REST API per gestire in modo programmatico il sistema di build Jenkins.

## Come Utilizzare

Per utilizzare il materiale di questa repository, segui questi passaggi:

1. **Clonare la Repository**:
   Puoi clonare questa repository sul tuo ambiente locale eseguendo il comando:
   ```bash
   git clone https://github.com/lucacis8/formazione_sou_extra
   ```

2. **Esecuzione degli Script**:
   Ogni esercizio contiene script o playbook che possono essere eseguiti direttamente dal terminale. Assicurati di avere i permessi necessari per eseguire gli script (ad esempio, usa `chmod +x script.sh` per i file Bash).

## Conclusione

Questa repository è strettamente legata alla Track Extra dell'Academy, una parte fondamentale del percorso di formazione aziendale. Gli esercizi inclusi riflettono le competenze e le tecnologie avanzate che vengono approfondite durante il corso, con l'intento di migliorare l'automazione dei processi, l'amministrazione di sistemi e l'interazione con strumenti di gestione come Jenkins.
