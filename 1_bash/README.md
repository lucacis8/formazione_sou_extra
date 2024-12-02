## Esercizi Bash

### 1. Kernel Version
**Obiettivo**: Verificare se il kernel attualmente in esecuzione è il più recente installato sulla macchina.  

**Descrizione**: Lo script esegue un controllo tra la versione del kernel attivo (`uname -r`) e l'ultima versione disponibile tra quelle installate. Restituisce `TRUE` se il kernel in esecuzione è il più recente, altrimenti `FALSE`.

**Script**:
```bash
#!/bin/bash

# Ottieni la versione del kernel attualmente in esecuzione
current_kernel=$(uname -r)

# Elenca tutte le versioni del kernel installate, considerando solo i pacchetti con numeri di versione
installed_kernels=$(dpkg --list | grep 'linux-image-.*-generic' | awk '{print $2}' | sed 's/linux-image-//' | sort -V)

# Ottieni l'ultima versione del kernel installata
latest_kernel=$(echo "$installed_kernels" | tail -n 1)

# Confronta il kernel in esecuzione con l'ultimo installato
if [ "$current_kernel" = "$latest_kernel" ]; then
    echo "TRUE: Il kernel in esecuzione ($current_kernel) è il più recente installato."
    exit 0
else
    echo "FALSE: Il kernel in esecuzione ($current_kernel) non è il più recente. L'ultima versione installata è $latest_kernel."
    exit 1
fi
```

---

### 2. Find
**Obiettivo**: Utilizzare il comando `find` per cancellare file più vecchi di 30 giorni e schedulare il comando tramite `crontab`.

**Descrizione**: Il comando `find` viene usato per individuare i file in directory come `/var/log` e cancellarli se più vecchi di 30 giorni. Il task viene schedulato ogni lunedì alle 6:30.

**Comando**:
```bash
find /var/log -type f -mtime +30 -exec rm {} \;
```

**Schedulazione in crontab**:
```bash
30 6 * * 1 find /var/log -type f -mtime +30 -exec rm {} \;
```

---

### 3. AWK
**Obiettivo**: Stampare il terzo campo di un file CSV solo se il secondo campo contiene la stringa `banana`.

**Comando**:
```bash
awk -F, '/banana/ {print $3}' file.cvs
```

**Esempio di input**:
```bash
apple,banana,strawberry
pear,peach,lemon
banana,kiwi,orange
```

**Output**:
```bash
strawberry
orange
```

---

### 4. REGEX
**Obiettivo**: Scrivere un'espressione regolare che matchi indirizzi IPv4 in decimal dotted notation.

**Regex**:
```regex
^((25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])\.){3}(25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])$
```

**Descrizione**:
- 25[0-5]: Matcha numeri da 250 a 255.
- 2[0-4][0-9]: Matcha numeri da 200 a 249.
- 1[0-9][0-9]: Matcha numeri da 100 a 199.
- [1-9]?[0-9]: Matcha numeri da 0 a 99.
- Il gruppo {3} assicura che ci siano tre sezioni separate da punti, seguito dall’ultima sezione che completa l’indirizzo IPv4.

---

### 5. Portscanner
**Obiettivo**: Scrivere uno script Bash che funzioni come port scanner utilizzando un ciclo e il comando `nc` (NetCat).

**Descrizione**: Lo script scansiona un range di porte su un indirizzo IP/host target specificato tramite argomenti. È inclusa la sanificazione degli input.

**Script**:
```bash
#!/bin/bash

# Verifica che siano passati i parametri target e range di porte
if [ $# -ne 3 ]; then
  echo "Uso: $0 <indirizzo IP> <porta_inizio> <porta_fine>"
  exit 1
fi

# Parametri passati da riga di comando
TARGET_IP=$1
START_PORT=$2
END_PORT=$3

# Ciclo sulle porte specificate
for ((port=$START_PORT; port<=$END_PORT; port++)); do
  # Utilizza nc in modalità verbose (-v) con timeout di 1 secondo (-z -w 1)
  nc -z -v -w 1 $TARGET_IP $port 2>&1 | grep -q "succeeded"
  
  # Se nc ha trovato una porta aperta, mostra il numero
  if [ $? -eq 0 ]; then
    echo "Porta $port è aperta su $TARGET_IP"
  fi
done
```

**Esecuzione**:
```bash
./portscanner.sh 192.168.1.1 1 65535
```

**Output**:
```bash
Porta 22 è aperta su 192.168.1.1
Porta 80 è aperta su 192.168.1.1
Porta 443 è aperta su 192.168.1.1
```

---

### Conclusione

Questi esercizi esplorano diverse funzionalità della Bash e di comandi utili come `find`, `awk`, e `nc`, oltre a mettere in pratica la manipolazione di espressioni regolari e la programmazione shell per task di automazione e gestione del sistema.

