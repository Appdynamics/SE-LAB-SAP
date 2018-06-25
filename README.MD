SE Lab for AppDynamics - SAP ABAP
======
# 1. General Information
**This Lab will cover:**
* SAP APM

# 2. Lab Setup
## 2.1. Create your environment
* Log into [Ravello] (appdynamicsravello)
* Create an App from the Blueprint ```sap-netweaver-dev-bp``` (this will take some time)
* Download and Install SAP GUI (click through Install)
  * [Windows]
  * [Linux]
  * [Mac]

## 2.2. Accessing the Lab
* SSH Login
  * Download [SSH Key]
  * `chmod 600 devops_level1_lab.pem`
  * `ssh -i devops_level1_lab.pem ubuntu@<Ravello VM Host/IP>`
* Accessing the AppDynamics UIs
  * Controller
    * http://<Ravello VM Host/IP>:8090
    * User: ```admin```
    * Password: ```appdynamics```
  * Enterprise Console
    * http://<Ravello VM Host/IP>:9191
    * User: ```admin```
    * Password: ```appdynamics```
* Open SAP GUI

![open sap gui](img/open_sap_gui.png)

* Create New Connection
  * Go To the `Advanced Tab`
  * Check `Expert Mode`
  * Put in your Connection Details like
    * `conn=/H/<Ravello VM Host/IP>/S/3200&wan=true&clnt=000&user=SAP*`
  * Save

![connection sap gui](img/connection_sap_gui.png)

* Connect
  * User: SAP*
  * Password: Appl1ance

![connected sap gui](img/connected_sap_gui.png)


[ravello]: https://cloud.ravellosystems.com/
[SSH Key]: https://singularity.jira.com/wiki/download/attachments/353600935/devops_level1_lab.pem?version=1&modificationDate=1517075637133&cacheVersion=1&api=v2&download=true
[Windows]: https://owncloud.appd.duckdns.org/s/GoR9z54GGgpQw9C
[Linux]: https://owncloud.appd.duckdns.org/s/BtF79RGWD3ca9pR
[Mac]: https://owncloud.appd.duckdns.org/s/894rw9QxDn9QJqX
