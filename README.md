# Cloud-Computing-Practicals
## Practical 2 : Launch your first amazon EC2 Instance
## **Step 1: Launch the EC2 Instance**

1.  **Access the Dashboard:** Sign in to the [AWS Management Console](https://aws.amazon.com/console/) and search for **EC2**.
2.  **Start Launch Process:** Click the **Launch instance** button.
3.  **Name and Tags:** Enter a name for your server (e.g., `Linux-Server-01`).
4.  **Application and OS Images (AMI):** Select **Amazon Linux 2023** (or Amazon Linux 2). Ensure the "Free tier eligible" badge is visible.
5.  **Instance Type:** Select **t2.micro** (Free tier eligible).
6.  **Key Pair (Login):** * Click **Create new key pair**.
    * Name it `my-ec2-key`.
    * Select **RSA** and **.pem**.
    * **Download the file.** Move it to a secure folder like `~/.ssh/` or your Documents.

---

## **Step 2: Configure Security Groups**

1.  **Network Settings:** Look for the **Firewall (security groups)** section.
2.  **SSH Rule:** Ensure the "Allow SSH traffic from" checkbox is selected.
3.  **Security Best Practice:** Change the dropdown from "Anywhere (0.0.0.0/0)" to **My IP**. This prevents unauthorized users from attempting to brute-force your instance.
4.  **Launch:** Review your summary and click **Launch instance**.

---

## **Step 3: Connect to the Instance via SSH**

Once the instance state is **Running**, follow these steps to log in:

### **On macOS, Linux, or Windows (Modern Terminal)**

1.  **Open your Terminal** and navigate to the directory where your `.pem` file is stored:
    ```bash
    cd ~/Downloads
    ```

2.  **Update Permissions:** You must restrict the permissions of your private key file for SSH to work.
    ```bash
    chmod 400 my-ec2-key.pem
    ```

3.  **Identify your Public DNS:** In the AWS Console, click on your instance ID and copy the **Public IPv4 DNS** (looks like `ec2-xx-xx-xx-xx.compute-1.amazonaws.com`).

4.  **Execute Connection:** Run the following command:
    ```bash
    ssh -i "my-ec2-key.pem" ec2-user@YOUR_PUBLIC_DNS_HERE
    ```

5.  **Confirm:** Type `yes` when prompted about the authenticity of the host.

---

## **Step 4: Cleanup (Optional but Recommended)**

To avoid any accidental charges after you are finished testing:
1.  Go to the **Instances** page.
2.  Select your instance.
3.  Click **Instance State** > **Terminate instance**.

---
<img width="1710" height="1112" alt="Screenshot 2026-04-16 at 7 56 53 PM" src="https://github.com/user-attachments/assets/d3a85cbe-78cb-487a-99da-bce3aaef05d4" />
<img width="3420" height="2224" alt="Screenshot 2026-04-16 at 8 23 55 PM" src="https://github.com/user-attachments/assets/ea37273e-23f9-4593-9b36-390543b048a5" />
<img width="3420" height="2224" alt="Screenshot 2026-04-16 at 8 29 45 PM" src="https://github.com/user-attachments/assets/b5347da3-af78-46bd-b0de-9c3076293fb7" />

## Practical 3 : Set up a VPC 
## **Part 1: Create the VPC & Subnets**

1.  **VPC Creation:**
    * Navigate to **VPC Dashboard** > **Your VPCs** > **Create VPC**.
    * Select **VPC only**.
    * **Name tag:** `MyCustomVPC`
    * **IPv4 CIDR block:** `10.0.0.0/16`
2.  **Subnet Creation:**
    * Go to **Subnets** > **Create subnet**.
    * **Public Subnet:** * VPC: `MyCustomVPC`
        * Name: `Public-Subnet`
        * Availability Zone: Select one (e.g., `us-east-1a`)
        * IPv4 CIDR: `10.0.1.0/24`
    * **Private Subnet:**
        * VPC: `MyCustomVPC`
        * Name: `Private-Subnet`
        * Availability Zone: Select the *same* AZ as above for simplicity.
        * IPv4 CIDR: `10.0.2.0/24`

---

## **Part 2: Configure Public Internet Access**

1.  **Internet Gateway (IGW):**
    * Go to **Internet Gateways** > **Create internet gateway**. Name it `My-IGW`.
    * Select `My-IGW` > **Actions** > **Attach to VPC** > Select `MyCustomVPC`.
2.  **Public Route Table:**
    * Go to **Route Tables** > **Create route table**. Name it `Public-RT`.
    * **Edit routes:** Click **Add route**. 
        * Destination: `0.0.0.0/0`
        * Target: **Internet Gateway** (`My-IGW`).
    * **Subnet associations:** Click **Edit subnet associations** and select `Public-Subnet`.

---

## **Part 3: Configure Private Outbound Access (NAT)**

1.  **NAT Gateway:**
    * Go to **NAT Gateways** > **Create NAT gateway**.
    * **Subnet:** Select `Public-Subnet` (Crucial: NAT Gateways must reside in a public subnet).
    * **Connectivity type:** Public.
    * Click **Allocate Elastic IP**.
2.  **Private Route Table:**
    * Go to **Route Tables** > **Create route table**. Name it `Private-RT`.
    * **Edit routes:** Click **Add route**.
        * Destination: `0.0.0.0/0`
        * Target: **NAT Gateway** (select the one you just created).
    * **Subnet associations:** Select `Private-Subnet`.

---

## **Part 4: Launching the Instances**

1.  **Public Instance (Bastion Host):**
    * Launch an EC2 (`t2.micro`).
    * **Network:** Select `MyCustomVPC` and `Public-Subnet`.
    * **Auto-assign public IP:** Enable.
    * **Security Group:** Allow SSH (Port 22) from your IP.
2.  **Private Instance:**
    * Launch another EC2 (`t2.micro`).
    * **Network:** Select `MyCustomVPC` and `Private-Subnet`.
    * **Auto-assign public IP:** Disable.
    * **Security Group:** Allow SSH (Port 22) from the *Security Group of the Public Instance*.

---

## **Part 5: Verification (The "Jump" Test)**

1.  **SSH into Public Instance:**
    ```bash
    ssh -i "key.pem" ec2-user@PUBLIC_IP
    ```
2.  **SSH into Private Instance:**
    From inside the Public Instance, use the Private Instance's **Private IP**:
    ```bash
    ssh ec2-user@PRIVATE_IP
    ```
<img width="1710" height="1112" alt="Screenshot 2026-04-16 at 8 45 15 PM" src="https://github.com/user-attachments/assets/9ed40ad3-9174-4de9-bca3-42885fd9ceea" />
<img width="1710" height="1112" alt="Screenshot 2026-04-16 at 8 45 40 PM" src="https://github.com/user-attachments/assets/da3e9e1e-577f-4be1-96ad-473ada52ef85" />
<img width="1710" height="1112" alt="Screenshot 2026-04-16 at 8 52 45 PM" src="https://github.com/user-attachments/assets/c1501ec8-506a-47b1-92f3-d60056fabed7" />
<img width="1706" height="488" alt="Screenshot 2026-04-16 at 9 11 55 PM" src="https://github.com/user-attachments/assets/73e2b939-89a9-4805-b406-53eef814fd26" />
<img width="1706" height="346" alt="Screenshot 2026-04-16 at 9 20 37 PM" src="https://github.com/user-attachments/assets/634434c0-1e0e-4b84-88fb-f03788281743" />
<img width="1710" height="347" alt="Screenshot 2026-04-16 at 9 56 18 PM" src="https://github.com/user-attachments/assets/f39a464c-20f7-4db0-87c9-5e48f445c769" />
<img width="1710" height="615" alt="Screenshot 2026-04-16 at 9 58 45 PM" src="https://github.com/user-attachments/assets/1a3610cc-7d3e-4d31-b167-f9023fbd91af" />
<img width="1710" height="693" alt="Screenshot 2026-04-16 at 10 00 11 PM" src="https://github.com/user-attachments/assets/4721dd61-239f-48a7-ab69-b8d1db3cd421" />
<img width="1709" height="521" alt="Screenshot 2026-04-16 at 10 01 40 PM" src="https://github.com/user-attachments/assets/d6f6fb1c-4f54-4de7-bc26-a8c493857f36" />
<img width="1710" height="705" alt="Screenshot 2026-04-16 at 10 03 25 PM" src="https://github.com/user-attachments/assets/2bb656bf-d7bc-425c-bc02-3e744f154016" />
<img width="1710" height="582" alt="Screenshot 2026-04-16 at 10 06 51 PM" src="https://github.com/user-attachments/assets/ddc679f1-f587-463c-81e2-40c327a42caa" />
<img width="1710" height="644" alt="Screenshot 2026-04-16 at 10 07 07 PM" src="https://github.com/user-attachments/assets/09418a3b-a7a7-43e0-b8fe-ec20fe19254e" />
<img width="1708" height="416" alt="Screenshot 2026-04-16 at 10 08 56 PM" src="https://github.com/user-attachments/assets/078c8f63-a4b9-44c7-9dae-d99596f51b80" />
<img width="1709" height="897" alt="Screenshot 2026-04-16 at 10 13 32 PM" src="https://github.com/user-attachments/assets/497ae470-5ca1-41e5-9103-64bc6467077b" />
<img width="1710" height="668" alt="Screenshot 2026-04-16 at 10 14 06 PM" src="https://github.com/user-attachments/assets/5e92ec47-939e-4c5b-ae98-8e98b1a70700" />
<img width="1710" height="660" alt="Screenshot 2026-04-16 at 10 21 18 PM" src="https://github.com/user-attachments/assets/bbf3c8a4-40f1-4fb2-9f4b-7714b62077a3" />
<img width="1710" height="824" alt="Screenshot 2026-04-16 at 10 22 18 PM" src="https://github.com/user-attachments/assets/8a2a2b82-db0c-4782-af21-0b061c989cfa" />
<img width="1710" height="484" alt="Screenshot 2026-04-16 at 10 23 45 PM" src="https://github.com/user-attachments/assets/c1f1ca51-0304-4e82-9716-a9d58da5dd5f" />
<img width="1710" height="713" alt="Screenshot 2026-04-16 at 10 24 33 PM" src="https://github.com/user-attachments/assets/86a8d2b3-f6bc-4e14-93a8-84e681da865b" />
<img width="1710" height="554" alt="Screenshot 2026-04-16 at 10 25 29 PM" src="https://github.com/user-attachments/assets/d1eb70ea-3160-48c2-9bfa-2bca8078fe2a" />
<img width="1710" height="585" alt="Screenshot 2026-04-16 at 10 26 01 PM" src="https://github.com/user-attachments/assets/d333d038-9cc8-4585-b49e-258176c2bd1e" />
<img width="750" height="282" alt="Screenshot 2026-04-16 at 10 37 24 PM" src="https://github.com/user-attachments/assets/415beb35-d8a9-4b37-9488-01d7ce7c8ee0" />
<img width="911" height="443" alt="Screenshot 2026-04-16 at 10 44 01 PM" src="https://github.com/user-attachments/assets/458c0597-3f61-4478-92bf-657cece19f19" />

---


## Practical 4 : Configure Auto Scaling and Load Balancing
## **Part 1: Create a Launch Template**

The Launch Template defines *what* will be launched (AMI, Instance Type, Security Groups).

1.  Navigate to **EC2 Dashboard** > **Launch Templates** > **Create launch template**.
2.  **Name:** `Web-Server-Template`.
3.  **AMI:** Select **Amazon Linux 2023** (Free tier eligible).
4.  **Instance Type:** Select **t2.micro**.
5.  **Key Pair:** Select your existing `.pem` key.
6.  **Security Group:** Create a new group that allows **HTTP (Port 80)** from anywhere and **SSH (Port 22)** from your IP.
7.  **Advanced Details (User Data):** Scroll to the bottom and paste this script to install a web server automatically:
    ```bash
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "<h1>Hello from $(hostname -f)</h1>" > /var/www/html/index.html
    ```
8.  Click **Create launch template**.

---

## **Part 2: Deploy an Application Load Balancer (ALB)**

The ALB will be the single point of contact for users and will distribute traffic to the ASG.

1.  Go to **Load Balancers** > **Create load balancer**.
2.  Select **Application Load Balancer**.
3.  **Network Mapping:** Select your **VPC** and at least **two Availability Zones** (Public Subnets).
4.  **Security Groups:** Select the security group that allows HTTP (Port 80).
5.  **Listeners and Routing:**
    * Click **Create target group**.
    * Target type: **Instances**.
    * Target group name: `TG-Web-Servers`.
    * Protocol: **HTTP** | Port: **80**.
    * (Back in ALB tab) Select this new target group.
6.  Click **Create load balancer**.

---

## **Part 3: Create the Auto Scaling Group (ASG)**

1.  Go to **Auto Scaling Groups** > **Create Auto Scaling group**.
2.  **Name:** `My-Web-ASG`.
3.  **Launch Template:** Select `Web-Server-Template`.
4.  **Network:** Select your VPC and the same subnets used for the ALB.
5.  **Load Balancing:** Select **Attach to an existing load balancer** and choose your Target Group (`TG-Web-Servers`).
6.  **Group Size:** * Desired capacity: `2`
    * Minimum capacity: `1`
    * Maximum capacity: `4`
7.  **Scaling Policies:**
    * Select **Target tracking scaling policy**.
    * Metric type: **Average CPU utilization**.
    * Target value: `70`. (ASG will add instances if CPU goes above 70%).
8.  Click **Create Auto Scaling group**.

---

## **Part 4: Testing the Setup**

### **1. Test the Load Balancer**
1.  Copy the **DNS name** of your ALB (e.g., `my-alb-123.us-east-1.elb.amazonaws.com`).
2.  Paste it into your browser. You should see "Hello from [hostname]".
3.  Refresh multiple times; you should see the hostname change as the ALB switches between your two instances.

### **2. Simulate High Traffic (Stress Test)**
To see if your scaling policy works, you can manually stress the CPU of one instance:
1.  SSH into one of your running instances.
2.  Install a stress tool:
    ```bash
    sudo amazon-linux-extras install epel -y
    sudo yum install stress -y
    ```
3.  Run the stress command:
    ```bash
    stress --cpu 1 --timeout 300
    ```
4.  Watch the **EC2 Instances** page. Within a few minutes, AWS CloudWatch will trigger the ASG to launch a 3rd instance.

---
<img width="1704" height="767" alt="Screenshot 2026-04-16 at 11 23 43 PM" src="https://github.com/user-attachments/assets/9ea2c59b-da15-44aa-894c-5b6673be58b0" />
<img width="1708" height="894" alt="Screenshot 2026-04-16 at 11 27 18 PM" src="https://github.com/user-attachments/assets/796f6a4a-db4a-4ae2-ba68-54de84bfea89" />
<img width="1702" height="277" alt="Screenshot 2026-04-16 at 11 28 04 PM" src="https://github.com/user-attachments/assets/2b87373e-2e25-49a3-915a-074d6ad28547" />
<img width="1695" height="894" alt="Screenshot 2026-04-16 at 11 30 33 PM" src="https://github.com/user-attachments/assets/75ec9f5d-b282-4a12-8f7a-93dc75489736" />
<img width="849" height="468" alt="Screenshot 2026-04-16 at 11 32 42 PM" src="https://github.com/user-attachments/assets/d0d4a414-8503-4bdd-a071-165582d4e249" />
<img width="1710" height="918" alt="Screenshot 2026-04-16 at 11 34 27 PM" src="https://github.com/user-attachments/assets/e6901d1f-7c3c-4929-9211-c215f6aae283" />
<img width="1705" height="887" alt="Screenshot 2026-04-16 at 11 36 08 PM" src="https://github.com/user-attachments/assets/99637d27-0f75-4d22-8c83-470211452cac" />
<img width="846" height="575" alt="Screenshot 2026-04-16 at 11 40 01 PM" src="https://github.com/user-attachments/assets/cc721aa1-86cd-4a05-97a0-d9d3bed9ec2d" />
<img width="1014" height="288" alt="Screenshot 2026-04-16 at 11 40 27 PM" src="https://github.com/user-attachments/assets/e159b02c-72a0-4da2-a086-31087c1e93ef" />
<img width="965" height="247" alt="Screenshot 2026-04-16 at 11 42 30 PM" src="https://github.com/user-attachments/assets/738156bc-41ed-406c-95bc-b919fb01ce10" />

## Practical 05 — Deploying a Static Website on the Cloud
## **Step 1: Create an S3 Bucket**

1.  Log in to the [AWS Management Console](https://aws.amazon.com/console/) and search for **S3**.
2.  Click **Create bucket**.
3.  **Bucket Name:** Enter a globally unique name (e.g., `my-unique-static-site-2026`).
4.  **Region:** Choose the AWS Region closest to your users.
5.  **Object Ownership:** Leave as **ACLs disabled (recommended)**.
6.  **Block Public Access settings:** * **Uncheck** the box "Block *all* public access". 
    * Acknowledge the warning that the bucket will become public. (This is required for website hosting).
7.  Click **Create bucket**.

---

## **Step 2: Upload Website Content**

1.  Click on your newly created bucket name.
2.  Click **Upload** and add your website files (e.g., `index.html`, `style.css`, and any images).
    * *Example `index.html` content:*
      ```html
      <!DOCTYPE html>
      <html>
      <head><title>My Cloud Site</title></head>
      <body><h1>Successfully Hosted on S3!</h1></body>
      </html>
      ```
3.  Click **Upload** at the bottom of the page.

---

## **Step 3: Enable Static Website Hosting**

1.  Go to the **Properties** tab of your bucket.
2.  Scroll to the bottom to find **Static website hosting** and click **Edit**.
3.  Select **Enable**.
4.  **Hosting type:** Choose "Host a static website".
5.  **Index document:** Enter `index.html`.
6.  Click **Save changes**.
7.  **Note:** After saving, scroll back down to this section. You will see a **Bucket website endpoint** URL. *It will not work yet until Step 4 is complete.*

---

## **Step 4: Configure Permissions (Bucket Policy)**

Even though we unblocked public access in Step 1, we still need to grant "Read" permission to everyone for the files inside.

1.  Go to the **Permissions** tab.
2.  Scroll down to **Bucket policy** and click **Edit**.
3.  Paste the following JSON policy (replace `YOUR-BUCKET-NAME` with your actual bucket name):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
        }
    ]
}

