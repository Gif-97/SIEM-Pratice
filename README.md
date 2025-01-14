# Implementazione di Wazuh in un Home Lab di Cybersecurity

## Introduzione
In questo documento descrivo i passaggi che ho seguito per implementare e configurare Wazuh nel mio Home Lab di Cybersecurity. Questo progetto include l'integrazione di Wazuh come SIEM nella DMZ configurata con pfSense, l'installazione di agent su macchine Windows e Kali Linux, la simulazione di attacchi per il monitoraggio delle attività e l'analisi dei log. Il mio obiettivo è dimostrare competenze pratiche in cybersecurity.

## Requisiti
- **Hardware e software:**
  - Server Ubuntu in una VM
  - pfSense configurato con DMZ
  - Macchine virtuali Windows 11 e Kali Linux nella LAN
- **Strumenti e pacchetti:**
  - OpenSSL per connessioni remote
  - Wazuh Indexer, Server e Dashboard
  - Wazuh agent per Windows e Linux

---

## Configurazione del Server Ubuntu nella DMZ

1. **Creazione della VM e collegamento alla DMZ:**
   - Ho configurato una DMZ separata utilizzando pfSense.
   - Ho assegnato alla VM Ubuntu Server l'indirizzo IP `192.168.3.12` nella rete DMZ.

