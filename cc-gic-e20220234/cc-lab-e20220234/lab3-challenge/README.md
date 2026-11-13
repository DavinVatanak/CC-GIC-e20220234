# Cloud Lab 3 Challenge Submission — Secure S3 Bucket

- **Student name:** Davin Vatanak
- **Student ID:** e20220234
- **Learner Lab user name:** DAVIN_VATANAK
- **Bucket name:** `clab-datastore-e20220234-2026`
- **Date completed:** 12.11.26

---

## 1. Bucket

![Bucket list](images/01-bucket.png)

## 2. Object ownership

![Object Ownership](images/02-object-ownership.png)

The bucket uses **Bucket owner preferred** with **ACLs enabled**.

## 3. Public access policy

![Bucket policy](images/03a-bucket-policy.png)

![Public object loaded in the browser](images/03b-public-object.png)

![Private object denied](images/03c-private-object.png)

My policy is in [bucket-policy.json](bucket-policy.json).

The `Resource` line limits public access to objects in the `public/` folder:

```json
"Resource": "arn:aws:s3:::clab-datastore-e20220234-2026/public/*"
```

The `public/*` suffix matches objects inside `public/` only. The policy grants only `s3:GetObject`; it does not grant bucket listing or access to root-level objects and other prefixes.

## 4. Versioning

![Versioning enabled](images/04-versioning.png)

## 5. Lifecycle rule

![Lifecycle rule](images/05-lifecycle.png)

The lifecycle rule applies to the entire bucket and moves noncurrent object versions to **S3 Standard-IA** after **30 days**.

---

## Questions

### 1. You turned off parts of **Block Public Access** for this bucket. Why was that needed, and why is it still safe for the objects outside `public/`?

It was needed so the bucket policy could grant anonymous `s3:GetObject` access to the public content. It remains safe because the policy resource is restricted to `public/*`. Objects at the bucket root and under other prefixes do not match the policy, so their object URLs return `AccessDenied`. The policy also grants no `s3:ListBucket` permission.

### 2. What is a **noncurrent version**? Give an example from your own test.

A noncurrent version is an older version of an object kept by S3 after a newer version is uploaded using the same object key. For example, after replacing `public/index.html`, the earlier `public/index.html` becomes noncurrent while the newly uploaded page becomes the current version. Versioning keeps both versions.

### 3. Why move noncurrent versions to S3 Standard-IA instead of deleting them?

Older versions may still be needed to restore previous content or investigate an accidental change. S3 Standard-IA reduces storage cost while retaining the versions. Deletion would save more storage cost but permanently remove recovery copies.

## AI note

An AI tool suggested checking that the lifecycle details showed **Entire bucket**, **Day 30**, and **Standard-IA**, because the lifecycle table truncated the transition text. I opened the lifecycle rule details and confirmed all three settings in `05-lifecycle.png`.

## Clean up

- [x] All object versions deleted and the bucket deleted.
