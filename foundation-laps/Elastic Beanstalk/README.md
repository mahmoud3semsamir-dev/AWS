# Activity: AWS Elastic Beanstalk

## 📝 Activity Overview
This hands-on activity provides an automated AWS environment where an **AWS Elastic Beanstalk** application has been pre-created. Through this deployment exercise, you will upload and deploy code packages directly to the platform, then audit and observe the underlying AWS infrastructure resources that compose the Elastic Beanstalk ecosystem.

---

## ⏱️ Duration
* **Estimated Time:** Approximately **30 minutes** to complete.

---

## 🚫 AWS Service Restrictions
> ⚠️ **CRITICAL GOVERNANCE NOTE:** In this lab environment, access to AWS services and IAM actions is restricted to the specific resources needed to complete the lab instructions. 
> 
> You may encounter **Access Denied** or **Unauthorized** errors if you attempt to access other AWS services or perform actions beyond the explicit scope described in this guide.

## 🚀 Task 1: Access the Elastic Beanstalk Environment

In this task, you will navigate to the pre-configured AWS Elastic Beanstalk console, verify the platform health, and test initial domain connectivity before deploying your application code.

---

### 🗺️ Navigation & Health Verification

1. In the **AWS Management Console**, use the top search box (to the right of *Services*) to search for and select **Elastic Beanstalk**.
2. The **Environments** page will open, displaying a table with details of an existing Elastic Beanstalk application.
3. > ⏳ **Note on Initialization:** If the status in the **Health** column does not initially display **Ok**, the environment is still provisioning resources. Wait a few moments and refresh until it transitions to **Ok**.
4. Under the **Environment name** column, click on the name of your environment to open its central **Dashboard**.

---

### 🌐 Testing Environment Access

* Look at the dashboard and confirm that the current **Health** status of your application environment is reporting **Ok**. 
* *At this stage, the Elastic Beanstalk infrastructure is fully provisioned and ready to host an application, but it does not yet have any running application code.*

<p align="center">
  <img src="./images/Screenshot 2026-07-05 030204.png" width="750">
</p>
1. Near the top of the dashboard page, locate and click the **Domain link** (the URL ending in `.elasticbeanstalk.com`).
2. A new browser tab will open. 

> 🔍 **Expected Behavior:** You should see an **`HTTP Status 404 - Not Found`** error message. 

<p align="center">
  <img src="./images/Screenshot 2026-07-05 025927.png" width="750">
</p>
> 
> 💡 **Concept:** This behavior is completely expected because the underlying web server environment is live, but no source code or application package has been uploaded to serve traffic yet.

3. Close the test tab and return to the **Elastic Beanstalk console** to prepare for deployment.

---

### ✅ Success Criteria
* Successfully accessed the pre-created Elastic Beanstalk environment dashboard.
* Verified environment stability via the green **Health: Ok** status check.
* Confirmed edge routing by triggering the expected default 404 response through the platform URL.


## 📦 Task 2: Deploy a Sample Application to Elastic Beanstalk

In this task, you will download a sample application package, deploy it directly onto the Elastic Beanstalk platform, and audit the underlying infrastructure configurations and monitoring metrics.

---

### 💿 Application Deployment Steps

1. Download the pre-packaged sample application by clicking this link: [tomcat.zip](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/samples/tomcat.zip).

2. Return to your **Elastic Beanstalk Dashboard** and click the **Upload and Deploy** button.
3. Click **Choose File**, then navigate to and select the `tomcat.zip` file you just downloaded.
4. Click **Deploy**.

> ⏳ **Note:** It will take a minute or two for Elastic Beanstalk to orchestrate the infrastructure update, unpack the source code, and deploy the application.

---<p align="center">
  <img src="./images/Screenshot 2026-07-05 031628.png" width="750">
</p>

### ✅ Success Verification

1. Once the deployment status returns to green, click the **Domain URL link** at the top of the dashboard *(or simply refresh the previous browser tab that displayed the 404 error)*.
2. The live web application will now successfully resolve and display on your screen. 🎉

---<p align="center">
  <img src="./images/Screenshot 2026-07-05 031838.png" width="750">
</p>

---

### 🔍 Infrastructure & Configuration Audit

Elastic Beanstalk acts as a Platform as a Service (PaaS), automatically provisiong and managing the underlying infrastructure components for you:

#### 1. Compute & Scaling Configurations
* In the left navigation pane of the console, choose **Configuration**.
* Review the **Instance traffic and scaling** panel:
  * Observe the assigned **EC2 Security Groups**.
  * View the Auto Scaling configurations including the **minimum and maximum instances**.
  * Check the specific **instance type** details hosting your active web application.

#### 2. Database Integration Capabilities
* Review the **Networking, database, and tags** panel. 
  * *Note: Currently, no configuration details display here because the environment does not include a database resource.*
* Click **Edit** next to this row to observe how easily a relational database can be attached. If needed, you would only specify a few basic parameters and click Apply. *(Do not add a database for this activity; click **Cancel** at the bottom of the screen to return)*.

#### 3. Platform Monitoring & Health
* In the left panel under *Environment*, choose **Monitoring**.
* Browse through the native CloudWatch charts to analyze the available real-time performance indicators, target response codes, and system latency metrics.

---<p align="center">
  <img src="./images/Screenshot 2026-07-05 032307.png" width="750">
</p>

---

## 🔍 Task 3: Explore the AWS Resources That Support Your Application

AWS Elastic Beanstalk handles the provisioning and management of infrastructure automatically, but the underlying resources are still standard AWS components created within your account. In this task, you will audit the core compute and scaling resources supporting your application.

---

### 🗺️ Infrastructure Resource Audit

1. In the **AWS Management Console**, use the top search bar (to the right of *Services*) to search for and select **EC2**.
2. From the left navigation pane, choose **Instances**.
3. Observe the running resources: You will see **two active EC2 instances** provisioning infrastructure for your web application (both containing `samp` in their names).

---
---<p align="center">
  <img src="./images/Screenshot 2026-07-05 032641.png" width="750">
</p>

### 🏛️ Underlying Core Resources Created by Elastic Beanstalk

While Elastic Beanstalk abstracts infrastructure management, you retain full visibility and access to the following native AWS resources it generated:

* **EC2 Instances:** The virtual servers actively hosting your Tomcat web container.
* **AWS Security Group:** Automatically configured with port `80` (`HTTP`) open to accept inbound public web traffic.
* **Elastic Load Balancer (ELB):** Distributes incoming application traffic across both running EC2 instances to ensure high availability.
* **Auto Scaling Group:** Configured to dynamically scale the infrastructure from a **minimum of 2 instances up to a maximum of 6 instances**, depending on real-time network load and traffic demands.

---

# 🏁 Activity Complete! 🎉