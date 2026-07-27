<h1 align="center">🗄️ Storage Management with Amazon EBS, IAM Role, and Amazon S3</h1>

<p align="center">
  <img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/5cb29c55-5033-4af9-8737-092b2a21b8b9" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/Amazon%20EBS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon EBS">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/status-completed-brightgreen?style=for-the-badge" alt="Status">
</p>

<p align="center">
  Hands-on project for automated backup of EBS volumes, with access control based on an IAM Role and file recovery through Amazon S3 versioning.
</p>

---

## 📚 About this Project

This repository documents the creation of an **automated disk data backup routine (Amazon EBS)**, including cleanup of old snapshots and file synchronization with Amazon S3 as an extra layer of protection.

The project also addresses a core cloud security topic: how to grant an EC2 instance access to other AWS services **without using fixed credentials**, through an IAM Role.

---

## 1. Context and Problem

This lab was carried out during the **Re/Start AWS AI + No Code Program**, offered by **Escola da Nuvem**, as part of my learning journey in Cloud Computing and AWS Infrastructure.

The scenario starts from a very common problem in production environments: **how do you protect data stored on a server against accidental loss, without relying on manual backup processes that someone might forget to run?**

There's also a second, security-related problem: when an EC2 instance needs to access other AWS services (such as S3), what's the correct way to grant that permission? Embedding fixed access keys inside the instance is a risky practice — if those credentials leak, the damage can be far greater than expected.

This project solves both problems at once: it creates an automated snapshot routine for the EBS volume, and uses an IAM Role to give the instance access to S3 following security best practices.

---

## 2. Objective

Implement a storage management routine on AWS, covering:

- Automated creation and maintenance of **EBS volume snapshots**;
- Granting permissions to an EC2 instance via an **IAM Role**, instead of fixed credentials;
- File synchronization between an EBS volume and an **S3 bucket**;
- Using **S3 versioning** as a recovery mechanism for accidentally deleted files.

---

## 🔐 Principle of Least Privilege

The **Principle of Least Privilege** states that any user, application, or resource should only have access to what is strictly necessary to perform its function — nothing more.

**Why this principle matters:**

- **Reduces the attack surface**: if a credential or resource is compromised, the potential damage is limited to the scope of permissions it actually has.
- **Avoids unnecessary coupling between services**: an instance that only needs to access S3 shouldn't have permission to, for example, delete databases or manage users.
- **Makes auditing easier**: specific permissions make it clear why each access exists, instead of relying on "full access just to be safe."

**How this principle was applied in this lab:**

Instead of storing an AWS access key directly on the "Processor" instance (which would require manually managing, rotating, and protecting that credential), the lab uses an **IAM Role** (`S3BucketAccess`) attached as an *instance profile* to the instance. This Role grants exactly the permissions the instance needs to interact with EBS volumes and the lab's S3 bucket, and nothing more.

This approach is considered an AWS security best practice: **Roles eliminate the need for fixed credentials on instances**, since AWS automatically manages the rotation of the temporary credentials behind the Role.

---

## 3. Solution Architecture

<p align="center">
  <img width="778" height="486" alt="Image" src="https://github.com/user-attachments/assets/5cfd5dca-99bb-40a7-b0b2-e3178fdcd60c" />
</p>

**Solution flow:**

```
Command Host  ──(administers via AWS CLI)──▶  Processor (IAM Role: S3BucketAccess)
                                                    │
                                                    ▼
                                             EBS Volume
                                              │        │
                                              ▼        ▼
                                         Snapshot   S3 sync ──▶ S3 Bucket (versioned)
```

The "Command Host" instance is used to administer the environment via AWS CLI. The "Processor" instance has the `S3BucketAccess` Role attached, which allows it (and the commands run on it) to access the EBS volume and the S3 bucket without needing fixed credentials. From there, the flow splits into two data-protection fronts: EBS volume snapshots and file synchronization with S3, the latter with versioning enabled.

---

## 4. What Was Done

Summary of the main steps carried out in this lab:

1. **Creation of an S3 bucket**, which served as the destination for syncing files from the EBS volume.
2. **Attaching the `S3BucketAccess` IAM Role** as an instance profile on the "Processor" EC2 instance, granting it permission to interact with EBS and S3.
3. **Connecting to the "Command Host" instance** via EC2 Instance Connect, used to administer the environment.
4. **Identifying the EBS volume and the "Processor" instance ID** via queries with `aws ec2 describe-instances`.
5. **Stopping the "Processor" instance** before creating the initial snapshot, to ensure data consistency.
6. **Creating the first EBS volume snapshot** (`aws ec2 create-snapshot`) and verifying the process completed successfully.
7. **Restarting the "Processor" instance** after the snapshot finished.
8. **Creating a cron job** to automatically generate a new snapshot every minute, simulating a recurring backup routine.
9. **Running the Python script `snapshotter_v2.py`**, which identifies all snapshots of the volume and keeps only the two most recent ones, automatically removing the rest.
10. **Optional challenge: syncing with Amazon S3**, including:
    - Enabling versioning on the S3 bucket (`aws s3api put-bucket-versioning`);
    - Syncing a local folder with the bucket via `aws s3 sync`;
    - Deleting a local file and propagating that deletion to S3 using `aws s3 sync --delete`;
    - Querying the version history of the deleted file (`aws s3api list-object-versions`);
    - Recovering the previous version of the file (`aws s3api get-object --version-id`) and syncing again to restore it in the bucket.

---

## 5. Tools and Services Used

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/Amazon%20EBS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon EBS">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/AWS%20CLI-232F3E?style=for-the-badge&logo=amazonwebservices&logoColor=white" alt="AWS CLI">
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/Cron-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" alt="Cron">
</p>

<br>

| Tool / Service | Purpose in the Project |
|---|---|
| **Amazon EC2** | "Command Host" (administration) and "Processor" (protected resource) instances |
| **Amazon EBS** | Disk storage volume that received the snapshots |
| **Amazon S3** | Destination for file synchronization and extra backup layer with versioning |
| **AWS IAM (Role)** | Granting permissions to the "Processor" instance without fixed credentials |
| **AWS CLI** | Running all snapshot, sync, and configuration commands |
| **Python** | Script for automatically retaining the two most recent snapshots |
| **Cron** | Scheduling recurring snapshot creation |

---

## 6. Key Challenges and Lessons Learned

**Role instead of a fixed access key**
Before this lab, I associated "granting access to an instance" only with the idea of setting up `aws configure` with an access key. Watching the `S3BucketAccess` Role get attached directly as an instance profile, with no credential visible or manually copied, changed how I think about how instances should access other AWS services.

**Why stop the instance before the snapshot**
I noticed the lab stops the "Processor" instance before taking the first snapshot. This reinforced an important data consistency concept: taking a snapshot while the instance is running can capture the disk in an intermediate state (with pending cached writes), while stopping the instance ensures everything that needs to be written to the volume actually is.

**Retention automation with the Python script**
Running `snapshotter_v2.py` and seeing the logic that keeps only the two most recent snapshots made me realize a real operational problem: automated snapshots without a retention policy generate indefinitely growing costs. Automating isn't just about creating backups — it's also about managing their lifecycle.

**Versioning as a real safety net, tested in practice**
Rather than just enabling versioning and moving on, this lab had me actually delete a file, confirm it disappeared from the bucket, and then recover it using the version history. That made it much clearer, in practice, why versioning is considered a reliability requirement and not just an optional setting.

---

## 7. Final Result

By the end of the project, the environment had:

- ✅ An automated snapshot routine for the EBS volume, with controlled retention of the two most recent snapshots;
- ✅ An EC2 instance accessing S3 through an IAM Role, without using fixed credentials;
- ✅ File synchronization with Amazon S3, including active versioning and successful recovery of a deleted file.

<!--
  📌 PLACEHOLDER: AWS CONSOLE SCREENSHOTS
  Include screenshots here of the AWS console showing:
  - The S3BucketAccess Role attached to the Processor instance
  - The list of snapshots before and after running the Python script
  - Versioning enabled on the S3 bucket
  - The version history of the recovered file
-->

---

## 8. Skills Developed

<p align="center">
  <img src="https://img.shields.io/badge/Cloud%20Computing-232F3E?style=flat-square&logo=amazonaws&logoColor=white" alt="Cloud Computing">
  <img src="https://img.shields.io/badge/Amazon%20EBS-FF9900?style=flat-square&logo=amazonaws&logoColor=white" alt="Amazon EBS">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=flat-square&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=flat-square&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/AWS%20CLI-232F3E?style=flat-square&logo=amazonwebservices&logoColor=white" alt="AWS CLI">
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white" alt="Python">
</p>

- ☁️ Cloud Computing (AWS)
- 💾 Amazon EBS (snapshots, backup, and restoration)
- 🔐 AWS IAM (Roles and the Principle of Least Privilege applied to compute resources)
- 🪣 Amazon S3 (file synchronization and versioning)
- 💻 AWS CLI and automation via Python script and cron
- 🛠️ Backup lifecycle management

---

<p align="center">
  <sub>Project developed as part of the <strong>Re/Start AWS AI + No Code Program</strong>, Escola da Nuvem</sub>
</p>