```
<img width="1710" height="950" alt="Screenshot 2026-04-16 at 11 56 24 PM" src="https://github.com/user-attachments/assets/ac44ea54-e92d-4946-9466-15f768a09753" />
<img width="1708" height="566" alt="Screenshot 2026-04-16 at 11 57 00 PM" src="https://github.com/user-attachments/assets/94c7ec74-cbd1-4662-ac40-595ee4e86f49" />
<img width="1706" height="370" alt="Screenshot 2026-04-17 at 12 02 57 AM" src="https://github.com/user-attachments/assets/efc9ecb2-4758-4bfb-a0d3-55f14465969f" />
<img width="1710" height="422" alt="Screenshot 2026-04-17 at 12 03 05 AM" src="https://github.com/user-attachments/assets/75770322-f7d1-4ee5-8179-a8930a051484" />
<img width="1710" height="664" alt="Screenshot 2026-04-17 at 12 03 16 AM" src="https://github.com/user-attachments/assets/736e909b-1a99-4320-9100-f1a715b31184" />
<img width="1710" height="628" alt="Screenshot 2026-04-17 at 12 03 45 AM" src="https://github.com/user-attachments/assets/01ce319a-ae33-4866-9dc8-91c7e158b00e" />
<img width="1710" height="452" alt="Screenshot 2026-04-17 at 12 04 20 AM" src="https://github.com/user-attachments/assets/e6f46583-2195-4016-bbb3-ed90025173e9" />
<img width="1710" height="782" alt="Screenshot 2026-04-17 at 12 06 46 AM" src="https://github.com/user-attachments/assets/50b15c9d-8fe9-4ed4-9279-d26caf2b6e35" />
<img width="1710" height="853" alt="Screenshot 2026-04-17 at 12 08 07 AM" src="https://github.com/user-attachments/assets/2771d8d3-6a5d-485a-8539-2f614b3860d3" />
<img width="1710" height="851" alt="Screenshot 2026-04-17 at 12 12 18 AM" src="https://github.com/user-attachments/assets/7d9a6a9e-3a0d-42f4-8778-ff3ed76f9a67" />
<img width="1710" height="893" alt="Screenshot 2026-04-17 at 12 09 09 AM" src="https://github.com/user-attachments/assets/2dbdf4e4-2ebb-4ea0-9157-ed1c7ff2d62f" />
<img width="1710" height="717" alt="Screenshot 2026-04-17 at 12 14 48 AM" src="https://github.com/user-attachments/assets/9c333b02-6f41-49b7-a2da-494a2ef801fb" />
<img width="1710" height="824" alt="Screenshot 2026-04-17 at 12 18 28 AM" src="https://github.com/user-attachments/assets/2d72227d-28c0-438b-a412-d9478acf3bcf" />
<img width="1344" height="105" alt="Screenshot 2026-04-17 at 12 33 09 AM" src="https://github.com/user-attachments/assets/4907f041-b0ff-4ec2-86c5-a67566e8287f" />
<img width="1014" height="632" alt="Screenshot 2026-04-17 at 12 33 45 AM" src="https://github.com/user-attachments/assets/a213bb72-f8c7-4142-9a69-4a6a28b65b4d" />

