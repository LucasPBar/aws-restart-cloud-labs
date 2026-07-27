<h1 align="center">🔐 Secure File Sharing with Amazon S3, IAM, and SNS</h1>

<p align="center">
  <img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/b4f77405-4fe1-49c9-be7a-afe9f4aed9a3" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/Amazon%20SNS-DD344C?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon SNS">
  <img src="https://img.shields.io/badge/status-completed-brightgreen?style=for-the-badge" alt="Status">
</p>

<p align="center">
  A hands-on project configuring a secure environment for file sharing with an external user, applying the Principle of Least Privilege and automated audit notifications.
</p>

---

## 📚 About This Project

This repository documents the configuration of an **Amazon S3 bucket shared with an external user**, simulating a real-world business scenario: a fictional coffee shop hires a media company to manage the product photos used on its website.

The core focus of this project isn't just "granting someone access," but rather **granting the right access, in the right amount**, while maintaining visibility over everything that changes in the environment.

---

## 1. Context and Problem

This lab was carried out during the **Re/Start AWS IA + No Code Program**, by **Escola da Nuvem**, as part of my training journey in Cloud Computing and AWS Infrastructure.

The scenario stems from a common problem when companies need to collaborate with external vendors or partners: **how do you grant access to a cloud resource to someone outside the company, without exposing the entire environment and without losing control over what that person does?**

In this project's case, a fictional coffee shop hired a media company to photograph its products and update the images used on the website. This external user (represented by `mediacouser`) needs to be able to upload, update, and remove images in an S3 bucket, but **should not have access to anything else in the AWS environment**, and especially not permission to change the bucket's own security settings.

In addition, the coffee shop's team needs to know, in real time, whenever the bucket's content changes, without having to manually check the console.

---

## 2. Objective

Configure a secure file-sharing environment using Amazon S3, applying the following concepts:

- Granular access control via **IAM groups and users**;
- Practical application of the **Principle of Least Privilege**;
- Task automation via **AWS CLI**;
- **Automatic notifications** for bucket events via Amazon SNS.

Beyond simply configuring permissions, the goal was to test and prove in practice that they actually work as expected — both for what should be allowed and for what should be blocked.

---

## 🔐 Principle of Least Privilege

The **Principle of Least Privilege** is one of the most important pillars of cloud security. It states that **any user, application, or service should receive only the permissions strictly necessary to perform its function, and nothing beyond that**.

In practice, this means that instead of granting broad access "for convenience," each permission is thought through and justified individually: *why does this user need this specific action on this specific resource?*

**Why this principle matters:**

- **Reduces the attack surface**: the fewer permissions a user or credential has, the smaller the potential damage if those credentials are compromised.
- **Limits the "blast radius" of a mistake**: a misconfigured user or a buggy automation cannot affect resources outside its permission scope.
- **Makes auditing and compliance easier**: specific, documented permissions make it much simpler to understand "who can do what" in an environment.
- **Prevents privilege accumulation over time**, a common problem in real environments where users keep receiving extra access and never lose it.

**How this principle was applied in this lab:**

The user `mediacouser` is associated with an IAM group (`mediaco`) that has a custom policy (`mediaCoPolicy`) granting **only three types of permission**, all restricted to a specific bucket prefix (`cafe-*/images/*`):

- Listing the bucket in the console;
- Reading, adding, and removing objects inside the `images/` folder.

This user **has no permission to change bucket policies, adjust public access settings, or access any other resource in the account**. This restriction was validated in practice during the lab (see the learnings section).

---

## 3. Solution Architecture

<p align="center">
  <img width="770" height="482" alt="Image" src="https://github.com/user-attachments/assets/50c6d0cc-f95a-4c89-aa4d-9fa5db8acc98" />
</p>

**Solution flow:**

```
mediacouser (Console or AWS CLI)
        │
        ▼
Amazon S3 (Cafe S3 bucket)
        │
        ▼  (object creation/removal event)
Amazon SNS (s3NotificationTopic)
        │
        ▼
Administrator (receives email notification)
```

The external user `mediacouser` interacts with the S3 bucket only within the scope allowed by the `mediaco` group's policy. Every time an object is created or removed from the bucket, S3 publishes an event to the SNS topic `s3NotificationTopic`, which automatically sends an email notification to the administrator.

---

## 4. What Was Done

Summary of the main steps performed in this lab:

