# Cloud Lab 3 Submission — AWS Storage Options

- **Student name:** Davin Vatanak
- **Student ID:** e20220234
- **Learner Lab user name:** DAVIN_VATANAK
- **Date completed:** 12.11.26

---

## Part 2.1 — Amazon S3

### Bucket list

Created S3 bucket `static-web-bucket-444077418443` in `us-east-1`.

![S3 bucket list](images/01-s3-bucket-list.png)

### Uploaded files

Uploaded `index.html` and `error.html` to the bucket.

![Objects in the bucket](images/02-s3-objects.png)

### Static website

Enabled static website hosting with:

```text
Index document: index.html
Error document: error.html
```

The endpoint first returned `403 Forbidden` because disabling Block Public Access did not grant object permissions. After adding a bucket policy allowing public `s3:GetObject` access, the website loaded successfully.

![Static website in the browser](images/03-s3-website.png)

---

## Part 2.2 — Amazon EBS

### Volumes page

Created an 8 GiB gp3 volume named `Backup Volume` in `us-east-1a` and attached it to `instance-1`.

![Backup Volume in use](images/04-ebs-volumes.png)

### Attached volume (`lsblk`)

![lsblk output](images/05-ebs-lsblk.png)

The volume appeared as device `/dev/nvme1n1`. It was attached in the EC2 console as `/dev/sdf`.

Formatted and mounted it on `instance-1`:

```bash
sudo mkfs -t ext4 /dev/nvme1n1
sudo mkdir -p /mnt/my-ebs-volume
sudo mount /dev/nvme1n1 /mnt/my-ebs-volume
cd /mnt/my-ebs-volume
echo "This is a test file" | sudo tee testfile.txt
```

### File read on instance-2

Detached `Backup Volume` from `instance-1`, attached it to `instance-2`, and mounted it without formatting. The existing file remained available.

![testfile.txt on instance-2](images/06-ebs-second-instance.png)

---

## Part 2.3 — Amazon EFS

### File system and mount targets

Created encrypted Regional EFS file system `clab-efs` with mount targets in `us-east-1a` and `us-east-1b`.

![clab-efs and its mount targets](images/07-efs-file-system.png)

### File created on instance-1

Mounted EFS on `instance-1` and created `file_from_1.txt`:

```bash
cd ~/efsmount
echo "Creating a sample file from instance 1" > file_from_1.txt
cat file_from_1.txt
```

![Writing the file on instance-1](images/08-efs-write.png)

### File listed on instance-3

Mounted the same EFS file system on `instance-3` and read the shared file:

```bash
cd ~/efsmount
ls
cat file_from_1.txt
```

![Reading the file on instance-3](images/09-efs-read.png)

---

## Questions

### 1. After you turned on static website hosting, the endpoint first answered **403 Forbidden**. Why, and what changed after you added the bucket policy?

Static website hosting only enables the website endpoint. It does not grant public object permissions. The bucket policy allowed public `s3:GetObject` access, so the endpoint could read `index.html` and `error.html` after the policy was added.

### 2. `testfile.txt` was still there after you moved **Backup Volume** to `instance-2`. What would have happened to a file saved on `instance-1`'s **root** volume after terminating `instance-1`?

`testfile.txt` remained because it was stored on the separate EBS volume. A file saved on `instance-1`'s root volume would normally be deleted when the instance was terminated because the root volume uses the instance's delete-on-termination setting by default.

### 3. Why did you run `mkfs` on `instance-1` but not on `instance-2`?

The new volume had no file system, so `mkfs` was needed on `instance-1`. It was not run on `instance-2` because the volume was already formatted and contained data. Running `mkfs` again could destroy the existing file system and files.

### 4. Why could `instance-1` and `instance-3` use the EFS file system at the same time, when an EBS volume had to be detached first? Why did they have to be in the subnets of the mount targets?

EFS is network file storage designed for simultaneous access by multiple clients. EBS is a block device normally attached to one EC2 instance at a time. Each instance needed network access to an EFS mount target in its Availability Zone subnet so it could mount the file system through NFS.

### 5. For each need, choose S3, EBS or EFS and give one reason: (a) a website's images, (b) the disk of a database server, (c) a folder shared by several web servers.

- **(a) Website images — S3:** object storage works well for durable, HTTP-accessible images.
- **(b) Database server disk — EBS:** block storage provides low-latency storage attached to the database instance.
- **(c) Shared folder — EFS:** multiple web servers can mount and use the same file system simultaneously.

## Problems and how I solved them

The S3 website initially returned `403 Forbidden`. I checked the bucket policy and added public `s3:GetObject` permission for the bucket objects. The website then loaded through the S3 website endpoint.

## AI note

An AI tool suggested checking the S3 bucket policy after the `403 Forbidden` response. I checked the policy and verified that it allowed `s3:GetObject` for the bucket objects before retesting the endpoint.

## Clean up

- [x] Instances `instance-1`, `instance-2`, `instance-3` terminated.
- [x] `clab-efs` and **Backup Volume** deleted.
- [x] Security groups `efs-sg` and `ec2instances-sg` deleted.
- [x] S3 bucket emptied and deleted.
