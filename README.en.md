<p align="right">
  <img src="https://img.shields.io/badge/🇺🇸-English-blue?style=for-the-badge" alt="English version">
  <a href="./README.md"><img src="https://img.shields.io/badge/🇧🇷-Portuguese-green?style=for-the-badge" alt="Versão em Português"></a>
</p>

<h1 align="center">☁️ AWS Project Portfolio: Cloud & Infrastructure</h1>

<p align="center">
  <img width="2752" height="1536" alt="AWS Project Portfolio" src="https://github.com/user-attachments/assets/6e09692a-e358-4310-8008-76d9230e0699" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/Amazon%20VPC-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon VPC">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
</p>

<p align="center">
  🇺🇸 Documentation in English (🇧🇷 Portuguese version available above)
</p>

<p align="center">
  A collection of hands-on Cloud Computing and AWS Infrastructure projects, documented with a focus on architecture, security, and technical reasoning — not just execution steps.
</p>

---

## 📚 About this repository

This repository brings together the hands-on labs I've been developing during the **Re/Start AWS AI + No Code Program**, from **Escola da Nuvem**, turned into professionally documented projects.

The goal here goes beyond simply "completing labs": each project documents the business problem behind the technical solution, the architecture decisions made, the challenges faced, and what was learned along the way. The idea is that anyone, technical or not, can understand not only **what** was done, but **why** it was done that way.

The projects are organized by focus area: storage and hosting, security and access control, and network architecture.

---

## 🗂️ Documented Projects

### 🪣 Storage and Hosting

**[🪣 Static Website Hosting with Amazon S3](./aws-s3-static-website-hosting/README.en.md)**

Creation of an Amazon S3 bucket configured to host a static website, with file uploads via the AWS CLI, controlled public access policies, and an automation script for repeatable site updates.

---

**[🗄️ Storage Management with Amazon EBS, IAM Role, and Amazon S3](./aws-storage-management/README.en.md)**

Automated backup routine for Amazon EBS volumes via scheduled snapshots, with an IAM Role granting secure access between instances and Amazon S3, plus file synchronization with versioning enabled for recovering deleted data.

### 🔐 Security and Access Control

**[🔐 Secure File Sharing with Amazon S3, IAM, and SNS](./aws-s3-file-sharing/README.en.md)**

Setup of a secure file-sharing environment with an external user, applying the Principle of Least Privilege through restricted IAM groups and policies, with automatic email notifications whenever the bucket's content changes.

### 🌐 Network Architecture

**[🌐 Secure Network Architecture with Amazon VPC, Bastion Host, and NAT Gateway](./aws-vpc-network-foundations/README.en.md)**

Construction of a network segmented into public and private subnets, with a bastion server controlling administrative access and a NAT Gateway ensuring secure outbound connectivity for resources isolated from the internet.

---

**[🖥️ Multi-AZ Network with Amazon VPC and a Highly Available Web Server](./aws-vpc-web-server-deployment/README.en.md)**

Construction of a network distributed across two Availability Zones based on a client architecture specification, with a web server published and automatically provisioned via a startup script (User Data).

---

## 🧭 How this repository is organized

Each project has its own README, following a consistent pattern:

- **Context and Problem:** the business scenario that motivated the technical solution.
- **Objective:** what the project set out to solve.
- **Solution Architecture:** diagram and explanation of the flow between components.
- **What Was Done:** the actual step-by-step of the stages executed.
- **Tools and Services Used:** the technical stack employed.
- **Main Challenges and Lessons Learned:** difficulties encountered and the reasoning behind the solutions.
- **Final Result:** what was delivered at the end of the project.

---

<a id="contact"></a>

## 📬 Contact

| Platform | Contact |
|:----------|:--------|
| 💼 **LinkedIn** | <a href="https://www.linkedin.com/in/lucaspimentabarretto" target="_blank">linkedin.com/in/lucaspimentabarretto</a> |
| 💻 **GitHub** | <a href="https://github.com/LucasPBar" target="_blank">github.com/LucasPBar</a> |
| 📧 **Email** | <a href="mailto:lucaspimenta1805@gmail.com">lucaspimenta1805@gmail.com</a> |

---

<p align="center">
  <sub>Portfolio developed as part of the <strong>Re/Start AWS AI + No Code Program</strong>, Escola da Nuvem</sub>
</p>
