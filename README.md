[![DEVNET](https://upload.wikimedia.org/wikipedia/en/f/f8/CiscoDevNet2.png)](https://developer.cisco.com)

# Ansible - Automated ThousandEyes deployment on Catalyst 9300

## Features
- ThousandEyes package distribution (.tar)
- ThousandEyes Application Hosting container configuration
- Enterprise Proxy SSL Certificate distribution and activation in ThousandEyes container

### Application Hosting on Catalyst 9000
[![APPH](https://pubhub.devnetcloud.com/media/application-hosting/docs/iox-app/images/iox-apphosting-arch.png#developer.cisco.com)](https://developer.cisco.com/docs/app-hosting/#!application-hosting-in-the-enterprise/what-is-application-hosting)
### Assumptions
- I am assuming that the TE agent will be deployed on a private subnet behind a NAT on each of the Catalyst 9000
- This deployment allows for simplified IP addressing, but prevents capability to employ agent-to-agent tests (ie each agent is behind a PAT)

### Deployment topology:
NETWORK--[PAT]--[Gig1/0/24]--Cat9k--[AppGigabitEthernet1/0/1]-[Vlan101]--TE

## Requirements
- Catalyst 9300, minimum IOS-XE 17.3.3
- Ansible 2.11.2
- Docker on local machine (for testing proxy function)
- Ansible automation script offers a DIY alternative to turn-key automation offered by Cisco DNA Center. Cisco DNA Center has built-in Application Deployment workflows. 

## References
- ThousandEyes, Enterprise Proxy Certificate installation: 
-- https://docs.thousandeyes.com/product-documentation/global-vantage-points/enterprise-agents/troubleshooting/installing-ca-certificates-on-enterprise-agents
- Cisco, ThousandEyes deployment guide
-- https://www.cisco.com/c/en/us/products/collateral/switches/catalyst-9400-series-switches/guide-c07-2431113.html
- Docker Squid Container (for verification of proxy operation)
-- https://github.com/salrashid123/squid_proxy

## Script folder structure
- pem: 
-- directory to keep enterprise proxy public certificates in .pem format that will be instantiated in each of the TE containers
- te-agent: 
-- directory to keep TE docker distribution in .tar format
- params.yml 
-- Edit to fit your environment (ie names of cert files, TE container setup parameters)
- hosts
-- Credentials to access catalyst via ssh

## Sample execution
Deployment script operates in two phases:
- TE agent deployment and activation
-- Verify if IOSXE meets minimum requirement
-- Verify TE App is already deployed
-- If TE App is not deployed, provision basic configuration (TE Vlan, AppGigEthernet trunk, NAT, NAT ACL)
-- Check for presence of TE installer tar, and distribution if necessary
-- TE App install, activate and start
- Enterprise Proxy SSL Certificate distribution
-- Verify if TE App is in 'RUNNING' state
-- Transfer list of Proxy SSL pub certificates to the switch
-- Transfer the same certs into the running TE container
-- Move the certs inside the container to the cert store folder (ie /usr/share/ca-certificates)
-- Update the cert reference config /etc/ca-certificates.conf
-- Regenerate certificate store

```sh
 ansible-playbook -i hosts ansible-te-complete.yml 

PLAY [Ansible TE Distribution] *********************************************************************************************************************************************************************************************************************************

TASK [include_vars] ********************************************************************************************************************************************************************************************************************************************
ok: [c9300p27]
ok: [c9300p28]

PLAY [Automated Distribution and Activation of ThousandEyes Enterprise Agent on Catalyst 9300] *****************************************************************************************************************************************************************

TASK [include_vars] ********************************************************************************************************************************************************************************************************************************************
ok: [c9300p27]
ok: [c9300p28]

TASK [Check IOSXE version] *************************************************************************************************************************************************************************************************************************************
ok: [c9300p28]
ok: [c9300p27]

TASK [Check running Apps] **************************************************************************************************************************************************************************************************************************************
ok: [c9300p27]
ok: [c9300p28]

TASK [Setting facts | TE App Present] **************************************************************************************************************************************************************************************************************************
skipping: [c9300p27] => (item=TE                                       DEPLOYED) 
ok: [c9300p28] => (item=cat9kte                                  RUNNING)

TASK [Check TE tar presence | Remote] **************************************************************************************************************************************************************************************************************************
skipping: [c9300p28]
ok: [c9300p27]

TASK [Check TE tar presence | Local] ***************************************************************************************************************************************************************************************************************************
skipping: [c9300p28]
ok: [c9300p27]

TASK [Setting facts | Remote TE archive size] ******************************************************************************************************************************************************************************************************************
skipping: [c9300p28]
ok: [c9300p27]

TASK [debug] ***************************************************************************************************************************************************************************************************************************************************
skipping: [c9300p28]
ok: [c9300p27] => {
    "msg": [
        "Size check: 178872320|178872320|True"
    ]
}

TASK [Configure TE Vlan] ***************************************************************************************************************************************************************************************************************************************
skipping: [c9300p28]
ok: [c9300p27]

TASK [Configure TE VLAN SVI] ***********************************************************************************************************************************************************************************************************************************
skipping: [c9300p28]
ok: [c9300p27]

TASK [Configure TE NAT ACL] ************************************************************************************************************************************************************************************************************************************
skipping: [c9300p28]
ok: [c9300p27]

TASK [Configure NAT on Outside inteface] ***********************************************************************************************************************************************************************************************************************
skipping: [c9300p28]
ok: [c9300p27]

TASK [Configure NAT on Inside inteface] ************************************************************************************************************************************************************************************************************************
skipping: [c9300p28]
ok: [c9300p27]

TASK [Configure NAT] *******************************************************************************************************************************************************************************************************************************************
skipping: [c9300p28]
ok: [c9300p27]

TASK [Enable AppH TE Docker Container definition] **************************************************************************************************************************************************************************************************************
skipping: [c9300p28]
ok: [c9300p27]

TASK [Enable AppH TE Docker Container routing] *****************************************************************************************************************************************************************************************************************
skipping: [c9300p28]
ok: [c9300p27]

TASK [Enable AppH TE Docker Container TE params] ***************************************************************************************************************************************************************************************************************
skipping: [c9300p28]
ok: [c9300p27]

TASK [Adding Proxy Authentication if required] *****************************************************************************************************************************************************************************************************************
skipping: [c9300p27]
skipping: [c9300p28]

TASK [Copy TE tar if remote size does not match] ***************************************************************************************************************************************************************************************************************
skipping: [c9300p27]
skipping: [c9300p28]

TASK [Activate AppH] *******************************************************************************************************************************************************************************************************************************************
skipping: [c9300p28]
ok: [c9300p27]

PLAY [Automated Distribution of ThousandEyes Proxy SSL Certificates] *******************************************************************************************************************************************************************************************

TASK [include_vars] ********************************************************************************************************************************************************************************************************************************************
ok: [c9300p28]
ok: [c9300p27]

TASK [Checking if App is running] ******************************************************************************************************************************************************************************************************************************
ok: [c9300p28]
ok: [c9300p27]

TASK [Checking for presense of certificate PEM on target] ******************************************************************************************************************************************************************************************************
ok: [c9300p28] => (item=gdig2.crt.pem)
ok: [c9300p27] => (item=gdig2.crt.pem)
ok: [c9300p28] => (item=gdroot-g2.pem)
ok: [c9300p27] => (item=gdroot-g2.pem)
ok: [c9300p28] => (item=CA_crt.pem)
ok: [c9300p27] => (item=CA_crt.pem)

TASK [Copying certificate pem file to target] ******************************************************************************************************************************************************************************************************************

TASK [Injecting certificate pem into running TE container] *****************************************************************************************************************************************************************************************************

TASK [Moving certificate in running TE container to cert store folder] *****************************************************************************************************************************************************************************************

PLAY RECAP *****************************************************************************************************************************************************************************************************************************************************
c9300p27                   : ok=24   changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
c9300p28                   : ok=11   changed=0    unreachable=0    failed=0    skipped=16   rescued=0    ignored=0   

```

## Resulting Application Hosting configuration on one of the cat9ks
```sh
app-hosting appid cat9kte
 app-vnic AppGigabitEthernet trunk
  vlan 101 guest-interface 0
   guest-ipaddress 192.168.2.3 netmask 255.255.255.0
 app-default-gateway 192.168.2.1 guest-interface 0
 app-resource docker
  prepend-pkg-opts
  run-opts 1 "-e TEAGENT_ACCOUNT_TOKEN=CHANGE_ME"
  run-opts 2 "--hostname c9300p27-te"
  run-opts 3 "-e TEAGENT_PROXY_TYPE=STATIC"
  run-opts 4 "-e TEAGENT_PROXY_LOCATION='10.1.1.3:3128'"
  run-opts 5 "-e TEAGENT_PROXY_USER='username'"
  run-opts 6 "-e TEAGENT_PROXY_PASS='password'"
 name-server0 128.107.212.175
 name-server1 64.102.6.247
 ```
 ```sh
C9300#sh run int gi 1/0/24
!
interface GigabitEthernet1/0/24
 no switchport
 ip address 10.1.1.5 255.255.255.0
 ip nat outside
end

C9300#sh run int vlan 101
!
interface Vlan101
 ip address 192.168.2.1 255.255.255.0
 ip nat inside
end

C9300#sh vlan id 101

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
101  AppH-TE-VLAN                     active    Ap1/0/1

C9300#sh run | s NAT_ACL
ip nat inside source list NAT_ACL interface GigabitEthernet1/0/24 overload
ip access-list standard NAT_ACL
 10 permit 192.168.0.0 0.0.255.255
 ```
 ## Resulting configuration inside docker TE container
Notice presence of CA_crt.pem, gdig2.crt.pem, and gdroot-g2.pem files in the certificate store
 ```sh
root@c9300p27-te:/# ls /usr/share/ca-certificates/
CA_crt.pem  gdig2.crt.pem  gdroot-g2.pem  mozilla  thousandeyes
 ```
 Notice the three certs added to the tail of the config file
```sh
root@c9300p27-te:/# tail /etc/ca-certificates.conf
mozilla/Microsoft_RSA_Root_Certificate_Authority_2017.crt
mozilla/Trustwave_Global_Certification_Authority.crt
mozilla/Trustwave_Global_ECC_P256_Certification_Authority.crt
mozilla/Trustwave_Global_ECC_P384_Certification_Authority.crt
mozilla/UCA_Extended_Validation_Root.crt
mozilla/UCA_Global_G2_Root.crt
thousandeyes/Microsoft_Root_Certificate_Authority_2011.crt
gdig2.crt.pem
gdroot-g2.pem
CA_crt.pem
```
 
 
 ## Proxy Verification
 To verify operation of the deployed TE agents, we can employ a pre-built Squid docker container with SSL decrypt:
 ```sh
 docker run  -p 3128:3128 -ti docker.io/salrashid123/squidproxy /apps/squid/sbin/squid -NsY -f /apps/squid.conf.intercept
 ```
 
## TE Proxy interop Verification - just before insertion of PEM certs. Note "peer certificate cannot be authenticated"
```sh
C9300#app-hosting connect appid cat9kte session /bin/bash
root@c9300p27-te:/# tail -f /var/log/agent/te-agent.log
2021-07-26 21:40:55.889 INFO  [993dea40] [te.agent.main] {} Agent starting up
2021-07-26 21:40:55.889 INFO  [993dea40] [te.agent.main] {} No agent id found, attempting to obtain one
2021-07-26 21:40:55.889 INFO  [993dea40] [te.agent.ClusterMasterAdapter] {} Attempting to get agent id from sc1.thousandeyes.com
2021-07-26 21:40:56.897 ERROR [993dea40] [te.agent.main] {} Error calling create_agent: Curl error - Peer certificate cannot be authenticated with given CA certificates
```
## TE Proxy interop Verification - this time with proxy SSL cert installed into TE
```sh
C9300#app-hosting connect appid cat9kte session /bin/bash
root@c9300p27-te:/# tail -f /var/log/agent/te-agent.log 
2021-07-26 22:45:41.225 DEBUG [c17fa700] [te.agent.DataSubmitter] {} Trying to get results for https://data1.agt.thousandeyes.com/mixed
2021-07-26 22:45:41.225 DEBUG [c17fa700] [te.agent.DataSubmitter] {} Trying to get results for https://data1.agt.thousandeyes.com/screenshots
2021-07-26 22:45:41.226 DEBUG [c17fa700] [te.agent.DataSubmitter] {} Trying to get results for https://data1.agt.thousandeyes.com/load
2021-07-26 22:45:53.924 DEBUG [a5feb700] [te.agent.AgentRelayStreamHandler] {} Proxying enabled
2021-07-26 22:45:53.925 DEBUG [a5feb700] [te.agent.AgentRelayStreamHandler] {} Establishing control channel connection to c1.thousandeyes.com
2021-07-26 22:45:57.473 DEBUG [ca418a40] [te.agent.NtpClient] {} Sending NTP packet to pool.ntp.org (82.197.164.46)
2021-07-26 22:46:00.476 WARN  [ca418a40] [te.agent.ClockOffsetCalculator] {} Error getting NTP clock offset: No response from server
2021-07-26 22:46:00.506 INFO  [ca418a40] [te.agent.ClockOffsetCalculator] {} NTP clock offset expired. Using estimated clock offset 19 seconds
2021-07-26 22:46:00.506 INFO  [ca418a40] [te.agent.main] {} Done calling check in
2021-07-26 22:46:00.510 INFO  [ca418a40] [te.agent.main] {} Controller tells us we are enabled
2021-07-26 22:46:00.515 DEBUG [ca418a40] [te.agent.main] {} Commit thread sleeping for 27 seconds
2021-07-26 22:46:03.925 DEBUG [a5feb700] [te.agent.AgentRelayStreamHandler] {} Control channel disconnected after 10 seconds
2021-07-26 22:46:03.926 DEBUG [a5feb700] [te.agent.AgentRelayStreamHandler] {} Reconnecting control channel in 30 seconds
  ```