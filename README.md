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
- Ansible automation script offers a DIY alternative to turn-key automation offered by Cisco DNA Center. Cisco DNA Center has built-in Application Deployment workflows. 

## References
- ThousandEyes, Enterprise Proxy Certificate installation: 
-- https://docs.thousandeyes.com/product-documentation/global-vantage-points/enterprise-agents/troubleshooting/installing-ca-certificates-on-enterprise-agents
- Cisco, ThousandEyes deployment guide
-- https://www.cisco.com/c/en/us/products/collateral/switches/catalyst-9400-series-switches/guide-c07-2431113.html

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
i) TE agent deployment and activation
-- Verify if IOSXE meets minimum requirement
-- Verify TE App is already deployed
-- If TE App is not deployed, provision basic configuration (TE Vlan, AppGigEthernet trunk, NAT, NAT ACL)
-- Check for presence of TE installer tar, and distribution if necessary
-- TE App install, activate and start
ii) Enterprise Proxy SSL Certificate distribution
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

## Sample execution
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