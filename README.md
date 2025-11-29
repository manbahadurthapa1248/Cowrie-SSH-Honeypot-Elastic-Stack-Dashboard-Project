Cowrie SSH Honeypot & Elastic Stack Dashboard Project
=====================================================

A cybersecurity monitoring project that sets up an **SSH honeypot using Cowrie** and integrates it with the **Elastic Stack (Elasticsearch, Kibana, Filebeat)** to visualize attacker activity in real time.

This system captures brute-force attempts, commands executed by attackers, TTP patterns, malware downloads, and other SSH interaction logs, and forwards them into Kibana dashboards.

   <img width="1024" height="1024" alt="Gemini_Generated_Image_h3yvdlh3yvdlh3yv" src="https://github.com/user-attachments/assets/5602a47f-a323-4c0a-8b87-dd0a2ff87673" />

📌 Features
-----------

*   High-interaction SSH honeypot (Cowrie)
    
*   Emulates a fake Linux environment for attackers
    
*   Captures SSH login attempts, commands, file downloads, session activity
    
*   Filebeat collects and forwards logs securely
    
*   Elasticsearch stores logs efficiently
    
*   Kibana dashboards for real-time attack visualization
    

🛠 Technologies Used
--------------------

*   **Cowrie Honeypot** (SSH/Telnet emulation)
    
*   **Elastic Stack**
    
    *   Elasticsearch (Log storage)
        
    *   Kibana (Dashboard visualization)
        
    *   Filebeat (Log forwarding)
        
*   **Kali Linux Server**
    

⚙️ Installation & Setup Guide
=============================

1\. Install Cowrie Honeypot
===========================

### 📥 Install Dependencies

```bash
  sudo apt update
  sudo apt install git python3 python3-venv python3-pip libssl-dev libffi-dev
```

### 📥 Clone Cowrie

```bash
  git clone https://github.com/cowrie/cowrie.git
  cd cowrie
```

### 🐍 Create Virtual Environment

```bash
  python3 -m venv cowrie-env
  source cowrie-env/bin/activate
  pip install --upgrade pip
  pip install -r requirements.txt
```

### 🛠 Configure Cowrie

Edit the main config file:

```bash
  cp etc/cowrie.cfg.dist etc/cowrie.cfg
  nano etc/cowrie.cfg
```

Modify SSH settings or Keep Default:

```
  listen_endpoints = tcp:2222:interface=0.0.0.0
  hostname = fake-linux
```
If you want to use port fowarding from default cowrie ssh port: 2222 to port: 22, use the following command:

```bash
  sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
```

If you have Python 3.13+, sometimes cowrie executable is not found, so you will have to install more build dependencies, use:

```bash
  pip install --use-pep517 -e .
```

### ▶️ Start Cowrie
a) If using Python3.12, 3.11 use following commands:

```bash
  bin/cowrie start   # To start
  bin/cowrie restart  # To restart
  bin cowrie stop  # To stop
```

b) If using Python3.13+ use following commands:

```bash
  cowrie-env/bin/cowrie start   # To start
  cowrie-env/bin/cowrie restart  # To restart
  cowrie-env/bin cowrie stop  # To stop
```
Check if cowrie is running

```bash
  ps aux | grep cowrie
```

Log files will be stored in:

```bash
  cowrie/var/log/cowrie/
```

You can also tail the logs of the cowrie using following command:

```bash
  tail -f ~/cowrie/var/log/cowrie/cowrie.log
```

2\. Install Elasticsearch & Kibana
==================================

### 📦 Download & Install Elasticsearch

```bash
  sudo apt install elasticsearch
  sudo systemctl enable --now elasticsearch
```

### 📦 Install Kibana

```bash 
  sudo apt install kibana
  sudo systemctl enable --now kibana
```   

Access Kibana UI in browser:

`   http://localhost:5601   `

3\. Install & Configure Filebeat
================================

### 📦 Install Filebeat

```bash
  sudo apt install filebeat
``` 

### 🔧 Configure Filebeat Inputs

Edit:

``` bash
  sudo nano /etc/filebeat/filebeat.yml
``` 

Add custom log input:

```
filebeat.inputs:    
 - type: log
   enabled: true
   paths:
    - /home/kali/cowrie/var/log/cowrie/cowrie.log
    - /home/kali/cowrie/var/log/cowrie/cowrie.json
```

### 🔧 Enable Elasticsearch & Kibana Output

```
output.elasticsearch:
  hosts: ["http://localhost:9200"]

setup.kibana:
  host: "localhost:5601"   
```

### ▶️ Start Filebeat

``` bash
  sudo systemctl enable filebeat
  sudo systemctl start filebeat
``` 

4\. View Logs in Kibana Dashboard
=================================

### 📊 Import Filebeat Dashboards

```bash
  sudo filebeat setup --dashboards
``` 

Then open Kibana →**Dashboard → Filebeat SSH / Syslog dashboards** or create a custom dashboard for Cowrie logs.

You will now see:

*   Login attempts
    
*   Password guesses
    
*   Commands executed
    
