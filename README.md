### Vault Server With Consul Storage Backend

### Note: 
#### Install the below tools first on your machine:
- docker
- docker-compose
- vault
- consul
##### Replace the following config file for consul-server.json to the existing one (consul/consul-server.json) 
	{
	  "datacenter": "dc1",
	  "key_file": "/etc/consul.d/certs/dc1-server-consul-0-key.pem",
	  "cert_file": "/etc/consul.d/certs/dc1-server-consul-0.pem",
	  "ca_file": "/etc/consul.d/certs/consul-agent-ca.pem",
	  "verify_incoming": true,
	  "verify_outgoing": true,
	  "encrypt": "rMsUDvFefgUboQEmwLjN//baklrVmNu7uQnblFmL8/0=",
	  "data_dir": "/consul/data",
	  "log_level": "DEBUG",
	  "server": true,
	  "ui": true,
	  "client_addr": "0.0.0.0",
	  "bind_addr": "0.0.0.0",
	  "bootstrap_expect": 1,
	  "ports": {
	    "dns": 53
	  }
	}
##### Startup Consul-server with docker-compose
    docker-compose up -d consul-server
##### check The result to see if consule-service container is up or not

##### Set the following env var 
    export CONSUL_HTTP_ADDR=REPLACE_ME_WITH_CONSUL_SERVER_IP:8500
##### Type this to see if you have any output or not
    consul members
##### Replace the following config file for consul-server.json to the existing one (consul/consul-server.json)
	{
	  "datacenter": "isc",
	  "datacenter": "dc1",
	  "key_file": "/etc/consul.d/certs/dc1-server-consul-0-key.pem",
	  "cert_file": "/etc/consul.d/certs/dc1-server-consul-0.pem",
	  "ca_file": "/etc/consul.d/certs/consul-agent-ca.pem",
	  "verify_incoming": true,
	  "verify_outgoing": true,
	  "encrypt": "rMsUDvFefgUboQEmwLjN//baklrVmNu7uQnblFmL8/0=",
	  "data_dir": "/consul/data",
	  "log_level": "DEBUG",
	  "server": true,
	  "ui": true,
	  "client_addr": "0.0.0.0",
	  "bind_addr": "0.0.0.0",
	  "bootstrap_expect": 1,
	  "acl": {
	   "enabled": true,
	   "default_policy": "deny",
	   "enable_token_persistence": true,
	  },
	  "ports": {
	    "dns": 53
	  }
	}

##### Restart Consul service with docker-compose
    docker-compose restart consul-server
##### check The result to see if consule-service container is up or not

##### Enable The consul ACL
    consul acl bootstrap
##### Set the HTTP token variable with the `SecretID` from the generated output in the previous step
    export CONSUL_HTTP_TOKEN=REPLACE_ME_WITH_SECRET_ID
##### Check if the consule Node is up and running again
    consul members
##### Create a generic agent token policy
    cd consul/ && consul acl policy create -name node-policy -description "Generic Rules for Nodes" -rules @generic-node-acl.hcl
##### Create the generic Agent Token
    consul acl token create -description "Agent Token" -policy-name node-policy
##### Replace the following config file for consul-server.json to the existing one (consul/consul-server.json)
	{
	  "datacenter": "isc",
	  "datacenter": "dc1",
	  "key_file": "/etc/consul.d/certs/dc1-server-consul-0-key.pem",
	  "cert_file": "/etc/consul.d/certs/dc1-server-consul-0.pem",
	  "ca_file": "/etc/consul.d/certs/consul-agent-ca.pem",
	  "verify_incoming": true,
	  "verify_outgoing": true,
	  "encrypt": "rMsUDvFefgUboQEmwLjN//baklrVmNu7uQnblFmL8/0=",
	  "data_dir": "/consul/data",
	  "log_level": "DEBUG",
	  "server": true,
	  "ui": true,
	  "client_addr": "0.0.0.0",
	  "bind_addr": "0.0.0.0",
	  "bootstrap_expect": 1,
	  "acl": {
	   "enabled": true,
	   "default_policy": "deny",
	   "enable_token_persistence": true,
	   "tokens": {
	     "agent": "REPLCAE_ME_WITH_AGENT_TOKEN"
	    }
	  },
	  "ports": {
	    "dns": 53
	  }
	}

##### Restart Consul service with docker-compose
    docker-compose restart consul-server
##### Check The result to see if consule-server container is up or not
##### Access the consul web ui
    http://REPLACE_ME_WITH_CONSUL_SERVER_IP:8500/ui/ 
### Token needed for login is the result of "echo $CONSUL_HTTP_TOKEN" 

##### Create the policy for vault
    cd consul/ && consul acl policy create -name vault-policy -rules @vault-policy.hcl
##### Create the token to allow vault cluster to use consul
    consul acl token create -description "Token for Vault Service" -policy-name vault-policy
    
##### Replace the following config file for consul-server.json to the existing one (vault/vault-server.hcl)

	listener "tcp" {
	    address = "0.0.0.0:8200"
	    cluster_address = "0.0.0.0:8201"
	    tls_disable = true
	}
	ui = true
	storage "consul" {
	    address = "REPLACE_ME_WITH_CONSUL_SERVER_IP:8500"
	    path = "vault"
	    token = "REPLACE_ME_WITH_TOKEN_TO_ALLOW_VAULT_NODE_TO_USE_CONSUL"
	}
	service_registration "consul" {
	    address = "REPLACE_ME_WITH_CONSUL_SERVER_IP:8500"
	    token = "REPLACE_ME_WITH_TOKEN_TO_ALLOW_VAULT_NODE_TO_USE_CONSUL"
	    service = "vault"
	}
	api_addr =  "http://REPLACE_ME_WITH_VAULT_SERVER_IP:8200"
	cluster_addr = "http://REPLACE_ME_WITH_VAULT_SERVER_IP:8201"
	
##### Startup vault-server with docker-compose
    docker-compose up -d consul-server
##### Set the Env var for vault_address
    export VAULT_ADDR=http://REPLACE_ME_WITH_VAULT_SERVER_IP:8200
##### initialize vault node
    vault operator init
##### Set the Env var for vault_token with the information generated
    export VAULT_TOKEN=REPLACE_ME_WITH_VAULT_ROOT_TOKEN
##### Unseal Vault Node
    vault operator unseal REPLACE_ME_WITH_FIRST_UNSEAL_KEY 
    vault operator unseal REPLACE_ME_WITH_SECOND_UNSEAL_KEY 
    vault operator unseal REPLACE_ME_WITH_THIRD_UNSEAL_KEY 
##### Access the vault web ui
    http://REPLACE_ME_WITH_VAULT_SERVER_IP:8200 
#### Token needed for login is the result of "echo $VAULT_TOKEN"
