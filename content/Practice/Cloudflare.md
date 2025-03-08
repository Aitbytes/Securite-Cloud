---
title: Setting up DNS with Cloudflare
---
### Setting Up Cloudflare for Terraform Integration

Before executing the Terraform script, you need to set up Cloudflare and obtain the necessary credentials. Follow these steps:

>[!info]
>This step assume you've already bought a domain from a provider.

#### Step 1: Create a Cloudflare Account

1. Go to the [Cloudflare website](https://www.cloudflare.com/).
2. Click "Sign Up" and create a new account by providing your email and a secure password.
3. Verify your email address by following the instructions sent to your email.

#### Step 2: Add Your Domain to Cloudflare

1. Log in to your Cloudflare account.
2. Click "Add a Site" and enter your domain name.
3. Select a Cloudflare plan (the free plan is sufficient for basic DNS management).
4. Follow the instructions to update your domain's nameservers to point to Cloudflare's nameservers. This step is crucial for Cloudflare to manage your DNS records.

#### Step 3: Obtain a Cloudflare API Token

1. In the Cloudflare dashboard, click on your profile icon in the top right corner and select "My Profile".
2. Navigate to the "API Tokens" tab.
3. Click "Create Token" and select "Create Custom Token".
4. Set permissions for the token:
   - **Zone**: DNS: Edit
   - **Zone**: Zone: Read
5. Specify the zone resources the token can access (e.g., all zones or specific zones).
6. Click "Continue to Summary" and then "Create Token".
7. Copy the generated API token and store it securely.


By following these steps, you will have set up Cloudflare to manage DNS records through Terraform, enabling efficient deployment and management of your cloud infrastructure.