*   TTY session playback
    
*   Malware download attempts
    
*   Attacker IP statistics
    

📁 Project Structure
====================

```
  /cowrie            # Honeypot files and logs  
  /var/log/cowrie                 # JSON and text logs 
  /etc/filebeat/filebeat.yml      # Filebeat configuration  
  /etc/elasticsearch/elasticsearch.yml    # Elastic configuration 
  /etc/kibana/kibana.yml   # Kibana configuration
```

🚀 Usage Workflow
=================

1.  Start Cowrie honeypot
    
2.  Attackers connect to the dummy SSH server
    
3.  Cowrie logs activity in JSON & log files
    
4.  Filebeat ships logs to Elasticsearch
    
5.  Kibana visualizes attacker behavior
    

This provides a complete intrusion monitoring and analysis environment.

📊 Output
=================

This section showcases the visual and JSON outputs generated from the Cowrie honeypot and analyzed through the **Elastic Stack**. Screenshots illustrate the log ingestion, filtering, and dashboard visualization, while JSON snippets represent the raw log data behind key events.

1. Kibana UI Displaying Cowrie Logs (1074 Events)
   
   <img width="1920" height="919" alt="1" src="https://github.com/user-attachments/assets/08721ec3-0700-4e28-a313-923fd56ff753" />

2. Source IPs Observed in Attacks (3 Unique IPs Including Localhost)

   <img width="1920" height="921" alt="2" src="https://github.com/user-attachments/assets/6ef045fb-ca74-48ab-8292-dbbcf4ebe74a" />

3. KQL Filter Applied: `src.ip = 127.0.0.1`

   <img width="1920" height="923" alt="3" src="https://github.com/user-attachments/assets/ac393195-3977-42fa-b2b7-d26e6be680f5" />

4. Table View – Failed Login Attempt

   <img width="540" height="826" alt="4" src="https://github.com/user-attachments/assets/9c921a15-9ff7-478d-b080-d3f32ba0e3d8" />

5. JSON Log for Failed Login Attempt

```json
{
  "_index": "cowrie",
  "_id": "O2qQvpoBHh7gAIEQf1bZ",
  "_version": 1,
  "_source": {
    "eventid": "cowrie.login.failed",
    "session": "216e7b597988",
    "message": "login attempt [root/131423] failed",
    "uuid": "4d7bc4b4-ca6a-11f0-967e-14755b4acf81",
    "src_ip": "127.0.0.1",
    "password": "131423",
    "protocol": "ssh",
    "system": "HoneyPotSSHTransport,0,127.0.0.1",
    "cowrie_ingest": "true",
    "sensor": "kali",
    "time": 1764133732.3046572,
    "username": "root",
    "timestamp": "2025-11-26T10:53:52.304657Z"
  },
  "fields": {
    "eventid": [
      "cowrie.login.failed"
    ],
    "session.keyword": [
      "216e7b597988"
    ],
    "session": [
      "216e7b597988"
    ],
    "eventid.keyword": [
      "cowrie.login.failed"
    ],
    "uuid.keyword": [
      "4d7bc4b4-ca6a-11f0-967e-14755b4acf81"
    ],
    "message": [
      "login attempt [root/131423] failed"
    ],
    "system.keyword": [
      "HoneyPotSSHTransport,0,127.0.0.1"
    ],
    "uuid": [
      "4d7bc4b4-ca6a-11f0-967e-14755b4acf81"
    ],
    "src_ip": [
      "127.0.0.1"
    ],
    "username.keyword": [
      "root"
    ],
    "password": [
      "131423"
    ],
    "protocol": [
      "ssh"
    ],
    "system": [
      "HoneyPotSSHTransport,0,127.0.0.1"
    ],
    "sensor.keyword": [
      "kali"
    ],
    "message.keyword": [
      "login attempt [root/131423] failed"
    ],
    "cowrie_ingest": [
      "true"
    ],
    "protocol.keyword": [
      "ssh"
    ],
    "sensor": [
      "kali"
    ],
    "src_ip.keyword": [
      "127.0.0.1"
    ],
    "time": [
      1764133800
    ],
    "password.keyword": [
      "131423"
    ],
    "cowrie_ingest.keyword": [
      "true"
    ],
    "timestamp": [
      "2025-11-26T10:53:52.304Z"
    ],
    "username": [
      "root"
    ]
  }
}
```

6. Table View – Successful Login Attempt

   <img width="539" height="825" alt="5" src="https://github.com/user-attachments/assets/a2413543-2722-4f29-a630-3fcf78bbaa84" />

7. JSON Log for Successful Login Attempt

