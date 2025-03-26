s# Milestone 1: Infrastructure as Code Setup - Architecture Details

## Terraform Configuration Updates

### Updated Module Structure

```
terraform/
├── modules/
│   ├── hetzner/
│   │   ├── main.tf           # Hetzner Cloud resources
│   │   ├── variables.tf      # Input variables
│   │   └── outputs.tf        # Output values
│   ├── cloudflare/
│   │   ├── main.tf           # Cloudflare DNS and Workers
│   │   ├── variables.tf      # Input variables
│   │   └── outputs.tf        # Output values
│   ├── azure/
│   │   ├── main.tf           # Azure Speech Services
│   │   ├── variables.tf      # Input variables
│   │   └── outputs.tf        # Output values
│   └── qdrant/
│       ├── main.tf           # Qdrant Cloud resources
│       ├── variables.tf      # Input variables
│       └── outputs.tf        # Output values
├── environments/
│   ├── production/
│   │   ├── main.tf           # Production environment
│   │   ├── variables.tf      # Environment-specific variables
│   │   └── terraform.tfvars  # Variable values (git-crypted)
│   └── staging/
│       ├── main.tf           # Staging environment
│       ├── variables.tf      # Environment-specific variables
│       └── terraform.tfvars  # Variable values (git-crypted)
└── global/
    ├── main.tf               # Global resources
    ├── variables.tf          # Global variables
    └── terraform.tfvars      # Variable values (git-crypted)
```

### Hetzner Cloud Configuration

```hcl
# modules/hetzner/main.tf

resource "hcloud_server" "swarm_manager" {
  count       = var.manager_count
  name        = "swarm-manager-${count.index + 1}"
  server_type = var.manager_type
  image       = var.server_image
  location    = var.location
  ssh_keys    = var.ssh_keys
  
  labels = {
    role = "manager"
  }
}

resource "hcloud_server" "swarm_worker" {
  count       = var.worker_count
  name        = "swarm-worker-${count.index + 1}"
  server_type = var.worker_type
  image       = var.server_image
  location    = var.location
  ssh_keys    = var.ssh_keys
  
  labels = {
    role = "worker"
  }
}

resource "hcloud_volume" "postgres_data" {
  name      = "postgres-data"
  size      = var.postgres_volume_size
  server_id = hcloud_server.swarm_manager[0].id
  format    = "ext4"
}

resource "hcloud_volume" "redis_data" {
  name      = "redis-data"
  size      = var.redis_volume_size
  server_id = hcloud_server.swarm_manager[0].id
  format    = "ext4"
}

resource "hcloud_network" "swarm_network" {
  name     = "swarm-network"
  ip_range = var.network_range
}

resource "hcloud_network_subnet" "swarm_subnet" {
  network_id   = hcloud_network.swarm_network.id
  type         = "server"
  network_zone = "eu-central"
  ip_range     = var.subnet_range
}

resource "hcloud_firewall" "swarm_firewall" {
  name = "swarm-firewall"
  
  rule {
    direction  = "in"
    protocol   = "tcp"
    port       = "22"
    source_ips = var.admin_ips
  }
  
  rule {
    direction  = "in"
    protocol   = "tcp"
    port       = "80"
    source_ips = ["0.0.0.0/0", "::/0"]
  }
  
  rule {
    direction  = "in"
    protocol   = "tcp"
    port       = "443"
    source_ips = ["0.0.0.0/0", "::/0"]
  }
  
  # Docker Swarm ports
  rule {
    direction  = "in"
    protocol   = "tcp"
    port       = "2377"
    source_ips = [var.subnet_range]
  }
  
  rule {
    direction  = "in"
    protocol   = "tcp"
    port       = "7946"
    source_ips = [var.subnet_range]
  }
  
  rule {
    direction  = "in"
    protocol   = "udp"
    port       = "7946"
    source_ips = [var.subnet_range]
  }
  
  rule {
    direction  = "in"
    protocol   = "udp"
    port       = "4789"
    source_ips = [var.subnet_range]
  }
}
```

### Cloudflare Configuration

