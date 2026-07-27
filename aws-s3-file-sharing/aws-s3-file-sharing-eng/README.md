<h1 align="center">🔐 Secure File Sharing with Amazon S3, IAM, and SNS</h1>

<p align="center">
  <img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/631ffb97-70fb-49d4-9651-fe28e76dd65c" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/Amazon%20SNS-DD344C?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon SNS">
  <img src="https://img.shields.io/badge/status-completed-brightgreen?style=for-the-badge" alt="Status">
</p>

<p align="center">
  Hands-on project configuring a secure environment for sharing files with an external user, applying the Principle of Least Privilege and automated audit notifications.
</p>

---

## 📚 About this project

This repository documents the configuration of an **Amazon S3 bucket shared with an external user**, simulating a real business scenario: a fictional coffee shop hires a media agency to manage product photos sold on their website.

The central focus of this project is not just "giving access to someone," but rather **giving the right access, in the right measure**, while maintaining visibility over everything modified within the environment.

---

## 1. Context and Problem Statement

This lab was conducted during the **Re/Start AWS AI + No Code Program** by **Escola da Nuvem**, as part of my learning journey in Cloud Computing and AWS Infrastructure.

The scenario stems from a common problem when companies need to collaborate with external vendors or partners: **how do you grant access to a cloud resource to someone outside the company without exposing the entire environment and without losing control over what they do?**

In this project, a fictional coffee shop hired a media agency to photograph products and update images used on the website. This external person (represented by the user `mediacouser`) needs to upload, update, and remove images in an S3 bucket, but **must not have access to anything else in the AWS environment**, let alone permission to modify the bucket's security settings.

Additionally, the coffee shop team needs real-time visibility whenever bucket content is modified, without having to manually check the console.

---

## 2. Objective

Configure a secure file-sharing environment using Amazon S3 by applying key concepts of:

- Granular access control via **IAM groups and users**;
- Practical application of the **Principle of Least Privilege**;
- Command automation via **AWS CLI**;
- **Automated notifications** for bucket events via Amazon SNS.

Beyond setting up permissions, the goal was to test and prove in practice that they work as expected—both for allowed actions and blocked actions.

---

## 🔐 Principle of Least Privilege

The **Principle of Least Privilege** is one of the most fundamental pillars of cloud security. It states that **any user, application, or service should receive only the strict permissions necessary to perform its intended function, and nothing more**.

In practice, this means that instead of granting broad access "for convenience," every permission is deliberated and justified individually: *why does this user need this specific action on this specific resource?*

**Why this principle matters:**

- **Reduces the attack surface**: the fewer permissions a user or credential holds, the smaller the potential impact if those credentials are compromised.
- **Limits the "blast radius" of errors**: a misconfigured user or a buggy script cannot affect resources outside its scope of permission.
- **Simplifies auditing and compliance**: granular, documented permissions make it straightforward to understand "who can do what" within an environment.
- **Prevents permission creep over time**, a common issue in real-world environments where users gain extra access rights and never lose them.

**How this principle was applied in this lab:**

The `mediacouser` is associated with an IAM group (`mediaco`) that has a custom policy (`mediaCoPolicy`) granting **only three types of permissions**, all restricted to a specific bucket prefix (`cafe-*/images/*`):

- List the bucket in the console;
- Read, add, and remove objects inside the `images/` folder.

This user **has no permission to modify bucket policies, adjust public permissions, or access any other resource in the account**. This restriction was validated in practice during the lab (see the key learnings section).

---

## 3. Solution Architecture

<p align="center">
  <img width="770" height="482" alt="Image" src="https://github.com/user-attachments/assets/6fdb5793-c24a-4c4f-9842-99370d50d9f8" />
</p>

**Solution Flow:**
