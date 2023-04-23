# HashiCorp Vault considerations for production environment

This page will show you some considerations you should take while running hashiCorp Vault in a production environment.

1. [General Recommendations](#general-recommendations)
2. [Operating Systemss](#operating-systems)
3. [Vault Configurations](#vault-configurations)

## General Recommendations
1. Use "single tenancy" where possible


   - _Vault stores the encryption key in memory, the fewer shared resources, the better. We don't want other applications to have access to the storage backend encrypt key_.
2. If you're running on Kubernetes, make sure you have a dedicated cluster to run Vault.


   - _Exception for this rule includes: metrics agent collector, and log file agent collector._
3. If you're running Vault on VMs, you should aim for Vault dedicated nodes
   - _If you have more services running on the same VM, it will require more firewall rules, you should also remember that vault encrypt key lives on memory, and we don't want other services to access that_.

   - _Such as, `kubectl exec -ti pod`, SSH/RDP_. Instead, you should access vault using the CLI or the API._
4. Allow only required ports on your firewall rules to reduce attack surface

   - _Default ports: Vault: 8200, 8201 | Consul: 8500, 8201,8301._
   
   - _If you're using a Cloud Platform to authenticate to Vault, you'll have to allow also tcp/443._
   
   - _If you're using Vault Secret Database, make sure to allow port 3306._
  
   - _If you're authenticating with LDAP, the port used is tcp/389._
   
   - _If you're using Consul, make sure the [network diagram](https://developer.hashicorp.com/vault/tutorials/day-one-consul/reference-architecture#recommended-architecture)_
   
   - _If you're using Raft, it does a make lot simpler, there's no need to open any extra port. Vault uses tcp/8200 for API communication ansd tcp/8201 for Raft Data Replication, see [diagram](https://developer.hashicorp.com/vault/tutorials/day-one-raft/raft-reference-architecture#recommended-architecture)_.
   
5. Aim for immutable upgrades

   - _Rather than you fixing broken nodes, you should aim into destroy that node and bring a new one into life because you know the result of the automation configuration of your infrastructure._
   
   - _If you're using Raft, ensure replication has been completed to newly added nodes before removing old nodes._
   
 ## Operating Systems
1. Never ever run Vault as root 
   - _It might expose Vault's data._
   
   - _Create a user named "vault" instead._
   
   - _Limit access to Vault configuration._ (`chmod -R vault:vault /opt/vault/data`)
 
2. Protect audit Vault directories, snapshots, binaries, plugins files (`chmod 740 -R /etc/vault.d/`)
3. Disable shell history
    - _Possible to discover tokens in history._
4. Patch the Operating System
5. Disable swap
    - _Disabling swap privdes an extra layer of protection, as previously mentioned, vault stores data in memory, unencrypted, data should never be written to disk_.
    - _Enable `mlock` to prevent swap_

## Vault Configuartions