```json
{
  "_index": "cowrie",
  "_id": "A2qavpoBHh7gAIEQ8lef",
  "_version": 1,
  "_source": {
    "eventid": "cowrie.login.success",
    "session": "1a74c34fe752",
    "message": "login attempt [root/michael] succeeded",
    "uuid": "4d7bc4b4-ca6a-11f0-967e-14755b4acf81",
    "src_ip": "127.0.0.1",
    "password": "michael",
    "protocol": "ssh",
    "system": "HoneyPotSSHTransport,18,127.0.0.1",
    "cowrie_ingest": "true",
    "sensor": "kali",
    "time": 1764134416.9746299,
    "username": "root",
    "timestamp": "2025-11-26T11:05:16.974630Z"
  },
  "fields": {
    "eventid": [
      "cowrie.login.success"
    ],
    "session.keyword": [
      "1a74c34fe752"
    ],
    "session": [
      "1a74c34fe752"
    ],
    "eventid.keyword": [
      "cowrie.login.success"
    ],
    "uuid.keyword": [
      "4d7bc4b4-ca6a-11f0-967e-14755b4acf81"
    ],
    "message": [
      "login attempt [root/michael] succeeded"
    ],
    "system.keyword": [
      "HoneyPotSSHTransport,18,127.0.0.1"
    ],
    "uuid": [
      "4d7bc4b4-ca6a-11f0-967e-14755b4acf81"
    ],
    "src_ip": [
      "127.0.0.1"
    ],
    "username.keyword": [
      "root"
    ],
    "password": [
      "michael"
    ],
    "protocol": [
      "ssh"
    ],
    "system": [
      "HoneyPotSSHTransport,18,127.0.0.1"
    ],
    "sensor.keyword": [
      "kali"
    ],
    "message.keyword": [
      "login attempt [root/michael] succeeded"
    ],
    "cowrie_ingest": [
      "true"
    ],
    "protocol.keyword": [
      "ssh"
    ],
    "sensor": [
      "kali"
    ],
    "src_ip.keyword": [
      "127.0.0.1"
    ],
    "time": [
      1764134400
    ],
    "password.keyword": [
      "michael"
    ],
    "cowrie_ingest.keyword": [
      "true"
    ],
    "timestamp": [
      "2025-11-26T11:05:16.974Z"
    ],
    "username": [
      "root"
    ]
  }
}
```

8. Dashboard View With KQL: `message.keyword : "*cmd*"`

  <img width="1920" height="921" alt="6" src="https://github.com/user-attachments/assets/6b03c339-92c4-495a-9125-cfae622c5232" />

9. JSON Log Example (Command Execution – e.g., `id`)

```json
{
  "_index": "cowrie",
  "_id": "rGqzvpoBHh7gAIEQlFcd",
  "_version": 1,
  "_source": {
    "src_ip": "127.0.0.1",
    "eventid": "cowrie.command.input",
    "input": "id",
    "protocol": "ssh",
    "system": "HoneyPotSSHTransport,18,127.0.0.1",
    "session": "ae0144846ed1",
    "cowrie_ingest": "true",
    "sensor": "kali",
    "time": 1764136031.2561884,
    "message": "CMD: id",
    "uuid": "4d7bc4b4-ca6a-11f0-967e-14755b4acf81",
    "timestamp": "2025-11-26T11:32:11.256188Z"
  },
  "fields": {
    "eventid": [
      "cowrie.command.input"
    ],
    "session.keyword": [
      "ae0144846ed1"
    ],
    "session": [
      "ae0144846ed1"
    ],
    "eventid.keyword": [
      "cowrie.command.input"
    ],
    "uuid.keyword": [
      "4d7bc4b4-ca6a-11f0-967e-14755b4acf81"
    ],
    "message": [
      "CMD: id"
    ],
    "system.keyword": [
      "HoneyPotSSHTransport,18,127.0.0.1"
    ],
    "uuid": [
      "4d7bc4b4-ca6a-11f0-967e-14755b4acf81"
    ],
    "src_ip": [
      "127.0.0.1"
    ],
    "input": [
      "id"
    ],
    "protocol": [
      "ssh"
    ],
    "system": [
      "HoneyPotSSHTransport,18,127.0.0.1"
    ],
    "sensor.keyword": [
      "kali"
    ],
    "message.keyword": [
      "CMD: id"
    ],
    "cowrie_ingest": [
      "true"
    ],
    "input.keyword": [
      "id"
    ],
    "protocol.keyword": [
      "ssh"
    ],
    "sensor": [
      "kali"
    ],
    "src_ip.keyword": [
      "127.0.0.1"
    ],
    "time": [
      1764136100
    ],
    "cowrie_ingest.keyword": [
      "true"
    ],
    "timestamp": [
      "2025-11-26T11:32:11.256Z"
    ]
  }
}
```

📚 Citations & Official References
==================================

1.  Cowrie Honeypot Documentation: [https://github.com/cowrie/cowrie](https://github.com/cowrie/cowrie)
    
2.  Cowrie Official Setup Guide: [https://cowrie.readthedocs.io/en/latest/](https://cowrie.readthedocs.io/en/latest/)
    
3.  Elastic Filebeat Documentation: [https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-overview.html](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-overview.html)
    
4.  Elasticsearch Installation Reference: [https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html)
    
5.  Kibana Installation Reference: [https://www.elastic.co/guide/en/kibana/current/install.html](https://www.elastic.co/guide/en/kibana/current/install.html)
