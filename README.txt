Tasks
1- install splunk
2- install forwarder
3- confi gure Domain Controller
4- install
Xampp
Web app server
5- install
Pfsense
fi
rewall
6- install Suricata IPS/IDS
6- confi gure
DVWA
vulnerable Web app on xampp
7- confi gure securityMod
WAF
To detect web app attacks
8- write rule in splunk to generate alert
9- detect Active Directory Attacks
splunk install
1- Download
Splunk
2- sudo dpkg -i splunk.deb
3- sudo /opt/splunk/bin/splunk start --accept-license
4- sudo /opt/splunk/bin/splunk enable boot-start
you can access splunk via web browser http://127.0.0.1:8000
install forwarder
Download
Forwarder
Add Splunk Machine IP Address And Default Port Like ==> 192.168.1.20:
8089
Add Splunk Machine IP Address And Default Port Like ==> 192.168.1.20:
9997
Confi gure Receiving In Splunk
Note:
Make sure allow traffi c through Firewall port 8089, 9997 on Windows Machine And Linux Machine
Write Default Port
9997
Add Date In Splunk
1- Select host from Available Host
2- Add A New Host Name
Create New Index Name
PC-01
After Create It Choose It And Click
Rivew
And
Done
Install And Confi gure Sysmon:
Download
Sysmon
Download
Confi guration
File
Copy Confi guration fi le to Sysmon Directory
Open PowerShell OR CMD As Administrator
Command:
Sysmon64.exe -accepteula -i sysmonconfi g.xml
Send Sysmon Logs To Splunk
Go To This Path ==>
C:\ProgramFiles\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local
Open Inputs.conf fi le And Add this
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
index = PC-01
Install Xampp Server On windows Server To Confi gure DVWA Lab
Download
Xampp
Installation Process Like As Any exe File Click Next Next...BlaBlaBla
Note: Make Sure Choose this Options
When Installation Complete Successfully Open App You will Show This, Click Start On
Apache
And
MySQL
Confi gure DVWA Vulnerable Web App
Download
DVWA
After Download It Extract File And Rename It
dvwa
And Copy To This Path ==>
C:\xampp\htdocs\
now you need edit
dvwa
confi guration fi le from this path ==>
C:\xampp\htdocs\dvwa\confi g\
you can see fi le name
confi g.inc.php.dist
rename it
confi g.inc.php
and open fi le
Add username and password you will use it to login
now we will confi gure Database open
http://127.0.0.1/phpmyadmin/
in your browser to create dvwa Database
Now Open dvwa in your browser
http://127.0.0.1/dvwa/
And scroll Down And click
Create / Reset Database
Now Refresh browser and login via this credential
admin:password
Congratulations
now create a new index in splunk name
webapp
to send logs add this confi guration in inputs.conf >> C:\Program Files\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local
[monitor://C:\xampp\apache\logs\access.log]
sourcetype = apache:access
index = webapp
disabled = false
[monitor://C:\xampp\apache\logs\error.log]
sourcetype = apache:error
index = webapp
disabled = false
install WAF
download mod security WAF from
here
unzip fi le and copy
mob_security2.so
to >> C:\xampp\apache\modules
create confi g fi le name
modsecurity.conf
in >> C:\xampp\apache\conf and add this settings
SecAuditLog "C:/xampp/apache/logs/modsec_audit.log"
SecDebugLog "C:/xampp/apache/logs/modsec_debug.log"
SecRuleEngine DetectionOnly
SecRequestBodyAccess On
SecAuditEngine On
SecAuditLogParts ABC
Note:
We will use the WAF here to capture the request body, and we will not rely on the default rules because we want to try writing custom rules by hand and generate alert
now create a new index in splunk name
waf
to send logs to splunk add this confi guration in
inputs.conf
>> C:\Program Files\SplunkUniversalForwarder\etc\apps\SplunkUniversalForwarder\local
[monitor://C:/xampp/apache/logs/modsec_audit.log]
disabled = false
index = waf
sourcetype = modsecurity:json
Install
pfsense
FW
download
pfsense
install it in VMware
create a 2 network card
(LAN, Bridge)
Ip some thing like 10.10.10.1/24 WAN 192.168.1.0/24 after installation complete access Pfsense from any Machine in NAT card via web browser by pfsense ip 10.10.10.1
require check update from Tab >> System >> update
Notes:
add all VM machine in LAN Network add statice Ip Range 192.168.200.0/24 Subnet 255.255.255.0 Default Gateway fi rewall Ip like 192.168.200.1
now send logs to splunk
scroll down to
and click save
now confi gure receiving in splunk >> settings >> data inputs >> UDP
and click next
source Type pfsense
scroll down to
click Review >> submit
check logs send succefuly
Install Suricata IPS/IDS System
from pfsense web GUI >> System >> Package Manger
after installation complete go to suricat from >>Services >> Suricata >> Global Settings
to do some confi guration
Link in First image >>
wget http://rules.emergingthreats.net/open/suricata-7.0.3/emerging-all.rules.tar.gz
Click save and go to >> services >> Suricata >> update
to apply rule and confi guration
Now go to >> System >> Advanced >> Networking
to enable suricate got on Raw Traffi c without any edit from fi rewall
scroll down to
Click save and go to >> Services >> Suricata >> Interfaces
to allow this confi guration in specifi c interface
scroll down to
Click save and go to check logs gerated successfl uy
go to this url to make suricata genrate alert >>
http://testmyids.com
now lets go to send logs to splunk
install Syslog-ng From package manger to send logs
go to >> services >> syslog-ng
now we will create three object to send logs to splunk
1- monitor fi le
object name: s_suricata_alerts
object type: Source
object Parameters:
{
wildcard-fi le(
base-dir("/var/log/suricata/suricata_em153486")
fi lename-pattern("*")
recursive(yes)
follow-freq(1)
fl ags(no-parse)
program-override("suricata")
);
};
and click save
2- create socket
object name: d_splunk_suricata
object type: destination
object Parameters:
{
syslog("
splunk-ip
" port(5141) transport("udp"));
};
and click save
3- forward logs
object name: log_suricata_alerts
object type: Logs
object Parameters:
{
source(s_suricata_alerts);
destination(d_splunk_suricata);
};
and click save
confi gure splunk forward and make sure port is 5141 source type json and create new index suricata
go to this url to make suricata genrate alert >>
http://testmyids.com
check logs send successfuly to splunk