2. **Preparazione del sistema operativo:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install openssl -y
   ```

3. **Installazione di Wazuh:**

   ### Installazione di Wazuh Indexer
   - Ho scaricato il file di configurazione e il file di installazione:
     ```bash
     curl -sO https://packages.wazuh.com/4.10/wazuh-install.sh
     curl -sO https://packages.wazuh.com/4.10/config.yml
     ```
   - Ho modificato il file `config.yml` sostituendo i nomi dei nodi e gli IP con quelli corrispondenti alla mia infrastruttura:
     ```yaml
     nodes:
       # Wazuh indexer nodes
       indexer:
         - name: node-1
           ip: "192.168.3.12"

       # Wazuh server nodes
       server:
         - name: wazuh-1
           ip: "192.168.3.12"

       # Wazuh dashboard nodes
       dashboard:
         - name: dashboard
           ip: "192.168.3.12"
     ```
   - Ho generato i file di configurazione per il cluster e gli altri componenti:
     ```bash
     bash wazuh-install.sh --generate-config-files
     ```
   - Ho copiato il file `wazuh-install-files.tar` generato sui server pertinenti.
   - Ho installato il Wazuh Indexer sul nodo configurato:
     ```bash
     bash wazuh-install.sh --wazuh-indexer node-1
     ```
   - Ho inizializzato il cluster:
     ```bash
     bash wazuh-install.sh --start-cluster
     ```
   - Per verificare che l'installazione fosse riuscita, ho utilizzato i seguenti comandi:
     ```bash
     tar -axf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt -O | grep -P "'admin'" -A 1
     curl -k -u admin:<ADMIN_PASSWORD> https://192.168.3.12:9200
     ```

   #### Problemi e Soluzioni
   - **Errore di connessione durante l'inizializzazione del cluster:**
     1. Ho verificato che le porte necessarie (9200 e altre richieste dal cluster) fossero aperte su pfSense.
     2. Ho controllato la sintassi del file `config.yml` per errori.

   ![Configurazione file Config.yml](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/1configurazioneFileConfigYmlPerInstallazioneWazuhIndEMan.png)
   ![Installazione e verifica cluster 1](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/2confermaFunzionamentoCluster.png)
   ![Installazione e verifica cluster](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/configurazioneVmPerUbuntuServerWazuh.png)

   ### Installazione di Wazuh Server
   - Ho scaricato il file di installazione:
     ```bash
     curl -sO https://packages.wazuh.com/4.10/wazuh-install.sh
     ```
   - Ho installato il server Wazuh:
     ```bash
     bash wazuh-install.sh --wazuh-server wazuh-1
     ```
   - Ho verificato che il file `wazuh-install-files.tar` fosse disponibile nella directory di lavoro durante l'installazione.

   #### Problemi e Soluzioni
   - **Certificati SSL mancanti:**
     1. Ho verificato che i certificati generati nella fase iniziale fossero corretti e disponibili.
     2. Ho controllato il file `ossec.conf` per configurazioni errate.

    ![Installazione e configurazione server](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/3installazioneServer.png)

   ### Installazione di Wazuh Dashboard
   - Ho scaricato il file di installazione:
     ```bash
     curl -sO https://packages.wazuh.com/4.10/wazuh-install.sh
     ```
   - Ho installato il dashboard:
     ```bash
     bash wazuh-install.sh --wazuh-dashboard dashboard
     ```
   - Ho utilizzato la porta predefinita `443` per accedere al dashboard.
   - Ho creato un utente amministratore e recuperato la password generata automaticamente:
     ```bash
     tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
     ```

   #### Problemi e Soluzioni
   - **Accesso negato:**
     1. Ho verificato che il file `wazuh-passwords.txt` fosse correttamente generato.
     2. Ho risolto problemi di rete assicurandomi che la porta 443 fosse aperta su pfSense.

    ![Installazione e dashboard](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/4installazioneDashboard.png)
    ![Accesso interfaccia web](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/4.1wazuhURL.png)

---

## Installazione degli Agent

### Agent su Windows 11
1. Ho scaricato il pacchetto di installazione dal sito ufficiale di Wazuh.
2. Ho eseguito l'installer tramite l'interfaccia grafica, seguendo questi passaggi:
   - Ho selezionato "Install Wazuh Agent".
   - Ho specificato l'indirizzo del server Wazuh: `192.168.3.12`.
   - Ho configurato la comunicazione tramite certificati SSL.
3. Ho aggiunto l'agent al Wazuh Manager utilizzando il dashboard, specificando:
   - Nome agente: **Windows11Agent**.
   - IP agente: **192.168.2.10**.
4. Ho esportato la chiave generata dal Manager per l'agente e l'ho inserita tramite la GUI del client Windows.
5. Ho completato l'autenticazione tra il server e l'agent.
6. Ho avviato il servizio tramite "Services" di Windows.

#### Problemi e Soluzioni
- **Connessione rifiutata dal server:**
  1. Ho verificato che il firewall di Windows consentisse il traffico verso il server Wazuh sulla porta appropriata.
  2. Ho aggiunto un'eccezione nel firewall per il processo del Wazuh Agent.
  3. Ho riavviato il servizio Wazuh Agent tramite il pannello "Services".

    ![Regole FW](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/5regoleFW.png)
    ![Regole FW2](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/5.1regoleFW2.png)
    ![Configurazione Windows Agent](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/6confAgentWin.png)
    ![Avvio Windows Agent](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/7configurazioneAgentWin.png)

### Agent su Kali Linux
1. Ho aggiunto il repository di Wazuh:
   ```bash
   curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
   echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
   ```
2. Ho installato l'agent utilizzando la variabile `WAZUH_MANAGER` per automatizzare la registrazione:
   ```bash
   WAZUH_MANAGER="192.168.3.12" apt-get install wazuh-agent
   ```
3. Ho configurato ulteriormente l'agent per includere nome specifico e gruppo:
   ```bash
   nano /var/ossec/etc/ossec.conf
   ```
4. Ho avviato e abilitato il servizio:
   ```bash
   systemctl daemon-reload
   systemctl enable wazuh-agent
   systemctl start wazuh-agent
   ```
5. Ho disabilitato gli aggiornamenti automatici per evitare incompatibilità:
   ```bash
   sed -i "s/^deb/#deb/" /etc/apt/sources.list.d/wazuh.list
   apt-get update
   ```

#### Problemi e Soluzioni
- **Errore durante l'avvio del servizio:**
  1. Ho verificato che il file di configurazione `/var/ossec/etc/ossec.conf` fosse correttamente configurato.
  2. Ho corretto i permessi del file di configurazione con il comando:
     ```bash
     chmod 640 /var/ossec/etc/ossec.conf
     ```

    ![Configurazione file ossec](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/8configurazioneOssec.png)
    ![Status Kali](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/10statusAgentKali.png)
    ![Lista Agent](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/listaAgentManager.png)
 

---

## Simulazione di Attacchi

### Attacchi eseguiti
1. **Creazione di vulnerabilità su Windows 11 (IP: 192.168.2.10):**
   - Ho disabilitato Windows Defender.
   - Ho aperto porte non necessarie tramite firewall.

   > **SCREEN:** Inserire screenshot delle modifiche al firewall di Windows e della disabilitazione di Defender.

    ![Disabilitazione Defender](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/9.3windwosFirewallDesable.png)
    ![Modifiche FW](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/9.1regoleFW3.png)
    ![Modifiche FW](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/9.2regoleFW4.png)

3. **Attacchi simulati da Kali Linux (IP: 192.168.3.11):**
   - **Scansione Nmap:**
     ```bash
     nmap -sS -p- 192.168.2.10
     ```
   - **Tentativo di brute force SSH:**
     ```bash
     hydra -l admin -P password_list.txt ssh://192.168.2.10
     ```
   - **Utilizzo di CrackMapExec:**
     ```bash
     crackmapexec smb 192.168.2.10 -u Administrator -p /usr/share/wordlists/rockyou.txt
     ```
     Questo comando tenta un brute force sulle credenziali SMB del sistema Windows 11 utilizzando il file `rockyou.txt` come dizionario di password. L'obiettivo è testare la sicurezza dei servizi SMB esposti, verificando se esistono credenziali deboli o facilmente indovinabili.

    ![Scansione NMap e Hydra Attack](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/11portaApertaEAttaccoHydra.png)
    ![CrackMapExec Attack](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/12crackmapAttack1.png)
    ![CrackMapExec Attack3](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/13crackmapAttack3.png)


5. **Esito degli Attacchi:**
   - L'attacco CrackMapExec ha restituito errori dovuti alla codifica UTF-8, evidenziando problemi con la gestione di caratteri speciali nel file di dizionario.
   - Sono stati generati numerosi eventi di **STATUS_ACCOUNT_LOCKED_OUT** su Windows 11 per l'utente Administrator, a seguito dei tentativi falliti di login SMB.

     ![CrackMapExec UTF Error](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/14crackmapAttackUTFError.png)
     ![CrackMapExec Account Block](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/15crackmapAttack2.png)

6. **Analisi dei Log in Wazuh Dashboard:**
   - Nel dashboard Wazuh, ho identificato i seguenti eventi:
     - Tentativi falliti di login con SMB.
     - Notifiche di account bloccati.
     - Incremento del traffico sospetto su porte SMB.

    ![Log Wazuh](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/16logWazuh.png)

### Raccolta e analisi dei log
- **Accesso ai log tramite Wazuh Dashboard:**
  - Ho navigato nei moduli "Alerts" e "Threats Detected".
  - Ho identificato eventi critici legati agli attacchi eseguiti.

- **Log significativi:**
  - Connessioni SSH non autorizzate.
  - Aumento del traffico sospetto durante la scansione di rete.
  - Tentativi di accesso SMB con credenziali errate, segnalati come tentativi di brute force.

    ![Screen Wazuh](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/Screen%20Wazuh.png)
    ![Vulnerabilità](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/screenVulnerabilit%C3%A0.png)
    ![Threat Hunting](https://github.com/Gif-97/SIEM-Pratice/blob/main/media/threatHunting.png)

---

## Conclusione
Questo progetto dimostra le mie competenze pratiche nella configurazione e utilizzo di strumenti SIEM per la rilevazione e gestione di minacce. Gli attacchi simulati evidenziano la mia capacità di analizzare log e identificare attività sospette, essenziali per ruoli in ambito red/blue team.
