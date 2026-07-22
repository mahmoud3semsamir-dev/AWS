# Lab 4: Working with Amazon EBS

## 📌 Overview
This lab focuses on **Amazon Elastic Block Store (Amazon EBS)**, a key underlying storage mechanism for Amazon EC2 instances. In this lab, you will learn how to create an Amazon EBS volume, attach it to an instance, apply a file system to the volume, and then take a snapshot backup.

  ---<p align="center">
  <img src="./images/lab-scenario.jpeg" width="750">
</p>
---

## 🎯 Objectives
By the end of this lab, you will be able to:
* [x] Create an Amazon EBS volume.
* [x] Attach and mount your volume to an EC2 instance.
* [x] Create a snapshot of your volume.
* [x] Create a new volume from your snapshot.
* [x] Attach and mount the new volume to your EC2 instance.

---

## ⚙️ Prerequisites & Details
* **Duration:** ~30 minutes
* **Skill Level:** Familiarity with basic **Amazon EC2** usage and **Linux server administration** (using command-line tools).
* **AWS Service Restrictions:** Access to AWS services in this lab environment might be restricted strictly to the required actions. Attempting to access other services may result in errors.

---

## 📖 Key Concepts

### What is Amazon Elastic Block Store (Amazon EBS)?
Amazon EBS offers **persistent, network-attached block storage** for Amazon EC2 instances that persists independently from the life of an instance. 

> 💡 **Key Advantage:** When used as a boot partition, Amazon EC2 instances can be stopped and restarted, enabling you to pay only for the storage resources used while maintaining your instance's state.

* **High Availability:** Automatically replicated on the backend within a single Availability Zone (AZ).
* **Durability & Backups:** Provides the ability to create point-in-time consistent **snapshots** stored in Amazon S3, which are automatically replicated across multiple AZs. These snapshots can act as a starting point for new volumes or be shared with other developers.

---

## 🚀 Amazon EBS Volume Features

| Feature | Description |
| :--- | :--- |
| **Persistent Storage** | Volume lifetime is independent of any particular Amazon EC2 instance. |
| **General Purpose** | Raw, unformatted block devices compatible with any operating system. |
| **High Performance** | Performance is equal to or better than local Amazon EC2 drives. |
| **High Reliability** | Built-in redundancy within a single Availability Zone. |
| **Resiliency by Design**| Annual Failure Rate (AFR) is exceptionally low (between 0.1% and 1%). |
| **Variable Size** | Flexible volume sizes ranging from **1 GB to 16 TB**. |
| **Easy Management** | Easily created, attached, backed up, restored, and deleted. |

---

## 🛠️ Step-by-Step Implementation Guide

### Task 1: Create a New EBS Volume

In this task, you will create a new Amazon EBS volume and prepare it to be attached to your running Amazon EC2 instance. 

> ⚠️ **Important:** Amazon EBS volumes can only be attached to EC2 instances that reside within the **same Availability Zone**. Make sure to verify your instance's zone before creating the volume.

#### Step 1: Verify the EC2 Instance Availability Zone
1. Open the **AWS Management Console**, click the **Services** menu, and select **EC2**.
2. In the left navigation pane, choose **Instances**.
3. Locate the pre-launched instance named **Lab**.
4. Check and **note down the Availability Zone** of this instance (e.g., `us-east-1a`).

#### Step 2: Configure and Create the EBS Volume
1. In the left navigation pane, choose **Volumes** under the *Elastic Block Store* section.
   * _Note: You will see an existing 8 GiB volume already attached to the Lab instance._
2. Click the **Create volume** button.
3. Configure the volume settings exactly as follows:
   * **Volume Type:** `General Purpose SSD (gp2)`
   * **Size (GiB):** `1` *(Note: Higher sizes may be restricted in this lab)*
   * **Availability Zone:** Select the **same availability zone** you noted from your EC2 instance.

#### Step 3: Add Tags and Finalize
1. Click **Add Tag** at the bottom of the configuration page.
2. In the Tag Editor, enter the following key-value pair:
   * **Key:** `Name`
   * **Value:** `My Volume`
3. Click **Create Volume**.
4. Your new volume will appear in the list. Wait for its state to change from `Creating` to `Available`. 
   * _💡 Hint: Click the **Refresh** icon if the status doesn't update automatically._

   ---<p align="center">
  <img src="./images/Screenshot 2026-07-10 054936.png" width="750">
</p>
---

### Task 2: Attach the Volume to an Instance

In this task, you will attach the newly created 1 GiB EBS volume to your running **Lab** EC2 instance.

