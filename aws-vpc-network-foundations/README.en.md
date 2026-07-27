<h1 align="center">🌐 Secure Network Architecture with Amazon VPC, Bastion Host, and NAT Gateway</h1>

<p align="center">
  <img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/bbfb0985-a56f-4c83-a37c-4063da34fbbd" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon VPC">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/NAT%20Gateway-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="NAT Gateway">
  <img src="https://img.shields.io/badge/status-completed-brightgreen?style=for-the-badge" alt="Status">
</p>

<p align="center">
  Hands-on project for building an isolated private network on AWS, with public and private subnets, controlled access via a bastion server, and internet egress protected by a NAT Gateway.
</p>

---

## 📚 About this Project

This repository documents the construction of a **segmented network architecture on AWS**, separating resources that need to be reachable from the internet from those that must stay isolated while still needing outbound connectivity.

The project simulates a very common scenario in corporate environments: how to structure a cloud network where only what's strictly necessary is publicly exposed, while everything else stays protected but still operational.

---

## 1. Context and Problem

This lab was carried out during the **Re/Start AWS AI + No Code Program**, offered by **Escola da Nuvem**, as part of my learning journey in Cloud Computing and AWS Infrastructure.

The scenario starts from a classic cloud network architecture problem: **how do you design an environment where some resources need to be directly accessible from the internet, while others (such as databases or internal servers) must be completely inaccessible from outside, yet still be able to update themselves and communicate with external services when needed?**

Exposing every resource directly to the internet is a risky practice: any publicly accessible instance becomes a potential target. On the other hand, completely isolating a resource with no internet egress at all is also impractical, since internal servers frequently need to download updates, packages, or communicate with external APIs.

This project resolves that dilemma using a widely adopted AWS architecture pattern: segmentation into public and private subnets, with a bastion server controlling administrative access and a NAT Gateway controlling outbound traffic from the private subnet.

---

## 2. Objective

Build a functional VPC, applying fundamental cloud network architecture concepts:

- Separating resources into **public and private subnets**;
- Controlling administrative access via a **bastion server** (jump box);
- Secure outbound connectivity in private subnets via a **NAT Gateway**;
- Configuring **route tables** to correctly direct local, public, and private traffic.

---

## 3. Solution Architecture

<p align="center">
  <img width="750" height="471" alt="Image" src="https://github.com/user-attachments/assets/6a8fd33f-d66f-4a70-9965-6523c37c4536" />
</p>

**Solution flow:**

```
Internet
   │
   ▼
Internet Gateway
   │
   ▼
Public Subnet (10.0.0.0/24)
   ├── Bastion Host  ◀── administrative access (SSH)
   └── NAT Gateway
             │
             ▼
Private Subnet (10.0.2.0/23)
   └── Private instance  ── outbound-only internet access, via NAT Gateway
```

The VPC (`10.0.0.0/16`) was split into a public subnet and a private subnet. The public subnet receives direct traffic from the internet through an Internet Gateway, and is where the bastion server and the NAT Gateway live. The private subnet has no direct route to the internet: all outbound traffic must go through the NAT Gateway, and the only administrative access path is through the bastion server.

---

## 4. What Was Done

Summary of the main steps carried out in this lab:

1. **Creation of the "Lab VPC"** with CIDR block `10.0.0.0/16` and DNS hostnames enabled.
2. **Creation of a public subnet** (`10.0.0.0/24`), configured to automatically assign public IPs to instances launched in it.
3. **Creation of a private subnet** (`10.0.2.0/23`), without automatic public IP assignment.
4. **Creation and attachment of an Internet Gateway** (`Lab IGW`) to the VPC, enabling communication between the public subnet and the internet.
5. **Route table configuration**: a public route table with a `0.0.0.0/0` route pointing to the Internet Gateway (associated with the public subnet), and a private route table, initially with only the local route.
6. **Launching the bastion server** (`Bastion Server`) in the public subnet, with a security group allowing SSH access.
7. **Creation of a NAT Gateway** in the public subnet, with an allocated Elastic IP.
8. **Updating the private route table**, adding a `0.0.0.0/0` route pointing to the NAT Gateway, giving the private subnet outbound internet access.
9. **Optional challenge: launching a private instance**, with a security group allowing SSH only from the VPC's internal range (`10.0.0.0/16`) — that is, only from the bastion.
10. **Logging into the private instance through the bastion server**, first connecting to the bastion via EC2 Instance Connect and, from there, via SSH to the private instance.
11. **Testing internet connectivity from the private instance** using `ping`, confirming that outbound traffic was correctly passing through the NAT Gateway.

