---
title: 1-Provisioning the VMs
---
Let's dive into understanding this [Terraform](https://github.com/Aitbytes/Kubernetes-At-Home/tree/gcp) code and explore how it's used to configure a cloud infrastructure on Google Cloud Platform, collaborate with Ansible for setup management, and interface with Cloudflare for DNS records—all while maintaining a secure environment. We'll break down each part so that we can see how it all fits together like pieces of a puzzle.

### Setting the Stage with Terraform

Firstly, we're using **Terraform** as our infrastructure-as-code tool. In our `main.tf` file, we start by configuring the necessary providers. 

- **Providers** are plugins that allow Terraform to interact with cloud platforms and other services. In this setup, we're using:
  - **Google Cloud** provider to manage our cloud resources.
  - **Ansible** provider, which fits into this orchestration symphony, helping us with configuration management.
  - **Cloudflare** provider to integrate DNS management with our cloud instances.

```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
    ansible = {
      version = "~> 1.3.0"
      source  = "ansible/ansible"
    }
    cloudflare = {
      source  = "cloudflare/cloudflare"
      version = "~> 4.0"
    }
  }
}
```
### Google Cloud Provider Configuration

We've defined the Google Cloud provider with specific credentials and project settings:

 ```hcl
  credentials = file("./secrets/credentials.json")
  ```

  This line pulls in our secure credentials from a local file to authenticate against Google Cloud. 

- We also set `project`, `region`, and `zone` using variables (`var.project_id`, `var.region`, `var.zone`) so that we can easily switch environments or scale our resources across different locations without changing much in our code.

### Obtaining and Setting Up Google Cloud Credentials

To configure the Google Cloud provider, you need a service account with the appropriate permissions and a credentials file. Follow the steps in [[Setting up an environement]] to create the service account with the necessary permissions. After creating the service account follow these steps to obtain the credentials:

1. In the "Service Accounts" page, find your newly created service account.
2. Click on the service account to open its details.
![[Pasted image 20250308031206.png]]
3. Go to the "Keys" tab.
4. Click "Add Key" > "Create New Key".
5. Select "JSON" as the key type and click "Create".
6. The credentials file will be downloaded to your computer.
#### Step 3: Place the Credentials File

1. Move the downloaded JSON credentials file to the `secrets` directory.
2. Rename the file to `credentials.json` to match the reference in your Terraform configuration.

### Virtual Machine (VM) Configurations

We describe our VM configurations using a neat feature of Terraform called **locals**.

- Within `locals`, we've defined a list of VM configurations, detailing the **name**, **machine type**, and **tags** (e.g., "master" or "worker") for each. 

- The tags help in managing traffic routing and firewall configurations, as we'll see later.

```hcl
locals {
  vm_configs = [
    {
      name = "k3s-vm-1-m"
      machine_type = "e2-medium"  # 2 vCPU, 4GB memory
      tags = ["k3s", "master"]
    },
    {
      name = "k3s-vm-2-m"
      machine_type = "e2-medium"
      tags = ["k3s", "master"]
    },
    {
      name = "k3s-vm-3-m"
      machine_type = "e2-medium"
      tags = ["k3s", "master"]
    },
    {
      name = "k3s-vm-4-w"
      machine_type = "e2-medium"  
      tags = ["k3s", "worker"]
    },
    {
      name = "k3s-vm-5-w"
      machine_type = "e2-medium"
      tags = ["k3s", "worker"]
    }
  ]
}

```
### Creating VMs

We utilize the `google_compute_instance` resource here:

- Each VM uses a **boot disk** initialized with Ubuntu 20.04 LTS, a widely used OS known for stability and security.
  
- **Network configurations** allow these VMs to communicate. By default, they reside in `network = "default"`, ensuring they can participate in our broader network strategy.

- **SSH keys** are included to ensure secure access via SSH to these VMs, pulling the keys from our local system. This setup adheres to security best practices by not hardcoding credentials within our Terraform files.

