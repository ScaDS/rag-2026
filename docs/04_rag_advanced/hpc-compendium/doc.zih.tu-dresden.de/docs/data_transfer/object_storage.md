# Transfer Data between ZIH Systems and Object Storage (S3)

Object Storage is an alternative to normal filesystem storage. It can be accessed via HTTPS and can
therefor be used where direct `scp` connections are prohibited, e.g. when one wants to copy data to
another data center. Access is provided on request via the corresponding
[self service page on object storage](https://selfservice.tu-dresden.de/services/objectstorage/).
The access key (`Zugriffsschlüssel`) and the secret key (`Geheimer Schlüssel`) are required later
when copying data to it.

Access to object storage is possible on ZIH systems via the module `rclone`:

```console
marie@login$ module load rclone
```

## Initial Configuration

Before you use `rclone` for the first time, you have to configure it. This is done interactively as
shown below. Replace `REPLACE_ME_WITH_ACCESS_KEY` and `REPLACE_ME_WITH_SECRET_KEY` with the values
from the self service portal:

```console
marie@login$ rclone config
2023/03/22 09:35:55 NOTICE: Config file "/home/marie/.config/rclone/rclone.conf" not found - using defaults
No remotes found, make a new one?
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n

Enter name for new remote.
name> s3store

Option Storage.
Type of storage to configure.
Choose a number from below, or type in your own value.
 1 / 1Fichier
   \ (fichier)
 2 / Akamai NetStorage
   \ (netstorage)
 3 / Alias for an existing remote
   \ (alias)
 4 / Amazon Drive
   \ (amazon cloud drive)
 5 / Amazon S3 Compliant Storage Providers including AWS, Alibaba, Ceph, China Mobile, Cloudflare, ArvanCloud, Digital Ocean, Dreamhost, Huawei OBS, IBM COS, IDrive e2, IONOS Cloud, Lyve Cloud, Minio, Netease, RackCorp, Scaleway, SeaweedFS, StackPath, Storj, Tencent COS, Qiniu and Wasabi
   \ (s3)
 6 / Backblaze B2
   \ (b2)
 7 / Better checksums for other remotes
   \ (hasher)
 8 / Box
   \ (box)
 9 / Cache a remote
   \ (cache)
10 / Citrix Sharefile
   \ (sharefile)
11 / Combine several remotes into one
   \ (combine)
12 / Compress a remote
   \ (compress)
13 / Dropbox
   \ (dropbox)
14 / Encrypt/Decrypt a remote
   \ (crypt)
15 / Enterprise File Fabric
   \ (filefabric)
16 / FTP
   \ (ftp)
17 / Google Cloud Storage (this is not Google Drive)
   \ (google cloud storage)
18 / Google Drive
   \ (drive)
19 / Google Photos
   \ (google photos)
20 / HTTP
   \ (http)
21 / Hadoop distributed file system
   \ (hdfs)
22 / HiDrive
   \ (hidrive)
23 / In memory object storage system.
   \ (memory)
24 / Internet Archive
   \ (internetarchive)
25 / Jottacloud
   \ (jottacloud)
26 / Koofr, Digi Storage and other Koofr-compatible storage providers
   \ (koofr)
27 / Local Disk
   \ (local)
28 / Mail.ru Cloud
   \ (mailru)
29 / Mega
   \ (mega)
30 / Microsoft Azure Blob Storage
   \ (azureblob)
31 / Microsoft OneDrive
   \ (onedrive)
32 / OpenDrive
   \ (opendrive)
33 / OpenStack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ (swift)
34 / Oracle Cloud Infrastructure Object Storage
   \ (oracleobjectstorage)
35 / Pcloud
   \ (pcloud)
36 / Put.io
   \ (putio)
37 / QingCloud Object Storage
   \ (qingstor)
38 / SMB / CIFS
   \ (smb)
39 / SSH/SFTP
   \ (sftp)
40 / Sia Decentralized Cloud
   \ (sia)
41 / Storj Decentralized Cloud Storage
   \ (storj)
42 / Sugarsync
   \ (sugarsync)
43 / Transparently chunk/split large files
   \ (chunker)
44 / Union merges the contents of several upstream fs
   \ (union)
45 / Uptobox
   \ (uptobox)
46 / WebDAV
   \ (webdav)
47 / Yandex Disk
   \ (yandex)
48 / Zoho
   \ (zoho)
49 / premiumize.me
   \ (premiumizeme)
50 / seafile
   \ (seafile)
Storage> 5

Option provider.
Choose your S3 provider.
Choose a number from below, or type in your own value.
Press Enter to leave empty.
 1 / Amazon Web Services (AWS) S3
   \ (AWS)
 2 / Alibaba Cloud Object Storage System (OSS) formerly Aliyun
   \ (Alibaba)
 3 / Ceph Object Storage
   \ (Ceph)
 4 / China Mobile Ecloud Elastic Object Storage (EOS)
   \ (ChinaMobile)
 5 / Cloudflare R2 Storage
   \ (Cloudflare)
 6 / Arvan Cloud Object Storage (AOS)
   \ (ArvanCloud)
 7 / Digital Ocean Spaces
   \ (DigitalOcean)
 8 / Dreamhost DreamObjects
   \ (Dreamhost)
 9 / Huawei Object Storage Service
   \ (HuaweiOBS)
10 / IBM COS S3
   \ (IBMCOS)
11 / IDrive e2
   \ (IDrive)
12 / IONOS Cloud
   \ (IONOS)
13 / Seagate Lyve Cloud
   \ (LyveCloud)
14 / Minio Object Storage
   \ (Minio)
15 / Netease Object Storage (NOS)
   \ (Netease)
16 / RackCorp Object Storage
   \ (RackCorp)
17 / Scaleway Object Storage
   \ (Scaleway)
18 / SeaweedFS S3
   \ (SeaweedFS)
19 / StackPath Object Storage
   \ (StackPath)
20 / Storj (S3 Compatible Gateway)
   \ (Storj)
21 / Tencent Cloud Object Storage (COS)
   \ (TencentCOS)
22 / Wasabi Object Storage
   \ (Wasabi)
23 / Qiniu Object Storage (Kodo)
   \ (Qiniu)
24 / Any other S3 compatible provider
   \ (Other)
provider> 24

Option env_auth.
Get AWS credentials from runtime (environment variables or EC2/ECS meta data if no env vars).
Only applies if access_key_id and secret_access_key is blank.
Choose a number from below, or type in your own boolean value (true or false).
Press Enter for the default (false).
 1 / Enter AWS credentials in the next step.
   \ (false)
 2 / Get AWS credentials from the environment (env vars or IAM).
   \ (true)
env_auth> 1

Option access_key_id.
AWS Access Key ID.
Leave blank for anonymous access or runtime credentials.
Enter a value. Press Enter to leave empty.
access_key_id> REPLACE_ME_WITH_ACCESS_KEY

Option secret_access_key.
AWS Secret Access Key (password).
Leave blank for anonymous access or runtime credentials.
Enter a value. Press Enter to leave empty.
secret_access_key> REPLACE_ME_WITH_SECRET_KEY

Option region.
Region to connect to.
Leave blank if you are using an S3 clone and you don't have a region.
Choose a number from below, or type in your own value.
Press Enter to leave empty.
   / Use this if unsure.
 1 | Will use v4 signatures and an empty region.
   \ ()
   / Use this only if v4 signatures don't work.
 2 | E.g. pre Jewel/v10 CEPH.
   \ (other-v2-signature)
region> 1

Option endpoint.
Endpoint for S3 API.
Required when using an S3 clone.
Enter a value. Press Enter to leave empty.
endpoint> s3.zih.tu-dresden.de

Option location_constraint.
Location constraint - must be set to match the Region.
Leave blank if not sure. Used when creating buckets only.
Enter a value. Press Enter to leave empty.
location_constraint> 

Option acl.
Canned ACL used when creating buckets and storing or copying objects.
This ACL is used for creating objects and if bucket_acl isn't set, for creating buckets too.
For more info visit https://docs.aws.amazon.com/AmazonS3/latest/dev/acl-overview.html#canned-acl
Note that this ACL is applied when server-side copying objects as S3
doesn't copy the ACL from the source but rather writes a fresh one.
Choose a number from below, or type in your own value.
Press Enter to leave empty.
   / Owner gets FULL_CONTROL.
 1 | No one else has access rights (default).
   \ (private)
   / Owner gets FULL_CONTROL.
 2 | The AllUsers group gets READ access.
   \ (public-read)
   / Owner gets FULL_CONTROL.
 3 | The AllUsers group gets READ and WRITE access.
   | Granting this on a bucket is generally not recommended.
   \ (public-read-write)
   / Owner gets FULL_CONTROL.
 4 | The AuthenticatedUsers group gets READ access.
   \ (authenticated-read)
   / Object owner gets FULL_CONTROL.
 5 | Bucket owner gets READ access.
   | If you specify this canned ACL when creating a bucket, Amazon S3 ignores it.
   \ (bucket-owner-read)
   / Both the object owner and the bucket owner get FULL_CONTROL over the object.
 6 | If you specify this canned ACL when creating a bucket, Amazon S3 ignores it.
   \ (bucket-owner-full-control)
acl> 1

Edit advanced config?
y) Yes
n) No (default)
y/n> n
```

## Copying Data from/to Object Storage

The following commands show how to create a bucket `mystorage` in your part of the object store:

```console
marie@login$ module load rclone
marie@login$ rclone mkdir s3store:mystorage
```

After these commands, you can copy a file, e. g. `largedata.tar.gz`, to it in a separate job with
the help of the [Datamover](datamover.md). Adjust the parameters `time` and `account` as required:

```console
marie@login$ dtrclone --time=0:10:00 --account=p_number_crunch copy --s3-acl "public-read" largedata.tar.gz s3store:mystorage
```

!!! warning "Restricted access"

    If you want to restrict access to your data, replace the last command with:

    ```console
    marie@login$ dtrclone --time=0:10:00 --account=p_number_crunch copy largedata.tar.gz s3store:mystorage
    ```

    Then, it is not possible to access your data without providing your credentials.

For small files, you can also directly copy data:

```console
marie@login$ module load rclone
marie@login$ rclone copy --s3-acl "public-read" largedata.tar.gz s3store:mystorage
```

## Accessing the Object Storage

The following commands show different possibilities to access a file from object storage.

### Copying a File from Object Storage to ZIH systems

```console
marie@login$ dtrclone --time=0:10:00 --account=p_number_crunch copy s3store:mystorage/largedata.tar.gz .
```

### Copying a File from Object Storage to Your Workstation

The following command assumes you have installed the command `s3cmd`, please also see the
[s3cmd Installation Instructions](https://tu-dresden.de/zih/dienste/service-katalog/arbeitsumgebung/datenspeicher/objektspeicher-s3)

```console
marie@local$ s3cmd get s3://mystorage/largedata.tar.gz
```

### Accessing a Public-Readable File

It is possible to copy a public-readable file via `wget` or similar command line tools. Replace
`$USER` with your ZIH account.

```console
marie@somewhere$ wget https://s3.zih.tu-dresden.de/$USER:mystorage/largedata.tar.gz
```
