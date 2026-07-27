<h1 align="center">🪣 Static Website Hosting with Amazon S3</h1>

<p align="center">
  <img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/8b56fd35-b710-4ca3-8291-f9f926fb78e0" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/AWS-S3-orange?logo=amazons3" alt="AWS S3">
  <img src="https://img.shields.io/badge/AWS-IAM-orange?logo=amazoniam" alt="AWS IAM">
  <img src="https://img.shields.io/badge/AWS-CLI-orange?logo=amazonaws" alt="AWS CLI">
  <img src="https://img.shields.io/badge/status-completed-brightgreen" alt="Status">
</p>

<p align="center">
  Hands-on project built to simulate hosting a simple, cheap, and secure institutional website using Amazon S3 as the core infrastructure.
</p>

---

## 📚 About this Project

This repository documents a practical **static website hosting lab on AWS**, focused on three pillars that matter to any Cloud/Infrastructure professional: **cost**, **availability**, and **access security**.

Rather than just "uploading a site to S3," the goal here was to understand **why** each configuration was made the way it was, and to document it in a way that anyone — technical or not — can understand the problem being solved.

---

## 1. Context and Problem

This lab was carried out during the **Re/Start AWS AI + No Code Program**, offered by **Escola da Nuvem**, as part of my learning journey in Cloud Computing and AWS Infrastructure.

The proposed scenario starts from a very common problem in the day-to-day of small businesses and tech teams: **how to host a simple website (institutional, promotional, or a landing page) cheaply, quickly, and without having to manage an entire server?**

Keeping a traditional web server (EC2, for example) running 24/7 just to serve a static site (HTML, CSS, images) is a waste of resources and money: you pay for computing capacity that, in practice, is barely used. On top of that, there's extra maintenance effort involved — OS updates, security patches, availability monitoring — for something that requires no processing at all, just delivering static files.

That's exactly the problem this project set out to solve.

---

## 2. Objective

Implement a **static website hosting solution using Amazon S3**, applying fundamental Cloud Computing concepts such as:

- Scalable, low-cost object storage;
- Conscious and secure public access control;
- Using the AWS CLI and a dedicated IAM user to manage resources following security best practices;
- Automating repetitive tasks via script (deploying/updating the site).

Beyond the technical aspect, this project also aimed to develop my ability to **document architecture decisions** — not just the "how I did it," but the "why I did it this way."

---

## 3. Solution Architecture

<p align="center">
  <img width="788" height="312" alt="Image" src="https://github.com/user-attachments/assets/33d02130-0f24-4db8-8c71-e273eb2da758" />
</p>

**Solution flow:**

```
User (Browser) → S3 Bucket Website Endpoint → Amazon S3 (Static Website Hosting)
                                                              ↓
                                              Public Access Policies + ACL
```

The S3 bucket was configured to act as a static file server, with `index.html` set as the main document. Public access was granted **only to the site's objects**, not to the bucket as a whole. This is an important distinction that I cover in more detail in the lessons-learned section.

---

## 4. What Was Done

Summary of the main steps carried out in this lab:

