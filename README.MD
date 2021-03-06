SE Lab for AppDynamics - SAP ABAP
======
# 1. General Information
**This Lab will cover:**
* SAP APM

# 1.1. Pre-requisites
* Ravello Account
* For Linux/Mac Java 1.8
* SSH Client
* **Optional:** RDP Client

# 2. Lab Setup
## 2.1. Create your environment
* Log into [Ravello] (appdynamicsravello)
* Create an App from the Blueprint `sap-netweaver-dev-bp-v1.1` (20 minutes till everything is up)
* **Optional:** Download and Install SAP GUI (click through Install)
  * [Windows]
  * [Linux] (Java 1.8 required)
  * [Mac] (Java 1.8 required)

## 2.2. Accessing the Lab

### 2.2.1. Using the Windows Client

* Lab consists of 3 VMs
  * Windows Client VM - `windows-client`
  * SAP Netweaver Developer Edition - `sap-netweaver`
  * AppDynamics Platform - `appd-platform`
* SSH Login
  * Download [SSH Key]
  * `chmod 600 devops_level1_lab.pem`
  * `ssh -i devops_level1_lab.pem ubuntu@<Ravello VM Host/IP>`
* RDP Login
  * You find the Credentials under `General -> Description` within Ravello
* Check the Startup status
  * `docker logs -f sap`
  * SAP is up when you see

```
starting SAP Instance D00
Startup-Log is written to /home/npladm/startsap_D00.log
-------------------------------------------
/usr/sap/NPL/D00/exe/sapcontrol -prot NI_HTTP -nr 00 -function Start
Instance on host sapnwdev started
```

* Accessing the AppDynamics UIs
  * Controller
    * `http://<appd-platform VM Host/IP>:8090`
    * User: `admin`
    * Password: `appdynamics`
  * Enterprise Console
    * `http://<appd-platform VM Host/IP>:9191`
    * User: `admin`
    * Password: `appdynamics`
* RDP into `windows-client` or use your local SAP GUI Installation
* Open SAP GUI
* Create New Connection
  * Click `Variable Logon`
  * Click `Next`
  * Put in your Connection Details like
    * Application Server `sap-netweaver`
    * Instance Number `00`
    * System ID `NPL`
  * Click `Log On`

![connection sap gui](img/connection_sap_gui.png)

* Logon Details
  * User: `SAP*`
  * Password: `Appl1ance`

![connected sap gui](img/connected_sap_gui.png)

### 2.2.2. Using the Java SAP GUI

* Create New Connection
  * Go To the `Advanced Tab`
  * Check `Expert Mode`
  * Put in your Connection Details like
    * `conn=/H/<Ravello VM Host/IP>/S/3200&wan=true&clnt=000&user=SAP*`
  * Save
* Click `Log On`
* Logon Details
  * User: `SAP*`
  * Password: `Appl1ance`
  

# 3. Install SAP ABAP Agent
* General Setup is documented within [Install SAP Netweaver Systems]

## 3.1. Install the SAP ABAP Components
* Go to the Transports
  * `cd /home/ubuntu/abap_transports`
* Copy the Transports into the SAP transport folders
  * `docker cp ./ sap:/usr/sap/trans/`
* Open the SAP GUI and go to the `/n STMS_IMPORT` Transaction
![stms import](img/stms_import_transaction.png)
* The list will still be empty. Now queue the requests by
  * Click `Extras -> Other Requests -> Add`
  * This has to be done for all 3 Transports
* Import the Transport requests by `Right click -> Import`
* **The Order of the Imports can be found in `/home/ubuntu/abap_transports/readme.txt`**
* Check `Synchronous` within `Execution`
![stms import exe](img/stms_import_execution.png)
* Check `Ignore invalid Component Version` within `Options`
![stms import opts](img/stms_import_options.png)
* Save and confirm for each of the Requests. This will take some time.
* **Go ahead and do 3.2. and 3.3. while the Transports are imported**
* **Ignore if you see Errors on the last Transport**

## 3.2. Run the Remote AppDynamics C++ SDK and HTTP SDK Proxy
* The Agent Bits are available within the Docker Image `michaelenglert/appd-sapagent`
  * Check [AppD SAP Agent] for details
* Run the Container
  * `docker run -d --restart=always --name appd-sapagent -p 7999:7999 -p 8900:8900 michaelenglert/appd-sapagent`

## 3.3. Install the Machine Agent
* We will install the Machine Agent within the SAP Container
* Unzip the Machine Agent
  * `unzip /home/ubuntu/machineagent.zip -d /home/ubuntu/machineagent`
* Configure the Machine Agent
  * `vi /home/ubuntu/machineagent/conf/controller-info.xml`
  * controller-host is `appd-platform`
  * controller-port is `8090`
  * account-access-key is `appdynamics`
  * account-name is `customer1`
  * sim-enabled is `true`
* Copy it into the Container
  * `docker cp /home/ubuntu/machineagent/ sap:/opt/machineagent`
* Start the Machine Agent
  * `docker exec -d sap /bin/bash -c "/opt/machineagent/bin/machine-agent -d -p machine.pid -Dmetric.http.listener=true -Dmetric.http.listener.port=8293 -Dmetric.http.listener.host=0.0.0.0"`

# 4. Configure SAP ABAP Agent
* Open the SAP GUI and go to the `/n /DVD/APPD_CUST` Transaction
* Fill out all the required Information
  * The SDK is remote
  * The Machine Agent is local
  * To connect to the Events Service we need an API Key
![stms import opts](img/appd_config.png)
* On the Status Screen you should see everything as green in the end

# Troubleshooting
* The AppDynamics platform does not come up
  * Delete the old one `docker rm -f appd`
  * Create a new one `docker run -d --restart always --name appd --hostname 3ead5cb9db00 -p 8090:8090 -p 9191:9191 -p 9080:9080 michaelenglert/appd-platform`
  * Copy in the license `docker cp /home/ubuntu/license.lic appd:/opt/appdynamics/platform/controller/`
* SAP License Error `Logon not possible (error in license check)`
  * Go to the transaction `/n SLICENSE`
  * Delete the existin license
  * Go to the [license renewal page] 
    * Select **NPL - SAP Netweaver 7.x (Sybase ASE)**
    * Fill in the requested information
    * Download the new license file `NPL.txt`
  * Replace the `SYSTEM-NR` key with the System Number from SAP GUI in the `NPL.txt` file
  * Import the license
* DNS Resolution does not work
  * run `sudo systemctl restart systemd-resolved` on the Ubuntu Servers


[AppD SAP Agent]: /docker/Dockerfile
[Install SAP Netweaver Systems]: https://docs.appdynamics.com/display/SAP/Install+SAP+Netweaver+Systems
[ravello]: https://cloud.ravellosystems.com/
[SSH Key]: https://singularity.jira.com/wiki/download/attachments/535298310/devops_level1_lab.pem?version=1&modificationDate=1530598758711&cacheVersion=1&api=v2&download=true
[Windows]: https://singularity.jira.com/wiki/download/attachments/535298310/sap_gui_740_windows.exe?version=1&modificationDate=1530191613449&cacheVersion=1&api=v2&download=true
[Linux]: https://singularity.jira.com/wiki/spaces/~michael.englert/pages/535298310/SAP+Lab?preview=/535298310/535527688/sap_gui_740_linux.jar#
[Mac]: https://singularity.jira.com/wiki/download/attachments/535298310/sap_gui_740_mac.jar?version=1&modificationDate=1530191345604&cacheVersion=1&api=v2&download=true
[license renewal page]: https://go.support.sap.com/minisap/#/minisap
