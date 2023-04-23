# HashiCorp Vault considerations for production environments

This page will show you some considerations you should take while running hashiCorp Vault in a production environment. This doc also assumes you have the basic Vault knoweldge. 

1. [General Recommendations](#general-recommendations)
2. [Operating Systems](#operating-systems)
3. [Vault Configurations](#vault-configurations)
4. [When to alert](#when-to-alert)

# Recommendations

### General Recommendations
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
   
 ### Operating Systems 
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

### Vault Configurations 
1. Use TLS for vault nodes communication and API.
   - _If you're using LoadBalancer, Vault can terminate TLS or use pass through to the Vault nodes._
   - Don't set `tls_disabled` to 1 (true)
2. Enable audit devices
   - _If possible, more that one_
   - _Send Vault audit log to a centralized data collector server_
   - _Create alerts based on Vault Audit logs actions._
3. Avoid cleartext credentials
   - _Avoid using crendentials in configuration files, use Environment variable insteaf or cloud-integrated services_
4. Upgrade Vault frequently
5. Stop using root tokens
   - _Root token have unrestrited access and no TTL_
   - _Get rid of it after the initial. You can regenerate a root token in case of emergency_
6. Always check the integrity of the Vault binary.
   - _You can use HashiCopr checksun to validate._
7. Disable the UI if not in use
8. Secure the Unseal/Recovery Keys
   - _You should initialize Vault using PGP keys, it will distribute to multiple team members and no single person should have all the keys._
9. Use the smalles TTL possible for tokens and leases
   - _ It helps to reduce burden on the storage backend because the experied tokens are purged from Vault._
   - _It prevents renewals beyon reasonble timeframe_
10. Follow the principle of Least Privilege
   - _Only give access to the paths that are required_
   - _Try to avoid `*` and `+` in policies, when possible_
11. Perform Regular Backups
   - _Regular test backups to ensure functionality_.
12. Integrate Existing Identity Providers
   - _Do not use Vault local user/pass._

### When to alert (_Using vault audit and operational logs_)
- Use of root token
- Creation of a new root token
- Vault policy change
- Enabling new auth method
- Change/Creation auth role
- Permission defined responses
- Seal status of Vault
- Audit lgs Failures
- Transit Key Deletion
- Cloud-based resources changes