1. **Secure connection via AWS Systems Manager (Session Manager)** to an EC2 instance, without needing to open SSH ports or manage keys.
2. **AWS CLI configuration** on the instance, using temporary credentials provided by the lab environment.
3. **Bucket creation via the command line** (`aws s3api create-bucket`), with a unique name and an explicitly defined region.
4. **Creation of a dedicated IAM user** with full access permissions to Amazon S3, reinforcing the practice of not using the root/administrative account for day-to-day operations.
5. **Adjusting bucket permissions**, unblocking public access in a controlled way and enabling ACLs for the objects that actually needed to be public (the site's files).
6. **Extracting the site files** (a package containing `index.html`, a `css` folder, and an images folder) directly on the EC2 instance.
7. **Uploading the site files** to the bucket, with public read permission set via ACL.
8. **Enabling static website hosting** on the bucket (`aws s3 website`), setting `index.html` as the index document.
9. **Creating an update script (`update-website.sh`)** to make the process of publishing site changes repeatable and automated, instead of manually repeating commands with every change.

---

## 5. Tools and Services Used

#### ☁️ AWS Services

<p align="center">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=for-the-badge&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="Amazon EC2">
  <img src="https://img.shields.io/badge/Systems%20Manager-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white" alt="AWS Systems Manager">
  <img src="https://img.shields.io/badge/AWS%20CLI-232F3E?style=for-the-badge&logo=amazonwebservices&logoColor=white" alt="AWS CLI">
</p>

#### 💻 Languages & Scripting

<p align="center">
  <img src="https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" alt="Bash">
  <img src="https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white" alt="HTML5">
  <img src="https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white" alt="CSS3">
</p>

<br>

| Tool / Service | Purpose in the Project |
|---|---|
| **Amazon S3** | Storage and hosting of the static website |
| **AWS IAM** | Creation of a dedicated user and permission control |
| **AWS Systems Manager (Session Manager)** | Secure access to the EC2 instance, without exposing SSH ports |
| **AWS CLI** | Automating resource creation and site deployment |
| **Amazon EC2** | Instance used as the working environment to run the commands |
| **Bash Script** | Automating the site update process |
| **HTML / CSS** | Content of the hosted static website |

---

## 6. Key Challenges and Lessons Learned

**Blocking public access vs. actually making objects public**
My first obstacle was understanding that "enabling public access" in S3 is not an all-or-nothing decision. By default, AWS blocks any attempt at public bucket exposure, which is the correct protection. The real lesson here was understanding the difference between unblocking public access **at the bucket level** (a necessary but not sufficient configuration) and granting public read access **on specific objects** via ACL. This distinction avoids the classic mistake of exposing an entire bucket when only a few files actually needed to be public.

**Versioning as a safety net, not just an "extra feature"**
By enabling versioning, I understood in practice why it's considered a reliability requirement rather than just an optional item: any human error, like overwriting the wrong `index.html`, stops being an irreversible problem. This completely changes how you think about maintaining even a "simple" website.

**Automating to reduce human error**
Creating the `update-website.sh` script seemed, at first, like a trivial step. But it turned out to be one of the most valuable lessons of the lab: realizing that any manual process repeated more than once is a candidate for automation, which reduces human error and saves time on future site updates.

**Dedicated IAM user instead of the main account**
Creating a specific user to operate S3, instead of using the environment's main credentials, reinforced in practice a security principle I already knew in theory: **never use administrator credentials for day-to-day operational tasks**, even in a lab environment.

---

## 7. Final Result

At the end of the project, the static website was made publicly available through the S3 bucket's website endpoint, following the pattern:

```
http://<bucket-name>.s3-website-<region>.amazonaws.com
```

The solution met the three core requirements of the mission:
- ✅ S3 bucket configured for static website hosting (`index.html`);
- ✅ Versioning enabled for protection against data loss;
- ✅ Public access policies correctly configured (access granted only where necessary).

<!--
  📌 PLACEHOLDER: AWS CONSOLE SCREENSHOTS
  Include screenshots here of the AWS console showing:
  - The created bucket
  - The static website hosting settings enabled
  - Versioning enabled
  - The site working in the browser
-->

---

## 8. Skills Developed

<p align="center">
  <img src="https://img.shields.io/badge/Cloud%20Computing-232F3E?style=flat-square&logo=amazonaws&logoColor=white" alt="Cloud Computing">
  <img src="https://img.shields.io/badge/Amazon%20S3-569A31?style=flat-square&logo=amazons3&logoColor=white" alt="Amazon S3">
  <img src="https://img.shields.io/badge/AWS%20IAM-DD344C?style=flat-square&logo=amazoniam&logoColor=white" alt="AWS IAM">
  <img src="https://img.shields.io/badge/AWS%20CLI-232F3E?style=flat-square&logo=amazonwebservices&logoColor=white" alt="AWS CLI">
  <img src="https://img.shields.io/badge/Troubleshooting-4B5563?style=flat-square&logo=probot&logoColor=white" alt="Troubleshooting">
  <img src="https://img.shields.io/badge/Technical%20Documentation-4B5563?style=flat-square&logo=readthedocs&logoColor=white" alt="Technical Documentation">
</p>

- ☁️ Cloud Computing (AWS)
- 🪣 Amazon S3 (Static Website Hosting, ACLs, Versioning)
- 🔐 AWS IAM (user creation, managed policies)
- 💻 AWS CLI and automation via Bash
- 🛠️ Troubleshooting permissions and public access
- 📝 Business-oriented technical documentation

---

<p align="center">
  <sub>Project developed as part of the <strong>Re/Start AWS AI + No Code Program</strong>, Escola da Nuvem</sub>
</p>
