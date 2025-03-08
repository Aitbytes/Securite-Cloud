---
title: 1-Setting up an environement
---
## Objective

Our goal is to create a standard cloud infrastructure, and leverage our understanding to highlight and exploit the vulnerabilities we would have left behind.

Throughout the labs, we will be leveraging IaC with terraform and ansible extensively, as it self-documents every step, while also making our configurations easily repeatable.

## Setting up authorisations
### Step 1: Create a Google Cloud Project

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Click on the project drop-down menu at the top of the page.
3. Click on "New Project" to create a new project.
4. Enter a name for your project and select a billing account.
5. Click "Create" to finalize the project creation.

![[unnamed.png]]

### Step 2: Set Up a Service Account

1. In the Google Cloud Console, navigate to the "IAM & Admin" section.
2. Click on "Service Accounts" in the left-hand menu.
3. Click "Create Service Account" at the top of the page.
4. Enter a name and description for the service account.
5. Click "Create and Continue".
6. Assign the necessary roles to the service account:
   - **Compute Viewer** (`roles/compute.viewer`)
   - **Compute Instance Admin (v1)** (`roles/compute.instanceAdmin.v1`)
   - **Compute OS Admin Login** (`roles/compute.osAdminLogin`)
   - **Compute Security Admin** (`roles/compute.securityAdmin`)
   - **Compute Image User** (`roles/compute.imageUser`)
   - **Service Account User** (`roles/iam.serviceAccountUser`)
7. Click "Done" to complete the service account setup.

---

You should have something similar to this following in the "IAM & Admin" > "IAM" section 

![[Pasted image 20250308020706.png]]
As you can see, we attributed the following rôles to the service account : 
- **Compute Viewer** (`roles/compute.viewer`):
    
    - This role allows users to view Compute Engine resources but not to modify them. It's useful for monitoring and auditing purposes.
- **Compute Instance Admin (v1)** (`roles/compute.instanceAdmin.v1`):
    
    - Provides permissions required to create, modify, and delete VM instances. Specifically, this role allows actions such as starting, stopping, and rebooting instances.
- **Compute OS Admin Login** (`roles/compute.osAdminLogin`):
    
    - This role allows users to connect to VM instances running Linux by granting administrator access via SSH.
- **Compute Security Admin** (`roles/compute.securityAdmin`):
    
    - Required to manage firewall rules and other security-related configurations, which might be necessary depending on your deployment and network setup.
- **Compute Image User** (`roles/compute.imageUser`):
    
    - Grants permission to use images to create and start instances. This role is essential when deploying instances from specific Linux images.
- **Service Account User** (`roles/iam.serviceAccountUser`):
    
    - Necessary to allow the deployment process to act as a service account. This is needed if instances are using service accounts for authentication to other Google Cloud services.

>[!warning]
> - Limit the use of the service account to specific tasks that absolutely require it. Avoid using it for general purposes, thereby respecting the principle of least privilege 

## Automating further deployments

Make sure you have Terraform or OpenTofu installed. Follow the specific [instructions for Terraform](https://developer.hashicorp.com/terraform/install) or [instructions for OpenTofu](https://opentofu.org/docs/intro/install) for your system. We will be using OpenTofu as it is open-source.

#### Step 1: Activate the Cloud Resource Manager API

1. In the Google Cloud Console, navigate to the "APIs & Services" section.
2. Click on "Library" in the left-hand menu.
3. Search for "Cloud Resource Manager API".
4. Click on the API and then click "Enable" to activate it.

![[Pasted image 20250308020516.png]]

Repeat for the following APIs :
- Compute Engine API
#### Step 2: Start a Tofu Project

1. Open your terminal or command prompt.
2. Create a new directory for your Tofu project:
   ```bash
   mkdir my-tofu-project
   cd my-tofu-project
   ```
3. Initialize a new Tofu project:
   ```bash
   tofu init
   ```
4. Create a new Tofu configuration file, e.g., `main.tf`, and define your infrastructure resources.

#### Step 3: Clone Our Repository

You can clone our repository to get started quickly with pre-configured scripts:

1. Clone the repository:
   ```bash
   git clone https://github.com/Aitbytes/Projet-Long-Infra
   ```
2. Navigate to the `k3s-php` directory:
   ```bash
   cd Projet-Long-Infra/k3s-php
   ```
3. Follow the instructions in the `README.md` file or execute the provided scripts to set up your environment.

>[!todo]
>Whether you which to use our script, or prefer creating yours, the following explanation on [[Provisioning|How to deploy with terraform]] will help you understand methodology.

>[!Success]
>If you are done with deploying the cluster, now it's time to [[Configuration|Configure Kubernetes with k3s and Ansible]].