---

## 5. Tools and Services Used

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon VPC">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/Internet%20Gateway-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Internet Gateway">
  <img src="https://img.shields.io/badge/NAT%20Gateway-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="NAT Gateway">
  <img src="https://img.shields.io/badge/Security%20Groups-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="Security Groups">
</p>

<br>

| Tool / Service | Purpose in the Project |
|---|---|
| **Amazon VPC** | Isolated virtual network, with public and private subnets |
| **Internet Gateway** | Bidirectional connectivity between the public subnet and the internet |
| **NAT Gateway** | Outbound connectivity for the private subnet, without exposure to inbound connections |
| **Amazon EC2** | Bastion server (administrative access) and private instance (protected resource) |
| **Security Groups** | Instance-level inbound traffic control (SSH allowed only from specific sources) |
| **Route Tables** | Directing local, public, and private traffic within the VPC |

---

## 6. Key Challenges and Lessons Learned

**The subnet's name doesn't determine whether it's public**
One of the most important lessons from this lab was realizing that calling a subnet "public" doesn't actually make it public. What really determines that is the **route table associated with it**: only after associating the subnet with a table that has a route to the Internet Gateway does it get real connectivity to the internet.

**Why use a bastion server instead of exposing everything directly**
Understanding the bastion host (jump box) pattern made it clear why it's so widely used in real-world architectures: instead of exposing every internal resource directly to the internet, there's a single controlled and monitored entry point. If that single access point is well protected, the entire internal network becomes more secure.

**The difference between an Internet Gateway and a NAT Gateway**
At first, the two concepts seemed similar, but the lab made the difference very clear in practice: the Internet Gateway allows both inbound and outbound traffic for resources with a public IP (like the bastion), while the NAT Gateway allows only outbound traffic, without exposing the private subnet to connections initiated from outside. It's a small difference in explanation, but a huge one in terms of security.

**Validating the configuration with a real test, not just assuming it worked**
Running `ping` from the private instance to confirm outbound internet access reinforced an important habit: after configuring a network, it's essential to test end-to-end connectivity, instead of assuming the configuration is correct just because no error showed up in the console.

---

## 7. Final Result

By the end of the project, the environment had a fully functional VPC, with:

- ✅ Public and private subnets correctly configured, each with its own route table;
- ✅ A bastion server reachable from the internet, serving as the single administrative entry point;
- ✅ A private instance with no direct internet exposure, but with working egress through the NAT Gateway;
- ✅ Connectivity validated in practice, with chained login (bastion → private instance) and an external access test via `ping`.

<!--
  📌 PLACEHOLDER: AWS CONSOLE SCREENSHOTS
  Include screenshots here of the AWS console showing:
  - The VPC and the subnets created
  - The public and private route tables, with their respective routes
  - The bastion server and the private instance running
  - The ping command output confirming egress via the NAT Gateway
-->

---

## 8. Skills Developed

<p align="center">
  <img src="https://img.shields.io/badge/Cloud%20Computing-232F3E?style=flat-square&logo=amazonaws&logoColor=white" alt="Cloud Computing">
  <img src="https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=flat-square&logo=amazonaws&logoColor=white" alt="Amazon VPC">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=flat-square&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/Networking-8C4FFF?style=flat-square&logo=amazonaws&logoColor=white" alt="Networking">
</p>

- ☁️ Cloud Computing (AWS)
- 🌐 Amazon VPC (subnets, route tables, gateways)
- 🔐 Bastion Host architecture pattern
- 🔁 NAT Gateway for secure outbound connectivity
- 🛠️ Network connectivity troubleshooting and validation
- 📝 Architectural reasoning applied to cloud networking

---

<p align="center">
  <sub>Project developed as part of the <strong>Re/Start AWS AI + No Code Program</strong>, Escola da Nuvem</sub>
</p>
