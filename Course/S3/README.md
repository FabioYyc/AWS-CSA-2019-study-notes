# CHAPTER 3 | S3 Object Storage and CDN

## S3
### Read S3 FAQ before exams

### [What's S3](https://aws.amazon.com/s3/)

* Safe place to store files
* Object-based storage: you can save only object (flat files, like jpg, txt etc.), you can't, for example, install an OS (In this case you need block-based storage).
* Files can save from 0 Bytes to 5 TB.
* No storage Limits.
* Files are stored in Buckets (a folder in a cloud).
* S3 bucket is a universal namespace, the name must be unique globally. So you **cannot** have the same name as someone else.
* Sample of an S3 URL: ```https://s3-eu-west-1.amazonaws.com/yourbucket```.
* When you upload (Post) an object in S3 you get an HTTP 200 OK code back.

### Data Consistency for S3
* Read after write consistency for PUTs: if a file is uploaded to S3, you can read it immeidately 
* Eventual consistency: if you uploaded a new version, you may read version 1 before overwriting or deletes propagated
* It's consistent in reads after a write on new objects.
* It's eventually consistent for overwriting and deletes (this means it can take some time to propagate)
* S3 is spread across multiple AZ's

### Components

* S3 is an object. Objects consist of:
  * Key (name of the object)
  * Value (data in bytes)
  * Version ID (Used on versioning, you can multiple version of the file)
  * Metadata (a set of data that describes and gives information about the object data.)
  * Subresources:
    * Access Control List (Decide who can access files)
    * Torrent (Not an exam topic)

### Basics