```hcl
# Create VMs
resource "google_compute_instance" "vms" {
  count        = length(local.vm_configs)
  name         = local.vm_configs[count.index].name
  machine_type = local.vm_configs[count.index].machine_type
  zone         = var.zone

  boot_disk {
    initialize_params {
      image = "ubuntu-os-cloud/ubuntu-2004-lts"  # Use Ubuntu 20.04 LTS
      size  = 20  # GB
      type  = "pd-standard"  # Makes the disk persistent
    }
  }

  network_interface {
    network = "default"
    access_config {
      // Ephemeral public IP
    }
  }

  metadata = {
    ssh-keys = "user:${file("~/.ssh/id_rsa.pub")}"
  }

  tags = local.vm_configs[count.index].tags

  #metadata_startup_script = <<-EOF
  #            #!/bin/bash
  #            # Add any startup configuration here
  #            EOF

}

```
### Configuring Firewall Rules

Our firewall setup permits necessary communications:

- Using `google_compute_firewall`, SSH traffic and specific service ports are opened, aligning with our VM tags like "k3s". 

```hcl
resource "google_compute_firewall" "allow-ssh" {
  name    = "allow-ssh"
  network = "default"

  allow {
    protocol = "tcp"
    ports    = ["22",
                "80",
                "8080",
                "6444",
                "6443",
                "32002"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["k3s"]
}

```

>[!warning]
>The wide-open source range (`0.0.0.0/0`) means these ports can be accessed from anywhere—in practice, we’d ensure tighter security controls.


### Ansible Inventory

- The `ansible.tf` file creates an **inventory** for Ansible by dynamically generating a list of hosts, both master and worker nodes, to manage configurations across our VMs.

- By using a `templatefile`, it allows flexibility to adjust host lists without modifying the Ansible configuration directly, a vital practice for scaling efficiently and automatically.

```yml
all:
  children:
    master:
      hosts:
%{ for name, node in master_nodes ~}
        ${name}:
          ansible_host: ${node.external_ip}
          ansible_user: user
          # You can add more host variables here if needed
          # ansible_user: your_ssh_user
          # ansible_ssh_private_key_file: path_to_key
%{ endfor ~}
    worker:
      hosts:
%{ for name, node in worker_nodes ~}
        ${name}:
          ansible_host: ${node.external_ip}
          ansible_user: user
          # You can add more host variables here if needed
          # ansible_user: your_ssh_user
          # ansible_ssh_private_key_file: path_to_key
%{ endfor ~}
    k3s_cluster:
      ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
      children:
        master: {}
        worker: {}
```
### Cloudflare DNS Records 

Finally, DNS magic happens with Cloudflare:

- For VMs designated as master or worker nodes, we create appropriate DNS **A records** which map friendly domain names to IP addresses.
  
- Independent configuration for master and worker nodes ensures that each node’s addressability is maintained, helping in scenarios like load balancing or distributed application deployment.

- Follow the steps in  [[Cloudflare]] to obtain the necessary credential to execute this part of the script. 

>[!info]
>This step assume you already own a domain.

```hcl
provider "cloudflare" {
  api_token = var.cloudflare_api_token
}

resource "cloudflare_record" "master_dns" {
  for_each = {
    for instance in google_compute_instance.vms :
    instance.name => instance.network_interface[0].access_config[0].nat_ip
    if can(regex("-m$", instance.name))
  }

  zone_id = var.cloudflare_zone_id
  name    = each.key
  value   = each.value
  type    = "A"
  proxied = false
}

resource "cloudflare_record" "worker_dns" {
  for_each = {
    for instance in google_compute_instance.vms :
    instance.name => instance.network_interface[0].access_config[0].nat_ip
    if can(regex("-w$", instance.name))
  }

  zone_id = var.cloudflare_zone_id
  name    = each.key
  value   = each.value
  type    = "A"
  proxied = false
}
```

  
By understanding each piece of this Terraform configuration, we can see how it ensures efficient deployment, management, and scaling within the cloud.