1. **Connected to the CLI Host instance via EC2 Instance Connect** and configured the AWS CLI with the credentials provided by the environment.
2. **Created the S3 bucket** (`cafe-xxxnnn`) via command line (`aws s3 mb`).
3. **Uploaded the initial set of images** to the bucket, under the `/images` prefix, using `aws s3 sync`.
4. **Reviewed the IAM group `mediaco`** and the `mediaCoPolicy` policy, understanding each of the three permissions granted (list bucket, list objects, and read/write/delete only within the `images/` prefix).
5. **Reviewed the IAM user `mediacouser`**, confirming it inherits permissions from the group, and created an access key for CLI use.
6. **Tested access as `mediacouser`** through the AWS Console: viewing an image, uploading a new image, and deleting an existing image — all actions succeeded as expected.
7. **Tested an unauthorized action**: attempted to change the bucket's permissions as `mediacouser`, which resulted in an access-denied error, confirming that the policy was correctly restrictive.
8. **Created the SNS topic `s3NotificationTopic`**, with an access policy allowing S3 to publish messages to it.
9. **Subscribed to the topic by email** and confirmed the subscription.
10. **Configured bucket event notifications** (`ObjectCreated` and `ObjectRemoved`, restricted to the `images/` prefix), linked to the SNS topic via a JSON file and AWS CLI.
11. **Ran tests via AWS CLI using `mediacouser`'s credentials**: object upload (triggered a notification), object read (did not trigger a notification, as expected), and object deletion (triggered a notification).
12. **Ran another unauthorized action test via CLI**: attempted to make an object public via `put-object-acl`, blocked with an `AccessDenied` error.

---

## 5. Tools and Services Used

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/Amazon%20SNS-DD344C?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon SNS">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/AWS%20CLI-232F3E?style=for-the-badge&logo=amazonwebservices&logoColor=white" alt="AWS CLI">
</p>

<br>

| Tool / Service | Purpose in the project |
|---|---|
| **Amazon S3** | Storage and sharing of product images |
| **AWS IAM** | Dedicated group and user for external access, with restricted permissions |
| **Amazon SNS** | Sending email notifications about changes to the bucket |
| **Amazon EC2 (EC2 Instance Connect)** | Access to the CLI Host instance used to run the commands |
| **AWS CLI** | Bucket creation, file uploads, and notification configuration |

---

## 6. Key Challenges and Learnings

**Understanding permissions by prefix, not just by bucket**
One of the most interesting aspects of the `mediaCoPolicy` policy was realizing that it grants access only to objects within `cafe-*/images/*`, not to the entire bucket. This made me rethink how to structure buckets in the future: organizing content by prefixes from the start makes it much easier to apply granular permissions later on.

**Confirming restrictions in practice, not just in theory**
Reading a JSON policy and understanding what it allows is one thing. Actually testing it — logged in as the restricted user itself, trying to do something outside its scope, and getting the access-denied error — is a completely different experience. That hands-on test gave me far more confidence that the configuration was correct than simply reviewing the policy text.

**Notifications as a visibility layer, not a control layer**
Configuring SNS made it clear that event notifications don't replace access control — they complement it. Even with correctly restricted permissions, having an automatic email whenever a file is created or removed adds an extra layer of traceability over what's happening in the environment, useful both for auditing and for detecting unexpected behavior.

**The difference between actions that trigger events and those that don't**
It was worth noting that reading an object (`get-object`) does not trigger a notification — only creation and removal do. This reinforces that the notification system was designed for state-change events, not for monitoring every type of access, which also has implications if it ever becomes necessary to audit reads.

---

## 7. Final Result

By the end of the project, the environment was configured with:

- ✅ A dedicated S3 bucket for sharing images with an external user;
- ✅ An IAM group and user with permissions restricted to the `images/` prefix, validated in practice (including testing what **should not** work);
- ✅ Automatic email notifications to the administrator whenever an object is created or removed.

<!--
  📌 PLACEHOLDER: AWS PANEL SCREENSHOTS
  Include screenshots here from the AWS console showing:
  - The mediaCoPolicy policy expanded in IAM
  - The access-denied test when attempting to change bucket permissions as mediacouser
  - The created SNS topic and confirmed subscription
  - A received notification email
-->

---

## 8. Skills Developed

<p align="center">
  <img src="https://img.shields.io/badge/Cloud%20Computing-232F3E?style=flat-square&logo=amazonaws&logoColor=white" alt="Cloud Computing">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=flat-square&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=flat-square&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/Amazon%20SNS-DD344C?style=flat-square&logo=amazonaws&logoColor=white" alt="Amazon SNS">
  <img src="https://img.shields.io/badge/AWS%20CLI-232F3E?style=flat-square&logo=amazonwebservices&logoColor=white" alt="AWS CLI">
</p>

- ☁️ Cloud Computing (AWS)
- 🔐 AWS IAM (groups, users, custom policies, and the Principle of Least Privilege)
- 🪣 Amazon S3 (prefix-based permissions, CLI management)
- 📩 Amazon SNS (topics, subscriptions, event notifications)
- 💻 AWS CLI and task automation
- 🛠️ Hands-on validation of security controls (testing both authorized and unauthorized access)

---

<p align="center">
  <sub>Project developed as part of the <strong>Re/Start AWS IA + No Code Program</strong>, Escola da Nuvem</sub>
</p>
