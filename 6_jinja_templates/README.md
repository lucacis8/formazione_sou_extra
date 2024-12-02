# Esercizio Ansible: Jinja Templates

## Scopo
L’obiettivo dell’esercizio è imparare ad utilizzare i **template Jinja** all’interno di un playbook Ansible per generare file configurabili dinamicamente. In particolare, verranno realizzati due task principali:
1. Aggiungere configurazioni personalizzate al file `/etc/security/limits.conf` in base all'ambiente.
2. Modificare il file `/etc/security/access.conf` per aggiungere una whitelist di utenti autorizzati.
3. [Opzionale] Utilizzare Jinja templates per:
   - Dockerfile
   - Deployment Kubernetes
   - Configurazione Virtual Host Apache

---

## Struttura del progetto
La directory del progetto è organizzata come segue:
```plaintext
6_jinja_templates/
├── templates/
│   ├── limits.conf.j2          # Template Jinja per /etc/security/limits.conf
│   ├── Dockerfile.j2           # Template Jinja per il Dockerfile
│   ├── deployment.yaml.j2      # Template Jinja per Deployment Kubernetes
│   └── virtualhost.conf.j2     # Template Jinja per Virtual Host Apache
├── playbook.yml                # Playbook Ansible
└── inventory.yml               # Inventario degli host
```

---

## Funzionalità del Playbook
### 1. **Configurazione di `/etc/security/limits.conf`**
- Imposta il numero massimo di file aperti (`nofile`) per un utente specifico.
- Il valore di nofile dipende dall’ambiente:
  - **Produzione**: 10.000 file aperti.
	- **Collaudo/Sviluppo**: 1.000 file aperti.
- Il file viene generato dinamicamente tramite il template Jinja `limits.conf.j2`.

### 2. **Modifica di `/etc/security/access.conf`**

- Aggiunge una whitelist di utenti autorizzati ad accedere al sistema.
- Gli utenti della whitelist vengono aggiunti prima della riga `- : ALL : ALL`, che nega l’accesso agli altri utenti.

### 3. **[Opzionale] Utilizzo di altri Jinja templates**

- **Dockerfile**: Configurazione dinamica per immagini Docker.
- **Deployment Kubernetes**: File YAML parametrizzato per creare un deployment.
- **Virtual Host Apache**: Configurazione di un Virtual Host flessibile.

---

## Come Utilizzare il Progetto
1. **Prerequisiti**
   - Ansible installato sulla macchina di controllo.
	 - Accesso amministrativo (sudo) alle macchine target.
	 - Creare la directory del progetto e posizionare i file nelle rispettive posizioni.

2. **Personalizzazione**
   - Modificare il file `playbook.yml` per adattarlo alle esigenze:
	   - `environment`: Specifica l’ambiente (`production`, `test`, `development`).
	   - `user`: Nome dell’utente per il quale configurare i limiti.
	   - `whitelist_users`: Lista degli utenti da aggiungere alla whitelist.

3. **Esecuzione**
   - Eseguire il playbook con il comando:
     ```bash
     ansible-playbook -i inventory.yml playbook.yml
     ```

4. **Output Atteso**
   - Il file `/etc/security/limits.conf` conterrà:
     ```bash
     myuser  -  nofile  10000  # (o 1000 in base all'ambiente)
     ```
   - Il file `/etc/security/access.conf` conterrà:
     ```bash
     + : alice : ALL
     + : bob : ALL
     + : charlie : ALL
     - : ALL : ALL
     ```

---

## File di Progetto
### 1. **Template Jinja: `limits.conf.j2`**
```jinja
# {{ ansible_managed }}
# Configurazione dei limiti per l'utente {{ user }}
{{ user }}  -  nofile  {{ max_open_files }}
```

### 2. **Template Jinja: `Dockerfile.j2`**
```bash
# {{ ansible_managed }}
FROM {{ base_image }}

RUN apt-get update && apt-get install -y {{ ' '.join(packages) }}

CMD ["{{ default_command }}"]
```

Esempio di variabili nel playbook:
```bash
vars:
  base_image: ubuntu:20.04
  packages:
    - curl
    - vim
  default_command: /bin/bash
```

### 3. **Template Jinja: `deployment.yaml.j2`**
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ app_name }}
spec:
  replicas: {{ replicas }}
  selector:
    matchLabels:
      app: {{ app_name }}
  template:
    metadata:
      labels:
        app: {{ app_name }}
    spec:
      containers:
      - name: {{ app_name }}
        image: {{ container_image }}
        ports:
        - containerPort: {{ container_port }}
```

Esempio di variabili nel playbook:
```bash
vars:
  app_name: my-app
  replicas: 3
  container_image: my-app:latest
  container_port: 8080
```

### 4. **Template Jinja: `virtualhost.conf.j2`**
```bash
<VirtualHost *:{{ port }}>
    ServerName {{ server_name }}
    DocumentRoot {{ document_root }}
    <Directory {{ document_root }}>
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog {{ log_dir }}/error.log
    CustomLog {{ log_dir }}/access.log combined
</VirtualHost>
```

Esempio di variabili nel playbook:
```bash
vars:
  port: 80
  server_name: example.com
  document_root: /var/www/example
  log_dir: /var/log/apache2/example
```

---

## Conclusione

Questo esercizio dimostra come i **Jinja templates** possano semplificare la generazione di configurazioni dinamiche e migliorare la flessibilità dei playbook Ansible. I file opzionali forniscono esempi pratici di utilizzo avanzato in diversi contesti (Docker, Kubernetes, Apache).
