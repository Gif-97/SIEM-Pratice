# Documentazione: Installazione e Configurazione di Wazuh

Questo documento descrive i passaggi eseguiti per integrare Wazuh nel mio progetto di laboratorio di cybersecurity. Ogni step è dettagliato, incluso come ho risolto i problemi emersi durante il processo. La configurazione è stata fatta tenendo conto del contesto del progetto e degli obiettivi di monitoraggio e sicurezza.

## **1. Configurazione della VM per Wazuh**

1. **Creazione della macchina virtuale**:
    - Ho configurato una nuova VM su VMware con le seguenti specifiche:
      - **Sistema operativo**: Ubuntu Server 22.04 LTS
      - **RAM**: 6 GB
      - **CPU**: 2 core
      - **Disco**: 50 GB
      - **Scheda di rete**: LAN SEGMENT > DMZ LAN

    <img src="(https://github.com/Gif-97/SIEM-Pratice/blob/main/media/configurazioneVmPerUbuntuServerWazuh.png)" width=450 height=250>


2. **Installazione di Ubuntu**:
    - Ho scaricato l'ISO di Ubuntu Server 22.04.01 dal sito ufficiale.
    - Durante l'installazione, ho configurato un hostname (`wazuhman`) e un IP statico per la rete DMZ (192.168.3.11).
    - Ho creato un utente amministratore con privilegi sudo.

3. **Aggiornamento del sistema**:
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

4. **Installazione di pacchetti di base**:
    ```bash
    sudo apt install curl wget nano ufw -y
    ```

5. **Configurazione del firewall UFW**:
    - Permesso per SSH:
      ```bash
      sudo ufw allow 22/tcp
      sudo ufw enable
      ```

---

## **2. Installazione di Wazuh Manager**

1. **Aggiunta del repository Wazuh**:
    - Ho scaricato e aggiunto la chiave GPG:
      ```bash
      curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh-keyring.gpg
      ```
    - Ho configurato il repository:
      ```bash
      echo "deb [signed-by=/usr/share/keyrings/wazuh-keyring.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
      ```
    - Ho aggiornato la lista dei pacchetti:
      ```bash
      sudo apt update
      ```

2. **Installazione di Wazuh Manager**:
    ```bash
    sudo apt install wazuh-manager -y
    ```

3. **Configurazione del firewall per Wazuh**:
    - Apertura delle porte:
      ```bash
      sudo ufw allow 1514/tcp
      sudo ufw allow 55000/tcp
      sudo ufw reload
      ```

4. **Verifica del servizio**:
    - Ho verificato che Wazuh Manager fosse attivo:
      ```bash
      sudo systemctl status wazuh-manager
      ```

---

## **3. Configurazione di Wazuh Agent su Windows**

1. **Scaricamento e installazione dell'agente**:
    - Ho scaricato l'installer per Windows dal sito ufficiale di Wazuh.
    - Durante l'installazione, ho configurato i percorsi predefiniti.

2. **Generazione della chiave di autenticazione**:
    - Sul server Wazuh Manager, ho eseguito:
      ```bash
      sudo /var/ossec/bin/manage_agents
      ```
      - Ho selezionato `A` per aggiungere un nuovo agente.
      - Ho fornito il nome `WindowsAgent` e l'IP della macchina Windows.
      - Ho selezionato `E` per ottenere l'elenco degli agenti e selezionare quello desiderato tramite il suo ID per visualizzarne la key.
      - Ho copiato la chiave di autenticazione generata.

3. **Configurazione dell'agente Windows**:
    - Nel GUI dell'agente:
      - **Manager IP**: Ho inserito l'IP del Wazuh Manager.
      - **Authentication Key**: Ho incollato la chiave generata.
    - Ho cliccato su `Save` e poi `Refresh` per avviare il servizio.

---

## **4. Verifica della Connessione**

1. **Verifica della connessione agent-manager**:
    - Sul Manager:
      ```bash
      sudo /var/ossec/bin/manage_agents
      ```
      - Ho selezionato `L` per verificare che l'agente fosse attivo.

2. **Controllo dei log**:
    ```bash
    sudo tail -f /var/ossec/logs/ossec.log
    ```

---

## **5. Simulazione di un attacco**

1. **Esecuzione di un attacco simulato**:
    - Ho utilizzato `nmap` per eseguire una scansione:
      ```bash
      nmap -A <IP_Target>
      ```
    - Ho verificato che l'attività fosse registrata nei log di Wazuh.

2. **Analisi degli alert**:
    - Ho verificato gli alert generati nella dashboard di Wazuh e documentato i dettagli rilevati.

---

## **6. Integrazione nel progetto**

- **Topologia del progetto**:
  - Wazuh Manager è stato posizionato nella DMZ e configurato per monitorare tutte le macchine del laboratorio.
  - Gli agenti sono stati installati su macchine Linux e Windows per raccogliere i log.

- **Regole firewall**:
  - Ho configurato regole per limitare l'accesso al Wazuh Manager alle sole macchine autorizzate.

---

## **7. Conclusione**

Questo progetto ha permesso di integrare Wazuh come soluzione SIEM per monitorare le attività del laboratorio di cybersecurity. La documentazione dettagliata e i test eseguiti rafforzano la validità del setup come esempio pratico per un portfolio professionale in ambito Red Team/Blue Team.

---
