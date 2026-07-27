<h1 align="center">🖥️ Multi-AZ Network with Amazon VPC and a Highly Available Web Server</h1>

<p align="center">
  <img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/47667ffe-4ade-4db8-a64b-22bd9fff0d23" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon VPC">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/Apache-D22128?style=for-the-badge&logo=apache&logoColor=white" alt="Apache">
  <img src="https://img.shields.io/badge/status-completed-brightgreen?style=for-the-badge" alt="Status">
</p>

<p align="center">
  Hands-on project for building a network distributed across two Availability Zones, with a public web server automatically provisioned via a startup script.
</p>

---

## 📚 About this Project

This repository documents the construction of a **multi-AZ AWS network requested by a fictional client**, including the automated provisioning of a public web server within that structure.

Instead of assembling each component manually piece by piece, this project also explores using AWS's **VPC Wizard** to speed up the creation of the base infrastructure, along with a startup script (User Data) to get the server ready to use as soon as the instance boots up.

---

## 1. Context and Problem

This lab was carried out during the **Re/Start AWS AI + No Code Program**, offered by **Escola da Nuvem**, as part of my learning journey in Cloud Computing and AWS Infrastructure.

The scenario starts from a real business need: a client (represented in the lab as a Fortune 100 company) requested the construction of a custom AWS network to host a web application, following a specific architecture diagram provided by them.

The core problem here isn't just "spinning up a web server," but **delivering exactly the network architecture the client specified**: a VPC distributed across two Availability Zones, with public and private subnets in each, and a web server published in the public subnet of the second zone. This kind of requirement is common in corporate environments, where the network architecture is often already defined by architecture or security teams, and it's up to the person implementing it to follow those specifications precisely.

---

## 2. Objective

Build the network infrastructure requested by the client and publish a functional web server within it, applying:

- Creating a **VPC distributed across multiple Availability Zones**, using the VPC Wizard;
- Configuring **replicated public and private subnets** across two AZs, to support high availability;
- Creating a **security group** that allows only the necessary traffic (HTTP);
- Automated provisioning of a web server via a **startup script (User Data)**.

---

## 3. Solution Architecture

<p align="center">
  <img width="789" height="379" alt="Image" src="https://github.com/user-attachments/assets/557df15b-c0d1-4351-b860-39a47981f1dc" />
</p>

**Solution flow:**

```
Internet
   │
   ▼
Internet Gateway
   │
   ├──▶ Availability Zone A
   │       ├── Public Subnet 1 (10.0.0.0/24) ── NAT Gateway
   │       └── Private Subnet 1 (10.0.1.0/24)
   │
   └──▶ Availability Zone B
           ├── Public Subnet 2 (10.0.2.0/24) ── Web Server 1 (Security Group: HTTP)
           └── Private Subnet 2 (10.0.3.0/24)
```

The VPC (`10.0.0.0/16`) was built with subnets replicated across two Availability Zones, following exactly the diagram provided by the client. The web server was published in the public subnet of the second AZ, protected by a security group that allows only HTTP traffic (port 80) from any source.

---

## 4. What Was Done

Summary of the main steps carried out in this lab:

1. **Creation of the initial network structure via the VPC Wizard** (the "VPC and more" option), generating all at once: the `Lab VPC` (`10.0.0.0/16`), a public subnet and a private subnet in the first Availability Zone, an Internet Gateway, a NAT Gateway, and their respective route tables (public and private).
2. **Manual creation of a second public subnet** (`Public Subnet 2`, `10.0.2.0/24`) and a **second private subnet** (`Private Subnet 2`, `10.0.3.0/24`) in a second Availability Zone, to distribute the architecture across two AZs.
3. **Associating the new subnets with the correct route tables**: the second public subnet with the public route table, and the second private subnet with the private route table.
4. **Creation of a security group** (`Web Security Group`), allowing inbound HTTP traffic (port 80) from any IPv4 address.
5. **Launching the `Web Server 1` instance** in the public subnet of the second Availability Zone, with an automatic public IP and the security group created earlier.
6. **Configuring a startup script (User Data)** to automate server provisioning:

   ```bash
   #!/bin/bash
   # Install Apache Web Server and PHP
   yum install -y httpd mysql php
   # Download Lab files
   wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RESTRT-1/267-lab-NF-build-vpc-web-server/s3/lab-app.zip
   unzip lab-app.zip -d /var/www/html/
   # Turn on web server
   chkconfig httpd on
   service httpd start
   ```

