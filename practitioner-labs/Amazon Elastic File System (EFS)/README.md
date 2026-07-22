# Lab: Amazon Elastic File System (EFS)

## 📌 Overview

This lab demonstrates how to provision and manage **Amazon EFS** using the AWS Management Console. It covers setting up a shared, scalable file system and mounting it to an **Amazon EC2** instance.

## 🎯 Lab Objectives

* **Access:** Log in to the AWS Management Console.
* **Provision:** Create and configure an Amazon EFS file system.
* **Connect:** Access an Amazon Linux EC2 instance via SSH.
* **Mount:** Mount the EFS file system to the EC2 instance.
* **Monitor:** Examine and track file system performance metrics.

---

## 📖 Overview

This project documents the step-by-step process of provisioning an **Amazon Elastic File System (EFS)** and integrating it with **Amazon EC2** instances. The objective is to establish a secure, shared, and scalable network file system within an AWS VPC environment.

---

## 🛠 Lab Procedures

### Task 1: Security Group Configuration

*Goal: Allow inbound NFS traffic (Port 2049) to the EFS mount targets.*

1. Navigate to **EC2 > Security Groups**.
2. Copy the `EFSClient` Security Group ID.
3. Create a new security group named `EFS Mount Target`:
* **Description:** Inbound NFS access from EFS clients.
* **VPC:** `Lab VPC`.


4. **Inbound Rule:** Add a rule with Type `NFS` and set the **Source** to the `EFSClient` Security Group ID.

  ---<p align="center">
  <img src="./images/Screenshot 2026-07-18 013601.png" width="750">
</p>


### Task 2: Amazon EFS Provisioning

*Goal: Create the file system and assign the correct network security settings.*

1. Navigate to **EFS > Create file system > Customize**.
2. **Configuration:**
* Disable *Automatic Backups*.
* Set *Lifecycle management* to `None`.
* Add tag: `Key: Name` | `Value: My First EFS File System`.


3. **Network Access:**
* Select `Lab VPC`.
* For each Availability Zone: **Remove** the default security group and **Attach** the `EFS Mount Target` security group.


4. **Completion:** Review and **Create**. Wait until the File System and all Mount Targets state change to **Available**.

  ---<p align="center">
  <img src="./images/Screenshot 2026-07-18 023243.png" width="750">
</p>

### Task 3: Instance Connectivity

*Goal: Establish a remote session to the EC2 instance.*

1. Access the **AWS Details** menu in the top bar.
2. Copy the `InstanceSessionURL`.
3. Paste the URL into a new browser tab to launch the **AWS Systems Manager Session Manager**.

---

## 🚀 Post-Lab Verification

Once connected to the instance, you can verify connectivity by running:

```bash
# Install EFS utils (if not already present)
sudo yum install -y amazon-efs-utils

# Verify mount target accessibility
showmount -e <file-system-dns-name>

```

---

## 💡 Notes & Best Practices

* **Security:** Always use the Principle of Least Privilege when configuring Security Group sources.
* **Persistence:** For production environments, remember to add an entry in `/etc/fstab` to ensure the EFS volume mounts automatically after an instance reboot.
* **Monitoring:** Use CloudWatch to track `ClientConnections` and `BurstCreditBalance` to ensure optimal performance.


## Task 4: EFS Provisioning and Mounting

### 1. Environment Setup

Install the necessary EFS utilities on the EC2 instance:

```bash
sudo su -l ec2-user
sudo yum install -y amazon-efs-utils

```

### 2. Mounting the File System

Create the mount directory and mount the EFS file system using the NFSv4.1 protocol:

```bash
# Create directory
mkdir efs

# Mount (Replace with your specific EFS DNS name from the AWS Console)
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-xxxxxxx.efs.region.amazonaws.com:/ efs

```

### 3. Verification

Verify the mount status and available disk space:

```bash
df -hT

```
  ---<p align="center">
  <img src="./images/Screenshot 2026-07-18 025909.png" width="750">
</p>
---

## Task 5: Performance Benchmarking and Monitoring

### 1. Benchmarking with `fio`

We used the `fio` utility to simulate high-write throughput on the file system:

```bash
sudo fio --name=fio-efs --filesize=10G --filename=./efs/fio-efs-test.img --bs=1M --nrfiles=1 --direct=1 --sync=0 --rw=write --iodepth=200 --ioengine=libaio


```
<p align="center">
  <img src="./images/Screenshot 2026-07-18 030832.png" width="750">
</p>
---


### 2. Monitoring with Amazon CloudWatch

* **PermittedThroughput:** Observed the peak throughput (burst capability).
* **DataWriteIOBytes:** Analyzed total bytes written over the test duration to calculate actual write throughput (B/s) using the formula:

$$\text{Throughput} = \frac{\text{Sum of DataWriteIOBytes}}{60 \text{ seconds}}$$

<p align="center">
  <img src=".//images/Screenshot 2026-07-18 031524.png" width="750">
</p>
---

---

## Key Performance Insights

* **Burst Capability:** Amazon EFS automatically scales throughput as the file system size increases.
* **Baseline Performance:** Provides a consistent 50 MiB/s per TiB of storage.
* **Scaling:** Throughput scales linearly and automatically with storage growth, making it ideal for spiky, file-based workloads.

---


# AWS EFS Performance & Benchmarking Project

## Project Summary

This lab demonstrates the end-to-end process of managing high-performance file storage in the cloud using **Amazon Elastic File System (EFS)**. It covers provisioning, configuration, performance benchmarking, and real-time monitoring via CloudWatch.

## Workflow Overview

1. **Environment Setup**: Connected to an Amazon Linux EC2 instance.
2. **EFS Provisioning**: Created and configured an EFS file system.
3. **Mounting**: Configured the EC2 instance as an NFS client to mount the EFS using `NFSv4.1`.
4. **Performance Testing**: Executed synthetic I/O benchmarking using `fio`.
5. **Monitoring**: Analyzed throughput metrics and burst behaviors via Amazon CloudWatch.

## Technical Details

### Mounting the File System

Used the following command to mount the EFS target to the local directory:

```bash
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport [file-system-id].efs.[region].amazonaws.com:/ efs

```

### Benchmarking Strategy

The write performance was validated using `fio` with the following parameters:

* **Filesize**: 10GB
* **Block Size**: 1MB
* **IO Depth**: 200
* **Engine**: `libaio`

### Monitoring Metrics

* **PermittedThroughput**: Validated the burst capability of the EFS.
* **DataWriteIOBytes**: Calculated actual throughput by dividing the sum of bytes by time (1-minute intervals).

## Key Takeaways

* **Scalability**: EFS throughput scales automatically with file system growth.
* **Bursting**: Designed to handle spiky workloads, allowing temporary bursts beyond the baseline performance.
* **Efficiency**: Amazon EFS provides a highly available, POSIX-compliant shared file system perfect for multi-instance architectures.

---

*Lab Status: **Successfully Completed***

---