1. In the **Volumes** list, select the checkbox next to **My Volume**.
2. Click on the **Actions** menu at the top right, then choose **Attach volume**.
3. Configure the following fields in the attachment window:
   * **Instance:** Click the field and select the pre-configured instance named **Lab**.
   * **Device Name:** Select `/dev/sdb` from the dropdown menu.
   
   > 📌 **Note:** Remember this device identifier (`/dev/sdb`), as you will need it to identify the disk inside the Linux operating system in the next tasks.

4. Click **Attach volume**.
5. Verify that the volume state has now changed from `Available` to **`In-use`**.

---<p align="center">
  <img src="./images/Screenshot 2026-07-10 060041.png" width="750">
</p>
---

### Task 3: Connect to Your Amazon EC2 Instance

In this task, you will securely connect to your Linux EC2 instance using **AWS Systems Manager Session Manager** directly from your browser.

#### Step 1: Establish the Connection
1. Navigate back to **Instances** in the left navigation pane of the EC2 Console.
2. Select the checkbox next to the **Lab** instance.
3. Click the **Connect** button at the top of the page.
4. Choose the **Session Manager** tab, then click **Connect**.
5. A new browser tab will open, displaying a fully functional terminal session connected to your instance.

#### Step 2: Switch to the Default User
Once the terminal loads, run the following command to switch from the default system user to the standard `ec2-user` home directory:

```bash
sudo su -l ec2-user
---<p align="center">
  <img src="./images/Screenshot 2026-07-10 060425.png[]" width="750">
</p>
---

# Task 4: Create and Configure Your File System

This guide walks you through formatting the newly attached EBS volume with an `ext3` file system, mounting it to `/mnt/data-store`, configuring persistent auto-mount on reboot, and verifying the setup.

---

## Step-by-Step Implementation

### Step 1: Check Current Storage
Run the following command to view the currently available storage on your instance:
```bash
df -h

```

**Expected Output:**
You should see output similar to the following, showing only the original 8GB disk volume. Your new volume is not yet shown:

```text
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        484M     0  484M   0% /dev
tmpfs           492M     0  492M   0% /dev/shm
tmpfs           492M  460K  491M   1% /run
tmpfs           492M     0  492M   0% /sys/fs/cgroup
/dev/xvda1      8.0G  1.5G  6.6G  19% /
tmpfs            99M     0   99M   0% /run/user/0
tmpfs            99M     0   99M   0% /run/user/1000

```

### Step 2: Format the New Volume

Create an `ext3` file system on the newly attached volume (`/dev/sdb`):

```bash
sudo mkfs -t ext3 /dev/sdb

```

### Step 3: Create Mount Point and Mount the Volume

Create a directory for mounting the new storage volume and mount it:

```bash
sudo mkdir /mnt/data-store
sudo mount /dev/sdb /mnt/data-store

```

### Step 4: Configure Auto-Mount on Reboot

To configure the Linux instance to mount this volume automatically whenever the instance is started, add the configuration line to `/etc/fstab`:

```bash
echo "/dev/sdb   /mnt/data-store ext3 defaults,noatime 1 2" | sudo tee -a /etc/fstab

```

Verify that the setting was added correctly to the last line of the file:

```bash
cat /etc/fstab

```

### Step 5: Verify the Mounted Volume

View the available storage again to confirm the volume is successfully mounted:

```bash
df -h

```

**Expected Output:**
The output will now contain an additional line for `/dev/xvdb` mapped to `/mnt/data-store`:

```text
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        484M     0  484M   0% /dev
tmpfs           492M     0  492M   0% /dev/shm
tmpfs           492M  460K  491M   1% /run
tmpfs           492M     0  492M   0% /sys/fs/cgroup
/dev/xvda1      8.0G  1.5G  6.6G  19% /
tmpfs            99M     0   99M   0% /run/user/0
tmpfs            99M     0   99M   0% /run/user/1000
/dev/xvdb       976M  1.3M  924M   1% /mnt/data-store

```

### Step 6: Write and Verify a Test File

On your mounted volume, create a file, add some text to it, and verify that the text has been successfully written:

```bash
sudo sh -c "echo some text has been written > /mnt/data-store/file.txt"
cat /mnt/data-store/file.txt

```

**Expected Output:**

```text
some text has been written