```hcl
# modules/cloudflare/main.tf

resource "cloudflare_zone" "domain" {
  zone = var.domain_name
}

resource "cloudflare_record" "chatwoot" {
  zone_id = cloudflare_zone.domain.id
  name    = "chat"
  value   = var.load_balancer_ip
  type    = "A"
  proxied = true
}

resource "cloudflare_record" "api" {
  zone_id = cloudflare_zone.domain.id
  name    = "api"
  value   = var.load_balancer_ip
  type    = "A"
  proxied = true
}

resource "cloudflare_worker_script" "llm_query_worker" {
  name    = "llm-query-worker"
  content = file("${path.module}/workers/llm_query_worker.js")

  kv_namespace_binding {
    name         = "SECRETS"
    namespace_id = var.kv_namespace_id
  }
}

resource "cloudflare_worker_script" "aircall_webhook_worker" {
  name    = "aircall-webhook-worker"
  content = file("${path.module}/workers/aircall_webhook_worker.js")

  kv_namespace_binding {
    name         = "SECRETS"
    namespace_id = var.kv_namespace_id
  }
}

resource "cloudflare_worker_script" "guesty_webhook_worker" {
  name    = "guesty-webhook-worker"
  content = file("${path.module}/workers/guesty_webhook_worker.js")

  kv_namespace_binding {
    name         = "SECRETS"
    namespace_id = var.kv_namespace_id
  }
}

resource "cloudflare_worker_script" "voice_processing_worker" {
  name    = "voice-processing-worker"
  content = file("${path.module}/workers/voice_processing_worker.js")

  kv_namespace_binding {
    name         = "SECRETS"
    namespace_id = var.kv_namespace_id
  }
}

resource "cloudflare_r2_bucket" "audio_storage" {
  name   = "audio-storage"
  region = "auto"
}
```

### Azure Speech Services Configuration

```hcl
# modules/azure/main.tf

resource "azurerm_resource_group" "speech_services" {
  name     = "speech-services-rg"
  location = var.location
}

resource "azurerm_cognitive_account" "speech_services" {
  name                = "speech-services-account"
  location            = azurerm_resource_group.speech_services.location
  resource_group_name = azurerm_resource_group.speech_services.name
  kind                = "SpeechServices"
  sku_name            = "S0"
}

resource "azurerm_key_vault" "speech_services_vault" {
  name                = "speech-services-vault"
  location            = azurerm_resource_group.speech_services.location
  resource_group_name = azurerm_resource_group.speech_services.name
  tenant_id           = var.tenant_id
  sku_name            = "standard"

  access_policy {
    tenant_id = var.tenant_id
    object_id = var.object_id

    key_permissions = [
      "Get", "List", "Create", "Delete", "Update",
    ]

    secret_permissions = [
      "Get", "List", "Set", "Delete",
    ]
  }
}

resource "azurerm_key_vault_secret" "speech_key" {
  name         = "speech-key"
  value        = azurerm_cognitive_account.speech_services.primary_access_key
  key_vault_id = azurerm_key_vault.speech_services_vault.id
}

resource "azurerm_key_vault_secret" "speech_region" {
  name         = "speech-region"
  value        = azurerm_cognitive_account.speech_services.location
  key_vault_id = azurerm_key_vault.speech_services_vault.id
}
```

## CI/CD Pipeline Updates

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml

name: Deploy Chatwoot

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  terraform:
    name: 'Terraform'
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout
      uses: actions/checkout@v2
      
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v1
      
    - name: Terraform Init
      run: terraform init
      working-directory: ./terraform/environments/production
      
    - name: Terraform Validate
      run: terraform validate
      working-directory: ./terraform/environments/production
      
    - name: Terraform Plan
      run: terraform plan
      working-directory: ./terraform/environments/production
      
    - name: Terraform Apply
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      run: terraform apply -auto-approve
      working-directory: ./terraform/environments/production
  
  ansible:
    name: 'Ansible'
    runs-on: ubuntu-latest
    needs: terraform
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    
    steps:
    - name: Checkout
      uses: actions/checkout@v2
      
    - name: Setup Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.9'
        
    - name: Install Ansible
      run: pip install ansible
      
    - name: Run Ansible Playbook
      run: ansible-playbook -i inventory/production playbooks/deploy.yml
      working-directory: ./ansible
