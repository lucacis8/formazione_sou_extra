# Esercizio Jenkins API: Automatizzare la Creazione di un Agent Node

## Scopo

L’obiettivo di questo esercizio è imparare a utilizzare le REST API di Jenkins per automatizzare la configurazione di un agent node. Utilizzeremo i dati già definiti nel file `provision.yml` per completare il processo e integrarlo in maniera automatizzata.

---

## Prerequisiti

1. **Ambiente configurato**:
- Una VM Rocky Linux 9 configurata tramite Vagrant e Ansible.
- Jenkins Master e Slave eseguiti come container Docker, già configurati nel file `provision.yml`.

2. **Accesso a Jenkins**:
- Interfaccia web disponibile all’indirizzo `http://localhost:8080`.
- Credenziali dell’amministratore di Jenkins (utente `admin` e password iniziale).

3. **Software necessario**:
- **curl**: Per interagire con le REST API.
- **jq**: Per manipolare le risposte JSON.
- **Ansible**: Per automatizzare il processo.

---

## Configurazione dell’Ambiente

1. **Avviare la VM**:
   ```bash
   vagrant up
   ```

Jenkins sarà disponibile all’indirizzo `http://localhost:8080`.

2. **Recuperare la password iniziale di Jenkins**:
   ```bash
   vagrant ssh
   sudo cat /var/lib/docker/volumes/jenkins_home/_data/secrets/initialAdminPassword
   ```

Utilizza questa password per accedere all’interfaccia web con username `admin` e completare la configurazione iniziale.

3. **Verifica della configurazione dell'agent**:

Nel file `provision.yml`, il container Docker per l’agent è definito come segue:
```bash
- name: Esegui il container Jenkins Slave
  docker_container:
    name: jenkins_slave
    image: jenkins/inbound-agent
    state: started
    restart_policy: always
    user: root
    networks:
      - name: jenkins_network
        ipv4_address: 172.20.0.3
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - jenkins_home:/var/jenkins_home
    env:
      JENKINS_URL: http://172.20.0.2:8080
      JENKINS_AGENT_NAME: "slave"
      JENKINS_AGENT_WORKDIR: "/home/jenkins"
      JENKINS_SECRET: "jenkins_secret"
```

Questi dati verranno utilizzati per configurare l’agent tramite le REST API.

---

## Procedura per l’Esercizio

### 1. Ottenere il Crumb

Il primo passo è ottenere un `crumb` di sicurezza, necessario per effettuare richieste API.

- Esegui una richiesta `GET`:
   ```bash
   curl -u admin:<PASSWORD> -c cookies.txt -s \
     'http://192.168.56.10:8080/crumbIssuer/api/json' | jq '.crumb'
   ```

- Sostituisci `<PASSWORD>` con la password dell’amministratore.
- Salva i cookie in `cookies.txt` per mantenere la sessione.
- Memorizza il valore del `crumb`.

### 2. Creare un API Token

Crea un token API per interagire con Jenkins senza usare la password.

- Esegui una richiesta POST:
   ```bash
   curl -u admin:<PASSWORD> -b cookies.txt -H "Jenkins-Crumb:<CRUMB>" -X POST \
     --data-urlencode "newTokenName=myToken" \
     'http://192.168.56.10:8080/me/descriptorByName/jenkins.security.ApiTokenProperty/generateNewToken'
   ```

- Sostituisci `<CRUMB>` con il valore ottenuto in precedenza.
- Memorizza il token generato.

### 3. Creare l'Agent Node

Configura un nuovo agent utilizzando i dati definiti nel file `provision.yml`.

- Esegui una richiesta `POST` per creare il node:
```bash
curl -u admin:<TOKEN> -b cookies.txt -H "Jenkins-Crumb:<CRUMB>" -X POST \
  --data-urlencode "name=slave" \
  --data-urlencode "type=hudson.slaves.DumbSlave" \
  --data-urlencode "json={
    \"name\": \"slave\",
    \"nodeDescription\": \"Agent configurato automaticamente\",
    \"numExecutors\": \"1\",
    \"remoteFS\": \"/home/jenkins\",
    \"labelString\": \"docker\",
    \"mode\": \"EXCLUSIVE\"
  }" \
  'http://192.168.56.10:8080/computer/doCreateItem'
```

- `name`: Usa il nome slave come definito nel file `provision.yml`.
- `remoteFS`: Specifica `/home/jenkins`, il percorso di lavoro dell’agent.

### 4. Recuperare il Secret

Recupera il secret dell’agent necessario per la registrazione al master.

- Esegui una richiesta `GET`:
   ```bash
   curl -u admin:<TOKEN> -b cookies.txt -s \
     'http://192.168.56.10:8080/computer/slave/jenkins-agent.jnlp'
   ```

- Memorizza il secret ottenuto.

---

## Integrazione nel Playbook Ansible

Automatizza l’intero processo aggiungendo i seguenti task al file `provision.yml`.

### Ottenere il Crumb
```bash
- name: Ottenere il Crumb di Jenkins
  uri:
    url: "http://192.168.56.10:8080/crumbIssuer/api/json"
    method: GET
    user: admin
    password: "{{ jenkins_password }}"
    return_content: yes
  register: crumb_response

- set_fact:
    jenkins_crumb: "{{ crumb_response.json.crumb }}"
```

### Creare un Token API
```bash
- name: Creare un token API Jenkins
  uri:
    url: "http://192.168.56.10:8080/me/descriptorByName/jenkins.security.ApiTokenProperty/generateNewToken"
    method: POST
    user: admin
    password: "{{ jenkins_password }}"
    headers:
      Jenkins-Crumb: "{{ jenkins_crumb }}"
    body_format: form-urlencoded
    body:
      newTokenName: "myToken"
    return_content: yes
  register: token_response

- set_fact:
    jenkins_api_token: "{{ token_response.json.tokenValue }}"
```

### Creare l'Agent Node
```bash
- name: Creare l'Agent Node
  uri:
    url: "http://192.168.56.10:8080/computer/doCreateItem"
    method: POST
    headers:
      Jenkins-Crumb: "{{ jenkins_crumb }}"
    body_format: form-urlencoded
    body:
      name: "slave"
      type: "hudson.slaves.DumbSlave"
      json: |
        {
          "name": "slave",
          "nodeDescription": "Agent configurato automaticamente",
          "numExecutors": "1",
          "remoteFS": "/home/jenkins",
          "labelString": "docker",
          "mode": "EXCLUSIVE"
        }
```

---

## Esecuzione

1. Modifica le variabili nel file `provision.yml`:

- `jenkins_password`: Password iniziale o aggiornata dell’amministratore.

2. Esegui il playbook:
   ```bash
   ansible-playbook provision.yml
   ```

---

## Conclusione

Seguendo questa guida, puoi automatizzare la configurazione di un agent node utilizzando i dati già definiti nel tuo file `provision.yml`, semplificando il processo di provisioning e integrazione di Jenkins.