```

---

## Verification Checklist

| Step | Verification Action | Expected Result | Checked |
| --- | --- | --- | --- |
| 1 | Run `df -h` after mounting | `/dev/xvdb` appears mounted on `/mnt/data-store` | ☐ |
| 2 | Run `cat /etc/fstab` | `/dev/sdb` line exists at the end of the file | ☐ |
| 3 | Run `cat /mnt/data-store/file.txt` | Outputs: `some text has been written` | ☐ |


# AWS Labs: EBS Snapshots and Restoration

This guide covers the process of creating a point-in-time snapshot of an Amazon EBS volume, simulating data loss by deleting a file, and then restoring that data using a new EBS volume created from the snapshot.



## Task 5: Create an Amazon EBS Snapshot

Amazon EBS snapshots provide a durable, point-in-time backup stored in Amazon S3. Only used storage blocks are copied, meaning empty blocks do not occupy snapshot storage space.

### Step 1: Create the Snapshot via AWS Console
1. Open the **AWS Management Console**.
2. Navigate to **Volumes** and select **My Volume**.
3. From the **Actions** menu, select **Create snapshot**.
4. Click **Add tag** and configure the following parameters:
   * **Key:** `Name`
   * **Value:** `My Snapshot`
5. Click **Create snapshot**.

### Step 2: Verify Snapshot Creation
1. In the left navigation pane of the EC2 Dashboard, click on **Snapshots**.
2. Locate your snapshot. The initial status will display **Pending** while the snapshot is being processed.
3. Wait until the status transitions to **Completed**.

### Step 3: Simulate Data Loss in Terminal
To test the backup integrity, delete the test file you created previously on your mounted volume:

```bash
sudo rm /mnt/data-store/file.txt

```

Verify that the file has been successfully deleted:

```bash
ls /mnt/data-store/

```

*(The output should be empty, confirming the file is gone).*

---

## Task 6: Restore the Amazon EBS Snapshot

If you ever need to retrieve data stored in a snapshot, you can restore it to a completely new EBS volume.

### Step 1: Create a Volume from Your Snapshot

1. In the **AWS Management Console**, navigate to **Snapshots** and select **My Snapshot**.
2. Open the **Actions** menu and select **Create volume from snapshot**.
3. **Availability Zone:** Select the **same Availability Zone** where your EC2 instance is running.
4. Click **Add tag** and configure:
* **Key:** `Name`
* **Value:** `Restored Volume`


5. Click **Create volume**.

---<p align="center">
  <img src=".//images/Screenshot 2026-07-10 062113.png" width="750">
</p>
---

### Step 2: Attach the Restored Volume to Your EC2 Instance

1. In the left navigation pane, choose **Volumes**.
2. Select **Restored Volume**.
3. In the **Actions** menu, select **Attach volume**.
4. Click the **Instance** field and select your running (Lab) instance from the list.
5. For the **Device Name**, select `/dev/sdc` (or the next available device identifier) from the dropdown.
6. Click **Attach volume**. The volume state will change to **in-use**.

### Step 3: Mount the Restored Volume

Switch back to your terminal session to create a new mount point and mount the newly attached volume.

1. **Create a directory for the restored volume:**
```bash
sudo mkdir /mnt/data-store2

```


2. **Mount the restored volume:**
```bash
sudo mount /dev/sdc /mnt/data-store2

```



### Step 4: Verify Restored Data

List the contents of the new mount directory to ensure that the file deleted in Task 5 has been recovered successfully:

```bash
ls /mnt/data-store2/

```

**Expected Output:**

```text
file.txt

```

---

## Verification Checklist

| Task | Action | Expected Result | Checked |
| --- | --- | --- | --- |
| Task 5 | Check Snapshot Status | Status changes from `Pending` to `Completed` | ☐ |
| Task 5 | Run `ls /mnt/data-store/` | Output is empty (File successfully simulated deleted) | ☐ |
| Task 6 | Run `ls /mnt/data-store2/` | Outputs: `file.txt` (Data successfully recovered) | ☐ |

```

```
---

## Conclusion

🎉 **Congratulations!** You have successfully completed the lab and achieved the following objectives:

* **Created an Amazon EBS volume** from scratch.
* **Attached the volume** securely to an EC2 instance.
* **Created an ext3 file system** on the new volume and configured persistent auto-mounting via `/etc/fstab`.
* **Added a test file** to verify read/write capabilities on the mounted directory.
* **Created a point-in-time snapshot** of your volume to back up critical data.
* **Created a new volume from the snapshot** to practice infrastructure recovery.
* **Attached and mounted the new volume** to a separate mount point (`/mnt/data-store2`).
* **Verified data integrity** by confirming the deleted file was successfully restored on the new volume.

### 🏁 Lab Complete!
Your storage architecture configuration, backup system, and disaster recovery simulation are now fully operational and verified.