```

### Secrets Management

```yaml
# .github/workflows/secrets.yml

name: Secrets Management

on:
  workflow_dispatch:

jobs:
  update_secrets:
    name: 'Update Secrets'
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout
      uses: actions/checkout@v2
      
    - name: Setup git-crypt
      run: |
        sudo apt-get update
        sudo apt-get install -y git-crypt
        
    - name: Unlock Secrets
      run: |
        echo "${{ secrets.GIT_CRYPT_KEY }}" | base64 -d > git-crypt-key
        git-crypt unlock git-crypt-key
        rm git-crypt-key
        
    - name: Update Cloudflare KV Secrets
      run: |
        curl -X PUT "https://api.cloudflare.com/client/v4/accounts/${{ secrets.CF_ACCOUNT_ID }}/storage/kv/namespaces/${{ secrets.CF_KV_NAMESPACE_ID }}/values/CHATWOOT_API_KEY" \
          -H "Authorization: Bearer ${{ secrets.CF_API_TOKEN }}" \
          -H "Content-Type: text/plain" \
          --data-binary @secrets/chatwoot_api_key.txt
          
        curl -X PUT "https://api.cloudflare.com/client/v4/accounts/${{ secrets.CF_ACCOUNT_ID }}/storage/kv/namespaces/${{ secrets.CF_KV_NAMESPACE_ID }}/values/AZURE_SPEECH_KEY" \
          -H "Authorization: Bearer ${{ secrets.CF_API_TOKEN }}" \
          -H "Content-Type: text/plain" \
          --data-binary @secrets/azure_speech_key.txt
          
        curl -X PUT "https://api.cloudflare.com/client/v4/accounts/${{ secrets.CF_ACCOUNT_ID }}/storage/kv/namespaces/${{ secrets.CF_KV_NAMESPACE_ID }}/values/GROQ_API_KEY" \
          -H "Authorization: Bearer ${{ secrets.CF_API_TOKEN }}" \
          -H "Content-Type: text/plain" \
          --data-binary @secrets/groq_api_key.txt
```

## Cloud Provider Configuration

### Hetzner Cloud Setup

```
┌─────────────────────────────────────────────────────────────────┐
│                        Hetzner Cloud                            │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                     Docker Swarm Cluster                    ││
│  │                                                             ││
│  │  ┌─────────────────┐   ┌─────────────────┐   ┌────────────┐ ││
│  │  │                 │   │                 │   │            │ ││
│  │  │  Manager Nodes  │   │  Worker Nodes   │   │  Volumes   │ ││
│  │  │  (3 instances)  │   │  (2+ instances) │   │            │ ││
│  │  │                 │   │                 │   │            │ ││
│  │  └─────────────────┘   └─────────────────┘   └────────────┘ ││
│  │                                                             ││
│  │  ┌─────────────────┐   ┌─────────────────┐   ┌────────────┐ ││
│  │  │                 │   │                 │   │            │ ││
│  │  │  Network        │   │  Firewall       │   │  Load      │ ││
│  │  │  Configuration  │   │  Rules          │   │  Balancer  │ ││
│  │  │                 │   │                 │   │            │ ││
│  │  └─────────────────┘   └─────────────────┘   └────────────┘ ││
│  │                                                             ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Cloudflare Configuration

```
┌─────────────────────────────────────────────────────────────────┐
│                        Cloudflare                               │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  DNS            │   │  Workers        │   │  R2 Storage     ││
│  │  Configuration  │   │                 │   │                 ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  KV Storage     │   │  SSL/TLS        │   │  Page Rules     ││
│  │  (Secrets)      │   │  Configuration  │   │                 ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Azure Speech Services

```
┌─────────────────────────────────────────────────────────────────┐
│                        Azure                                    │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Resource       │   │  Cognitive      │   │  Key Vault      ││
│  │  Group          │   │  Services       │   │                 ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```