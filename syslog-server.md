# Install the Linux OMS Agent Ubuntu 20.04 or 19.10

## Install the Python Libraries

`sudo apt install python auditd -y`

## Install the OMS Linux Agent

`sudo wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <workspace-id> -s <workspace-key>`

## Populate the security_events.conf

`sudo wget -O /etc/opt/microsoft/omsagent/<workspace-id>/conf/omsagent.d/security_events.conf "https://aka.ms/syslog-config-file-linux"`

## Elevate to Root

`sudo su`

## Edit these file and match the config 

`nano /etc/rsyslog.conf`

    # provides TCP syslog reception
    module(load="imtcp")
    input(type="imtcp" port="514")



`nano /etc/rsyslog.d/security-config-omsagent.conf`


    :rawmsg, regex, "CEF\|ASA" ~
    local4.debug @@127.0.0.1:25226

## Edit this file and match the config 

`nano /etc/rsyslog.d/50-default.conf`


    local.*;auth,authpriv.none      -/var/log/syslog


## Restart the services

`sudo /opt/microsoft/omsagent/bin/service_control restart <workspace-id>`

`sudo service rsyslog restart`

## Run the trouble shooting script

`sudo wget https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/CEF/cef_troubleshoot.py&&sudo python cef_troubleshoot.py <workspace-id>`

## These errors can be ignored

    Error - security-config-omsagent.conf does not contain rsyslog.d daemon routing to oms-agent
    Error: rsyslog daemon configuration was found invalid.

    Error: daemon incoming port is not open, please check that the process is up and running and the port is configured correctly.
    Action: enable ports in '/etc/rsyslog.conf' file which contains daemon incoming ports.

    Error: agent is not listening to incoming port 25226 please check that the process is up and running and the port is configured correctly.[Use netstat -an | grep [daemon port] to validate the connection or re-run ths script]




# Troubleshooting:

## Monitor incoming logs
Confirm that syslogs are being collected by listening to port 514

    sudo su
    tcpdump -A -ni any port 514 -vv


## Show OMS Log File 
Open a new SSH session and run the following to se a live view of the OMS logs

    sudo su 
    tail -f /var/opt/microsoft/omsagent/<workspace-id>/log/omsagent.log