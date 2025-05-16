![Screenshot (7)](https://github.com/user-attachments/assets/7712c4ad-5289-4f2d-b673-676bbad79ef1)# Azure-to-GCP-Migration
This repository provides a comprehensive, step-by-step guide for migrating virtual machines from Microsoft Azure to Google Cloud. It covers the entire process, from configuring the environment in Azure to executing the migration and ensuring a seamless transition to Google Cloud. The guide is designed to help users manage the migration efficiently while maintaining system integrity and minimizing downtime.

## Prerequisites
Before starting the migration process, ensure the following tools and accounts are set up:

1. **Microsoft Azure Account:** You must have access to the Azure Portal with administrator privileges to export the VM.
2. **Google Cloud Account:** A Google Cloud account with sufficient IAM permissions to create and manage virtual machines.
3. **Google Cloud SDK:** Install Google Cloud SDK for managing GCP services.
     - link to download the CLI - https://cloud.google.com/sdk/docs/install
     - After opening the link, click on Download the **Google Cloud CLI Installer** after choosing your `Operating System`.
     - Once downloaded, Perform the steps to install the CLI.
     - Open the CMD and run command `gcloud components update` to update the components.
     - Then run the command `gcloud auth login` to login to the environment.
     - Enable the APIs, type:
          ```bash
          gcloud services enable vmmigration.googleapis.com servicemanagement.googleapis.com servicecontrol.googleapis.com iam.googleapis.com cloudresourcemanager.googleapis.com compute.googleapis.com
          ```

## What is Cloud Migration?

Cloud migration refers to the process of moving data, applications, or workloads from one cloud environment to another. like, Azure to Google Cloud migration involves transferring virtual machines, applications, or other resources from Microsoft Azure to Google Cloud Platform (GCP). This process includes exporting VM images, converting formats if needed, and reconfiguring services to ensure they work efficiently in GCP, while maintaining data integrity and minimizing disruptions.

### Phases of Migration

1. **Assessment & Planning:** Identify workloads to migrate, evaluate the compatibility between source (Azure) and target (Google Cloud), and create a migration strategy.
2. **Preparation:** Set up both Azure and Google Cloud environments, configure networking, security, and resource management. Back up data to ensure safety during migration.
3. **Migration:** Transfer VMs, applications, or services. For Azure to GCP migration, this involves exporting Azure VMs and importing them into GCP.
4. **Testing & Validation:** Validate that the migrated workloads function correctly in Google Cloud, ensuring performance, security, and availability match requirements.
5. **Optimization:** Fine-tune the migrated environment in GCP for performance improvements, cost savings, and scalability.
6. **Post-Migration Monitoring:** Continuously monitor the workloads for performance issues, security vulnerabilities, and ensure proper integration within GCP.

Let's begin the Migration Process!

## In the Azure Portal

## Step 1: Create a Resource Group
![ss1](https://github.com/user-attachments/assets/ca7a8977-33a9-4ad6-9c18-1a2077849f8b)

1. Open the Azure portal.
2. Use the search bar to find `Resource Groups` and click on `Create`.
3. Enter `Migrate2GCP-RG` as the **Resource Group Name**.
4. Choose `West US 2` (or your preferred region) for the **Region**.
5. Click `Review + Create`, then select `Create`.

## Step 2: Create a Virtual Network with a Subnet

![ss2](https://github.com/user-attachments/assets/b1fc6742-42e7-4c4c-932e-48fb423d2088)

1. In the search bar, look for `Virtual Networks` and click on `Create`.
2. Enter `MigrateVNet` as the **Name**.
3. Select `West US 2` for the **Region**.
4. Set the **Address Space** to `10.0.0.0/24`.
5. Add a Subnet:
   - **Web-Subnet:** Address range `10.0.0.0/27`.
6. Click `Review + Create`, then `Create`.

## Step 3: Create a Virtual Machine

![ss3](https://github.com/user-attachments/assets/fe64355d-dc09-4d87-a1b3-aa44d95ebf80)

1. In the search bar, look for `Virtual Machines` and select `Create`.
2. Enter `USW-WEB-VM` as the **Name**.
3. Choose `WestUS 2` for the **Region**.
4. Pick your preferred OS image (e.g., Ubuntu Server or Windows).
5. Under the Networking section, select `MigrateVNet` and `Web-Subnet`.
6. Click `Review + Create`, and then hit `Create`.

## Step 4: Host `Index.html` on Web VM (IIS Windows Server)

### 1. Promote WebVM to IIS Windows Server

![Screenshot (5)](https://github.com/user-attachments/assets/b43cdad1-19b6-472b-bd98-a98d6b390db5)

- After your Web VM is deployed, connect to it using Remote Desktop (RDP).
- Launch `Server Manager` on the Windows VM.
- In `Server Manager`, choose `Add roles and features`.
- Proceed through the wizard, and in the `Roles` section, select `Web Server (IIS)`.
- Finish the wizard and wait for the IIS role to install.

### 2. Navigate to the wwwroot Directory and Upload your HTML File

![Screenshot (6)](https://github.com/user-attachments/assets/86c97e51-3af6-4d89-9004-2d5498a153fc)



- Once IIS is installed, go to the following path on your VM: `C:\inetpub\wwwroot`.
- In the `wwwroot` folder, delete the default `index.html` file (if it exists).
- Upload your frontend HTML file into the `wwwroot` folder.
- Rename your HTML file to `index.html`.

### 3. Verify the Website

![Screenshot (62)](https://github.com/user-attachments/assets/3d6cf18b-f268-4220-ac42-e4090c872f87)


- In a browser, navigate to your Web VM’s public IP address: `http://<your-web-vm-public-ip>`.
- You should now see your frontend site, accessible via the Web VM's public IP.

For example, if your Web VM's public IP is 52.183.21.18, enter http://52.183.21.18 in your browser to view the site.

## Step 5: Create an App Registration
![Screenshot (55)](https://github.com/user-attachments/assets/5ad62d15-1690-4255-8c0e-4274816aa012)


1. In the search bar, search for `App Registration` and hit `New Registration`.
2. Enter the **Name** and hit `Register`.
3. Go to manage, inside the app registered and click on `Certificates & Secrets`.
4. Click on `New client secret`. Enter the description and click on `Add`.
5. Copy the **Value** in Notepad.

## Step 6: Add Custom Role

![Screenshot (56)](https://github.com/user-attachments/assets/552f0f56-bc55-47da-8819-bb0f9fe6bb80)
1. Go to Subscriptions, inside the subscription click on `Access control IAM`.
2. Click `Add` then `Add Custom Role` and then click on `Start from json`.
3. paste the below json file after entering your **SUBSCRIPTION_ID** and hit `Review + Create`.
```json
       {
  "properties": {
        "roleName": "Minimum M2VM permissions role",
        "description": "This role contains the bare minimum of Azure IAM permissions to support M2VM flow",
        "assignableScopes": [
              "/subscriptions/SUBSCRIPTION_ID"
        ],
  "permissions": [
              {
              "actions": [
                    "Microsoft.Resources/subscriptions/resourceGroups/write",
                    "Microsoft.Resources/subscriptions/resourceGroups/read",
                    "Microsoft.Resources/subscriptions/resourceGroups/delete",
                    "Microsoft.Compute/virtualMachines/read",
                    "Microsoft.Compute/virtualMachines/write",
                    "Microsoft.Compute/virtualMachines/deallocate/action",
                    "Microsoft.Compute/disks/read",
                    "Microsoft.Compute/snapshots/delete",
                    "Microsoft.Compute/snapshots/write",
                    "Microsoft.Compute/snapshots/beginGetAccess/action",
                    "Microsoft.Compute/snapshots/read",
                    "Microsoft.Compute/snapshots/endGetAccess/action"
              ],
              "notActions": [],
              "dataActions": [],
              "notDataActions": []
              }
        ]
  }
  }
  ```

## Step 7: Add Role Assignment
![Screenshot (57)](https://github.com/user-attachments/assets/f42b7315-3e0c-4539-8c9a-8f98ae7f06e4)


1. Go to Subscriptions, inside the subscription click on `Access control IAM`.
2. Click `Add` then `Add role assignment`.
3. Inside the `Role`. Search for your custom role like for me it is **Minimum M2VM permissions role**.
4. Inside the `Members`. click on `select members` and search for the `App Registration Name` for me it is **TestingGCPMigration**.
5. Click `Review + Assign`.

## In the Google Cloud

## Step 1: Click on Migrate to Virtual Machines



![Screenshot (58)](https://github.com/user-attachments/assets/5a26ac67-64a7-4536-8aa2-9dc693b1c352)

1. In the GCP, search for `Migrate to Virtual Machines`.
2. Click on `sources` and then `Add Source`, then hit the `Add Azure Source`.
3. Enter the **Name**, **GCP Region**, and **Azure Location** where the VM exists. Then, enter the following details from **Azure**:
   - **Subscription ID** of Azure.
   - **Client ID** from Azure, available inside `App Registration`.
   - **Tenant ID** from Azure, available inside `App Registration`.
   - **Client Secret Value** that was copied in `Step 5` of Azure.
4. Click on `Create`.

## Step 2: ADD the VM Migration
![Screenshot (59)](https://github.com/user-attachments/assets/959cc951-635c-472d-95d3-7dc81911486a)

1. Wait till the `source` status becomes **Active**.
2. Select the VMs, click on `Add Migrations` and then click on `VM Migration`.
3. Go to `VM Migration`. Select the VMs click on `Migration` and then hit `Start Replication`.

## Step 3: Edit Target Details


![Screenshot (60)](https://github.com/user-attachments/assets/5e9ba248-9290-410c-ac00-d2f9ac13fa46)


1. Click on `Edit target details` inside the `VM Migrations` after selecting the VMs.
2. Enter the **Instance Name**, **Project** and **Zone** inside the `General`.
3. Inside the `Machine Configuration`
   - Choose Machine type series - **e2**
   - Choose Machine type - **e2-highcpu-2**
   - On host maintenance, choose **Migrate VM Instance**
   - Set the Automatic Start to **OFF**
4. Inside the `Networking`, choose everything **Default**.
5. Inside the `Additional Configuration`
   - Choose service account as - **Compute Engine default service account**
   - disk type - **Balanced**
6. Click on `Save`.

## Step 4: Cut-Over and Test-Clone

![Screenshot (61)](https://github.com/user-attachments/assets/22668320-ceef-4a55-98e9-7371f31eb9dd)

1. Wait until the `Replication Status` becomes `Active` from `First sync`.
2. Select the VMs, click on `Cut-Over and Test-Clone` and then click on `Test-Clone`.

## Step 5: Testing the Migration

![Screenshot (7)](https://github.com/user-attachments/assets/11005866-83e7-47b0-a27a-2fd538dba3c5)


1. Search for the `Compute Engine`, inside it go to `VM Instances`.
2. Copy the External IP of VM and run the RDP.
3. Use the same Username and Password which you set for Azure VM.
4. In the VM, check for the site is still hosting or not. TYPE `localhost` in the browser.

Congratulations!, You have successfully Migrate your VM from Azure to Google Cloud.

## Contact

For any questions, doubts, or clarifications regarding this repository, feel free to reach out:

- Email: mailto:dev.pradhumn2108@gmail.com
- LinkedIn: https://www.linkedin.com/in/pradhumn-singh-bhadoria-0a9291226/
   
   


















