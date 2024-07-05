# PPDM Roles for Ansible


the Roles utilize the rest API by using the URI Method

You can find the Roles in the Roles Directory.  
In the main directory, you can find several Playbooks for example workflows.  
The vars/main.yaml essentially represents an interpolation from environment variables to ansible vars  

## Documentation
the modules have been forked from a larger repo I maintain personally, so they start here as a Version 1 and Documentation if not inline is WIP :-(

You can find a good Documentation and Examples from our "BOB" CLass at [Ansible Labs](https://bob-builds-labs.github.io/)

Also, usecases are described from [terraforming DPS](https://github.com/bottkars/terraforming-dps)



## Playbooks / Example Workflows

### Playbook Parameters

Parameter | Usage  | Default value 
------|---------------------|---  
ppdm_fqdn  |  |"https://{{ lookup('env','PPDM_FQDN') }}" 
ppdm_new_password  |  |"{{ lookup('env','PPDM_INITIAL_PASSWORD') }}"
ppdm_ntp_server  |  |"{{ lookup('env','PPDM_NTP_SERVERS') }}"
ppdm_setup_password  |  |"{{ lookup('env','PPDM_SETUP_PASSWORD') }}" # "admin"
ddve_username  |  |"sysadmin"
ppdm_sdr_path  |  |"{{ lookup('env','PPDM_SDR_PATH') }}"
ddve_password  |  |"{{ lookup('env','DDVE_PASSWORD') }}"
ddve_fqdn  |  |"{{ lookup('env','DDVE_PRIVATE_FQDN') }}"
ddve_address  |  |"{{ lookup('env','DDVE_PRIVATE_FQDN') }}"
ddve_repository_path  |  |"{{ lookup('env','PPDD_PATH') }}"
ppdm_smtp_mailServer  |  |"{{ lookup('env','PPDM_SMTP_SERVER') }}"
ppdm_smtp_port  |  |"{{ lookup('env','PPDM_SMTP_PORT') }}"
ppdm_smtp_mailFrom  |  |"{{ lookup('env','PPDM_SMTP_FROM') }}"
k8s_fqdn  |  |"{{ lookup('env','K8S_FQDN') }}"
ppdm_rule  |  |"ppdm_policy={{ lookup('env','PPDM_POLICY')}}"
k8s_port  |  |443
k8s_token  |  |"{{ lookup('env','PPDM_K8S_TOKEN') }}"
k8s_name  |  |"{{ lookup('env','K8S_CLUSTER_NAME') }}"
vcenter_username  |  |"{{ lookup('env','VCENTER_USERNAME') }}"
vcenter_password  |  |"{{ lookup('env','VCENTER_PASSWORD') }}"
vcenter_fqdn  |  |"{{lookup('env','VCENTER_FQDN') }}"
subscriptionId  |  |"{{lookup('env','AZURE_SUBSCRIPTION_ID') }}"
domain  |  |"{{lookup('env','AZURE_TENANT_ID') }}"
userKey  |  |"{{lookup('env','AZURE_CLIENT_ID') }}"
secretKey  |  |"{{lookup('env','AZURE_CLIENT_SECRET') }}"
storageaccount  |  |"{{lookup('env','AZURE_STORAGEACCOUNT') }}"
privateKey  |  |"{{ lookup('env','PPDM_PRIVATE_KEY') }}"
certificateChain  |  |"{{ lookup('env','PPDM_CERTIFICATE_CHAIN') }}"
licenseKey  |  |"{{ lookup('env','PPDM_LICENSE_KEY') }}"
ppdm_timeZone  |  |"{{ lookup('env','PPDM_TIMEZONE') }}"
rbac_source  |  |"{{ lookup('env','PPDM_RBAC_SOURCE', default='https://raw.githubusercontent.com/bottkars/ppdm-rbac/19.16/' ) }}"

### PPDM Configuration Playbooks ( 1.x - 10.x )



Runbook | Usage  | Parameters   
------|---------------------|---  
1.0-playbook_configure_ppdm.yml | Initial configuration of ppdm | ppdm_timeZone <br> ppdm_setup_password <br> ppdm_fqdn <br> ppdm_new_password 
1.0-playbook_wait_ready.yml | Wait for API to accept requests  |  ppdm_fqdn <br> ppdm_new_password 
1.1-playbook_get_ppdm_config.yml  |  Read the actual ppdm COnfiguration  |  ppdm_fqdn <br> ppdm_new_password  
1.2-playbook_disable_api_enforcement.yml  |  Disable / Enable Strict API Validation  | status,true or false  default false <br> ppdm_fqdn  <br> ppdm_new_password 
2.0-playbook_set_csm.yml  | Add Cloud Snapshot Manager to PPDM   |  ppdm_fqdn <br> ppdm_new_password <br>csm_tenant, tenant id of csm <br> csm_credentials, csm api token 
  |    |  



### Agent Playbooks ( 100.x)

Runbook | Usage  | Parameters   
------|---------------------|---  
100.0_list_agents.yml | list available Agents on PPDM server |  ppdm_fqdn <br> ppdm_new_password 
100.1_download_agents.yml  |  download agents to a local temp dir  |   ppdm_fqdn <br> ppdm_new_password <br> download_destination: /tmp
100.2_set_dns_for_agents.yaml  |  Enable DNS Resolution for Agents on PPDM Server  |  ppdm_fqdn <br> ppdm_new_password <br> state, default TRUE
100.3_playbook_check_AgentService.yaml  | Checks agent Service runnig on Windows machines   |  win_service call to inventory hosts
100.3_playbook_clear_hosts_windows_agent.yaml |  Removes localhost loopback entries from hosts file in windows  |  win_hosts call to inventory hosts
100.3_playbook_copy_and_deploy_windows_agent_awx.yaml  |  Deploys Windows Agents to inventory Hosts  |  agent_src: /tmp
100.4_create_whitelistentry_from_addressquery.yaml  |    | host_list, list of comma seperated hostnames

### Agent Policy Playbooks ( 120.x)

Runbook | Usage  | Parameters   
------|---------------------|---  
  |    |  


## Examples:


Also, most of the Variables may be looked up from ENV Vars, supporting the output from [terraforming DPS](https://github.com/bottkars/terraforming-dps)

```bash
# Refresh you Environment Variables if Multi Step !
eval "$(terraform output --json | jq -r 'with_entries(select(.key|test("^PP+"))) | keys[] as $key | "export \($key)=\"\(.[$key].value)\""')"
export PPDM_INITIAL_PASSWORD=Change_Me12345_
export PPDM_NTP_SERVERS='["169.254.169.254"]'
export PPDM_SETUP_PASSWORD=admin          # default password on the Cloud PPDM rest API
export PPDM_TIMEZONE="Europe/Berlin"
export PPDM_POLICY=PPDM_GOLD


```
Set the initial Configuration:    
```bash

ansible-playbook ~/workspace/ansible_ppdm/1.0-playbook_configure_ppdm.yml
```
verify the config:

```bash
ansible-playbook ~/workspace/ansible_ppdm/1.1-playbook_get_ppdm_config.yml
```
we add the DataDomain:  

```bash
ansible-playbook ~/workspace/ansible_ppdm/2.0-playbook_set_ddve.yml 
```
we can get the sdr config after Data Domain Boost auto-configuration for primary source  from PPDM

```bash
ansible-playbook ~/workspace/ansible_ppdm/3.0-playbook_get_sdr.yml
```
and see the dr jobs status
```bash
ansible-playbook ~/workspace/ansible_ppdm/31.1-playbook_get_activities.yml --extra-vars "filter='category eq \"DISASTER_RECOVERY\"'"
```

get protected assets
```bash
ansible-playbook $HOME/workspace/ansible_ppdm/30.0-playbook_get_assets.yml -e "asset_filter='protectionStatus eq \"PROTECTED\"'"
```

create a kubernetes policy and rule ...

```bash
ansible-playbook ~/workspace/ansible_ppdm/playbook_add_k8s_policy_and_rule.yml 
```

##Advanced Thingy´s

### Some advanced K8S
```json
{"details":{"k8s":{"configurations":[{"type":"CONTROLLER_CONFIG","key":"k8s.ppdm.vspherecsi.use.fsagent","value":"true"},{"type":"CONTROLLER_CONFIG","key":"k8s.docker.registry","value":"harbor.pks.home.labbuildr.com"},{"type":"CONTROLLER_CONFIG","key":"ppdm.backup.concurrency","value":"2"},{"type":"POD_CONFIG","key":"VELERO","value":"c3BlYzoKICB0ZW1wbGF0ZToKICAgIG1ldGFkYXRhOgogICAgICBsYWJlbHM6CiAgICAgICAgYXBwLmt1YmVybmV0ZXMuaW8vbmFtZTogdmVsZXJvLXBwZG0KICAgIHNwZWM6CiAgICAgIGltYWdlUHVsbFNlY3JldHM6CiAgICAgICAtIG5hbWU6IHJlZ2NyZWQ="},{"type":"CONTROLLER_CONFIG","key":"k8s.image.pullsecrets","value":"regcred"}]}}}
```


```bash
ansible-playbook ../ansible_ppdm/playbook_rbac_add_k8s_to_ppdm_reg_cred.yml \
--extra-vars="ppdm_fqdn=ppdm-demo.home.labbuildr.com ppdm_new_password=Change_Me12345_ rbac_source=/Users/bottk/workspace/ocs_vsphere/rbac/" \
-e="dockerconfigjson=eyJhdXRocyI6eyJoYXJib3IucGtzLmhvbWUubGFiYnVpbGRyLmNvbSI6eyJ1c2VybmFtZSI6InJvYm90JHBwZG1fYWNjb3VudCIsInBhc3N3b3JkIjoiTnJVU2ZWMGVPVFFmVGN0RHVienhNblV1a3IwRDBXbnkiLCJhdXRoIjoiY205aWIzUWtjSEJrYlY5aFkyTnZkVzUwT2s1eVZWTm1WakJsVDFSUlpsUmpkRVIxWW5wNFRXNVZkV3R5TUVRd1YyNTUifX19" \
-e='{"details":{"k8s":{"configurations":[{"type":"CONTROLLER_CONFIG","key":"k8s.ppdm.vspherecsi.use.fsagent","value":"true"},{"type":"CONTROLLER_CONFIG","key":"k8s.docker.registry","value":"harbor.pks.home.labbuildr.com"},{"type":"CONTROLLER_CONFIG","key":"k8s.image.pullsecrets","value":"regcred"},{"type":"POD_CONFIG","key":"VELERO","value":"c3BlYzoKICB0ZW1wbGF0ZToKICAgIG1ldGFkYXRhOgogICAgICBsYWJlbHM6CiAgICAgICAgYXBwLmt1YmVybmV0ZXMuaW8vbmFtZTogdmVsZXJvLXBwZG0KICAgIHNwZWM6CiAgICAgIGltYWdlUHVsbFNlY3JldHM6CiAgICAgICAtIG5hbWU6IHJlZ2NyZWQ="}]}}}'
```
### calling the playbooks directly with an external vars file.
```bash
ansible-playbook ansible_dps/ppdm/2.3-playbook_configure_cloud_dr_account.yml --extra-vars "@ppdm-prod/vars.yaml"
```
kubectl patch serviceaccount default  -p '{"imagePullSecrets": [{"name": "regcred"}]}'