* S3 SLA: 99.9% availability
* S3 is built for 99.99%
* S3 guarantees 11x9s (99.999999999) durability for S3 information.
* Tiered Storage (classes) available
* You can have lifecycle management: files can be move around tiers based on their age
* Versioning
* Supports [multi-part upload](https://docs.aws.amazon.com/AmazonS3/latest/dev/mpuoverview.html)
* Encryption
* Access control (permissions on single files) and bucket policies (permissions on buckets)
* Multi-factor authentication to delete the object

### [S3 Storage Tiers](https://aws.amazon.com/s3/storage-classes/) -> Important for exams

* S3 standard: 99.99% availability 11x9s durability (it sustains the loss of 2 facilities concurrently), stored redundant across facilities
* S3 IA: (Infrequently Accessed): For data that is accessed less frequently, but needs rapid access. You are charged a retrieval fee per GB retrieved
* S3 One Zone IA: Like S3 IA but data is stored only in one AZ, low cost option of IA.
* S3 Intelligent Tiering: uses machine learning to optimize the cost by moving data across tiers
* Glacier: Most cheap, used for archival only.
  * Expedited: few minutes for retrieval
  * Standard: 3-5 hours for retrieval
  * Bulk: 5-12 hours for retrieval
  * It encrypts data by default
  * Regionally availability
  * Designed with 11x9s durability, like S3
* Glacier deep archive: retrival time 12 hours

### [Charges](https://aws.amazon.com/s3/pricing/)

S3 is charged for:

* Storage
* Requests
* Storage management pricing
* Data Transfer Pricing
* Transfer acceleration (it's using CloudFront the AWS CDN) using edge locations
* Cross region replication

### Creating bucket note
* If the object is not made public, the url will return access denied (An xml file)
* You can change the permission to make the object public
* We can change the tiering in object level and bucket level

### [Server side Encryption and ACL](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingServerSideEncryption.html)

* AES-256 Use Server-Side Encryption with Amazon S3-Managed Keys (SSE-S3)
* AWS-KMS Use Server-Side Encryption with AWS KMS-Managed Keys (SSE-KMS)
* Server-Side Encryption with Customer-Provided Keys (SSE-C)

* Control access to the bucket using bucket ACL or Bucket policies
* By default all buckets and objects within are private 

### [S3 Version Control](https://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html)

* Once you enable versioning in the bucket, you can't disable it, you can only suspend it. A way of disabling it is to delete the bucket and re-create it
* Every time you update an object, it will become private by default, you will need to make it public again in action options.
* If you make older version public, and you uploaded a new version, new version cannot be access before you make it public, but you can access the content of older version
* It integrates with Lifecycle rules: you can use prefix or tag to apply lifecycle rule, the verision which lifecycle rules will be applied on can be specify as well.
* You can set expire day of your files in lifecycle setting
* You pay for each version you have
* Size of the file will be sum of different versions -> You might need lifecycle policies to retire versions quickly
* Delete an object:


    Once you delete a file inside a versioned bucket, you don't delete the file, you simply add a Delete Marker (this basically creates a new version of the object)
    
    If you delete the version with the Detele Marker you will basically restore the object.

    If you want to permanently delete the object, you have to delete all the Versions of the object.

    You can optionally add another layer of security by configuring a bucket to enable MFA Delete

    More info about versioning:
    [ObjectVersioning](https://docs.aws.amazon.com/AmazonS3/latest/dev/ObjectVersioning.html)
    and
    [Versioning](https://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html)

### [S3 Cross region replication](https://docs.aws.amazon.com/AmazonS3/latest/dev/crr.html)

* Regions must be unique
* Versioning is required to be enabled for cross region replication
* If you uploaded a new file and turned on the service, the new file will be replicated to a new region
* Cross region replication doesn't replicate existing object by default, only new ones (after the replicate is set) will be replicated automatically.
* New version of file will be automatically replicate to the CRR bucket
* In order to replicate the existing objects, you need to do a `cp` using the aws cli:

    `aws s3 cp --recursive s3://alessio-casco-versioning s3://alessio-casco-versioning-replica-sydney`
* If you delete an object in the primary bucket, the delete action and markers won't be done or replicated in your remote bucket, this is a security function, so the accidental delete will not spread between buckets
(delete delete marker will restore the object)
* deleting individual version or delete marker will also not be replicated
Only creations and modifications are replicated to the bucket in the other regions NOT the delete
* You can't replicate over multiple buckets, the maps are always 1-to-1

## [CloudFront](https://aws.amazon.com/cloudfront/)

### [What's a CDN](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/)

* A content delivery network or content distribution network is a geographically distributed network of proxy servers and their data centres

* Key terminology about CloudFront:
  * Edge Location: Is the location where the content is cached (separate from AWS AZ's or regions)
    Be aware that you can also write on edge locations, is not ready only.
  * Invalidating (erasing) the cache costs money.
  * Origin: Is the source of the files the CDN will distribute. An origin can be an EC2 instance, an S3 bucket, an Elastic Load Balancer or Route53, you can also have your own origin, it not mandatory that is within AWS.
  * Distribution: Is the name AWS calls CDN's.
    * You can Have two types: Web that is for generic web contents and RTMP that is for video streaming
    * TTL: time to live of the cached object.

### [S3 Security & Encryption](https://aws.amazon.com/blogs/aws/new-amazon-s3-encryption-security-features/)

* You can configure S3 to create access logs for requests made to the S3 bucket
* Access control for buckets:
  * Bucket policies: Permission bucket wide
  * Access control list: Permissions that can be applied to the single object
  * can configure access log -> logs all requests to the bucket

* Encryption:
  * In transit: from to your bucket, HTTPS for example (traffic is encrypted in transit), achieved by ssl or t;s
  * At rest: encrypt data been stored
    * Server-side encryption:
      * S3 Managed Keys: SSE-S3 (Keys are managed by S3), automatically managed the keys
      * Key Management Service: SS3-KMS the customer manages the keys, you and amazon managed the key together
      * Server-side encryption: Here you manage the keys, and Amazon manage the writes, customer provided key
  * Client-side Encryption: You encrypt the data and you upload it encrypted to S3

## [Amazon Storage](https://aws.amazon.com/products/storage/)

### [Amazon Storage Gateway](https://aws.amazon.com/storagegateway/)

What's an Amazon Storage Gateway: AWS Storage Gateway connects an on-premises software appliance with cloud-based storage to provide seamless integration with data security features between your on-premises IT environment and the AWS storage infrastructure.

* File Gateway: For flat files, stored directly in S3. You can NFS Mount points
* VOlume gateway (iSCSI): Block-based storage
  * Store volume (you keep all your data on prem)
  * Cached Volumes (you keep only the most recent data on prem)
Tape Gateway (VTL): Virtual tapes

### [Snowball](https://aws.amazon.com/snowball/)

Import Export is still available and was the first version of snowball, you used to ship your drives to AWS

Snowball is (an appliance) a petabyte-scale data transport solution that uses devices designed to be secure to transfer large amounts of data into and out of the AWS Cloud

Snowball edge: is a 100TB data transfer device with onboard storage-computer capabilities. It's like an AWS DC in a box

Snowmobile: AWS Snowmobile is an Exabyte-scale data transfer service used to move extremely large amounts of data to AWS. A truck full of disks basically.

## [S3 Transfer Acceleration](https://docs.aws.amazon.com/AmazonS3/latest/dev/transfer-acceleration.html)

Instead of uploading files directly to your S3 bucket, you can use the AWS edge network.
Using a specific URL, you upload the file to your local edge and then the file will be uploaded to S3
an example or URL:  alessio-casco-accelerate.s3-accelerate.amazonaws.com

## [Static Website Using S3](https://docs.aws.amazon.com/AmazonS3/latest/dev/HowDoIWebsiteConfiguration.html)

* You can use bucket policies to make entire S3 buckets public
* You can use S3 to host only static websites.
* S3 Scales automatically to meet demand

[Permissions Required for Website Access](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteAccessPermissionsReqd.html)

On console: Amazon S3 => Your_Bucket => Permissions => Bucket Policy

```json
{
  "Version":"2012-10-17",
  "Statement":[{
    "Sid":"PublicReadGetObject",
        "Effect":"Allow",
      "Principal": "*",
      "Action":["s3:GetObject"],
      "Resource":["arn:aws:s3:::example-bucket/*"
      ]
    }
  ]
}
```

_[!!! Read the S3 FAQs before the exam !!!](https://aws.amazon.com/s3/faqs/)_