7. **Verifying the instance**, waiting for the status checks (2/2) to confirm it was healthy.
8. **Accessing the web server through the browser**, using the instance's public IPv4 DNS, confirming the application was up and responding correctly.

---

## 5. Tools and Services Used

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon VPC">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/Security%20Groups-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="Security Groups">
  <img src="https://img.shields.io/badge/Apache-D22128?style=for-the-badge&logo=apache&logoColor=white" alt="Apache">
  <img src="https://img.shields.io/badge/PHP-777BB4?style=for-the-badge&logo=php&logoColor=white" alt="PHP">
  <img src="https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" alt="Bash">
</p>

<br>

| Tool / Service | Purpose in the Project |
|---|---|
| **Amazon VPC** | Virtual network distributed across two Availability Zones |
| **Amazon EC2** | Instance hosting the published web server |
| **Security Groups** | Inbound traffic control, allowing only HTTP |
| **Apache / PHP** | Stack used to serve the lab's web application |
| **Bash (User Data)** | Automating server installation and configuration at instance launch |

---

## 6. Key Challenges and Lessons Learned

**The VPC Wizard speeds things up, but doesn't replace understanding the architecture**
Using the VPC Wizard to create most of the infrastructure at once was much faster than configuring each piece manually, as I did in another VPC lab. But that only worked well because I already understood what each automatically generated component was doing (subnets, route tables, NAT Gateway). Using a wizard without understanding what it's creating behind the scenes would be risky in a real environment.

**Replicating the architecture in a second Availability Zone**
Having to manually create the second public and private subnets, and correctly associate them with the route tables, made it clear why multi-AZ architectures exist: distributing resources across physically separate zones is what ensures that a failure in a single zone doesn't take down the entire application.

**Automated provisioning via User Data**
Compared to manually installing and configuring a server, using a User Data script to install Apache, PHP, and download the application files automatically at instance launch showed how AWS lets you treat servers as disposable and re-creatable, rather than something manually configured and maintained "by hand" forever.

**Following a client's architecture specification**
Unlike other labs where I freely defined the names and structure, here there was a specific diagram provided by the "client" that had to be followed precisely (exact CIDRs, subnet names, correct Availability Zone). This closely resembled a real work situation, where the architecture is often already defined and the delivery needs to match exactly what was specified.

---

## 7. Final Result

By the end of the project, the requested network infrastructure was fully implemented and the web server was publicly accessible:

- ✅ VPC distributed across two Availability Zones, with public and private subnets replicated in each;
- ✅ Security group allowing only the strictly necessary traffic (HTTP);
- ✅ Web server automatically provisioned via User Data, with no manual intervention after launch;
- ✅ Application publicly accessible via the instance's DNS, confirming that the architecture requested by the client was successfully delivered.

<!--
  📌 PLACEHOLDER: AWS CONSOLE SCREENSHOTS
  Include screenshots here of the AWS console showing:
  - The VPC with subnets in both Availability Zones
  - The Web Security Group with the HTTP rule
  - The Web Server 1 instance with status check 2/2 passed
  - The web application page working in the browser
-->

---

## 8. Skills Developed

<p align="center">
  <img src="https://img.shields.io/badge/Cloud%20Computing-232F3E?style=flat-square&logo=amazonaws&logoColor=white" alt="Cloud Computing">
  <img src="https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=flat-square&logo=amazonaws&logoColor=white" alt="Amazon VPC">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=flat-square&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/High%20Availability-8C4FFF?style=flat-square&logo=amazonaws&logoColor=white" alt="High Availability">
  <img src="https://img.shields.io/badge/Bash-4EAA25?style=flat-square&logo=gnubash&logoColor=white" alt="Bash">
</p>

- ☁️ Cloud Computing (AWS)
- 🌐 Amazon VPC (multi-AZ, replicated subnets, route tables)
- 🖥️ Automated server provisioning via User Data
- 🔐 Security group configuration driven by actual traffic needs
- 📐 Implementing architecture from a client specification
- 🛠️ Verifying and validating the availability of a published application

---

<p align="center">
  <sub>Project developed as part of the <strong>Re/Start AWS AI + No Code Program</strong>, Escola da Nuvem</sub>
</p>
