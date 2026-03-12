# AWS Security Flaws Challenge

The flAWS challenge is created by Scott Piper to help individuals learn about AWS security through a series of interactive, hands-on challenges. The challenge simulates real-world security issues that can occur in AWS environments and guides participants through identifying and exploiting these issues. The challenge is divided into multiple levels that focus on specific aspects of AWS security. Each level presents a scenario where a security misconfiguration or vulnerability exists that must be identified and exploited in order to progress to the next level.
**Scope**: Everything is run out of a single AWS account, and all challenges are sub-domains of *flaws.cloud*.

## Level 1

Did a DNS lookup on flaws.cloud by running the following command: `dig flaws.cloud`

![DNS Lookup](<attachments/flAWS challenge/DNS lookup.png>)

The output shows the A Records which lists the IP addresses associated with the domain flaws.cloud.

Visiting 52.92.161.35 on the browser will direct to https://aws.amazon.com/s3/

![IP address on the web](<attachments/flAWS challenge/IP on the web browser.png>)

This shows flaws.cloud is hosted as an S3 bucket.

Then run a reverse lookup with the command `nslookup 52.92.136.107`

![Reverse DNS lookup](<attachments/flAWS challenge/Reverse DNS lookup.png>)

The `nslookup` command output for the IP address `52.92.136.107` provides a reverse DNS lookup, translating the IP address back into a domain name, `s3-website-us-west-2.amazonaws.com`.

This reveals that flaws.cloud is hosted in the AWS region `us-west-2` and the IP address 52.92.136.107 is indeed part of Amazon's infrastructure, specifically linked to an Amazon S3 website endpoint.

Now we browse the bucket by using the AWS CLI by running: `aws s3 ls s3://flaws.cloud --no-sign-request --region us-west-2`

![Browse bucket with AWS CLI](<attachments/flAWS challenge/Browse bucket with AWS CLI.png>)

The output of the `aws s3 ls` command lists the contents of the S3 bucket flaws.cloud in the us-west-2 region, without requiring authentication (`--no-sign-request`). The use of --no-sign-request indicates that the bucket allows public access to its contents without requiring authentication.

Each line lists a file in the bucket, along with its size (in bytes) and last modification date.

Out of the lines, the `secret-dd02c7c.html` looks interesting. It might contain sensitive or hidden information.

So, attach the secret page onto the url flaws.cloud to see where it takes you: `flaws.cloud/secret-dd02c7c.html`

![Level 1 Completed](<attachments/flAWS challenge/Completed level 1.png>)

The `secret-dd02c7c.html` takes you to the end of Level 1 page that gives access to the beginning of Level 2.

Level 2 is at `http://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud`

## Level 2

An AWS key is needed to use the AWS CLI. Similar to the first level, we discover that this sub-domain is hosted as an S3 bucket with the name "level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud".

Its permissions are too loose, but you need your own AWS account to see what's inside.
So the first step is to configure your profile:

![AWS Profile configuration](<attachments/flAWS challenge/Profile configuration.png>)

Then run: `aws s3 --profile [AWS_ACCOUNT] ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud`
E.g. `aws s3 --profile cli-olick ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud`

![Browse AWS CLI with authenticated account](<attachments/flAWS challenge/Browse AWS bucket authenticated.png>)

The output of the aws s3 ls command lists the contents of another S3 bucket (level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud) using a specified profile (cli-olick).
From the line list, secret-e4443fc.html seems interesting. It might contain some information. So, add the secret-e4443fc.html to our address bar to see where it takes you:

![Level 2 Completed](<attachments/flAWS challenge/Completed level 2.png>)

The `secret-e4443fc.html` page takes you to another page that gives access to Level 3.

Level 3 is at http://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud

## Level 3

This command lists the files in this directory:
`aws s3 ls s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud --no-sign-request --region us-west-2`

![Browse level 3 bucket](<attachments/flAWS challenge/Browse level 3 bucket.png>)

From the list output, you can see that this S3 bucket has a .git file. There are probably interesting things in it. Download this whole S3 bucket using:
`aws s3 sync s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/ . --no-sign-request --region us-west-2`

- `aws s3 sync`: AWS CLI command to synchronize the contents between an S3 bucket and a local directory.
- `s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/`: specifies the source S3 bucket.
- `.`: specifies the destination directory, which in this case is the current directory.
- `--no-sign-request`: tells the AWS CLI to make the request without signing it. This is typically used when accessing public buckets that do not require authentication.
- `--region us-west-2`: specifies the AWS region where the bucket is located.

![Downloaded S3 Bucket](<attachments/flAWS challenge/Downloaded S3 bucket.png>)

The output shows the files being downloaded from the S3 bucket to the local directory.

Use the `ls` command to list the contents of the directory. You can also use `ls -la` command to get additional information like hidden files (those starting with a dot `.`).

![List downloaded S3 bucket](<attachments/flAWS challenge/List downloaded S3 bucket.png>)

The `ls -la` command listed an additional `.git` folder that seems interesting to explore for access to the access keys.

People often accidentally add secret things to git repos, and then try to remove them without revoking or rolling the secrets. You can look through the history of a git repo by running this command: `git log`

![Git log](<attachments/flAWS challenge/Git log.png>)

The output shows two commits. The latest commit `b64c8dcfa8a39af06521cf4cb7cdce5f0ca9e526` with the message indicating an accidental addition.

The initial commit `f52ec03b227ea6094b04e43f475fb0126edb5a61` with a message indicating it was the first commit to the repository.

You can look at what a git repo looked like at the time of a commit by running this command: `git checkout f52ec03b227ea6094b04e43f475fb0126edb5a61`

![View git commit](<attachments/flAWS challenge/View git commit.png>)

The `git checkout f52ec03b227ea6094b04e43f475fb0126edb5a61` command switches the current working directory to the state of the repository at commit `f52ec03b227ea6094b04e43f475fb0126edb5a61`.

Use the `ls` command to list the contents in the directory. You find an additional file: `access_keys.txt`

![List files after git checkout](<attachments/flAWS challenge/List files after git checkout.png>)

Use the `cat access_keys.txt` command to display the contents of the `access_keys.txt` file.

You should have found the AWS key and secret. Create a new profile for it using: `aws configure --profile flaws`

![Exposed access keys](<attachments/flAWS challenge/Exposed access keys.png>)

Then to list S3 buckets using the new profile, run the following command:  `aws s3 ls --profile flaws`

![List S3 buckets with new profile](<attachments/flAWS challenge/List S3 buckets with new profile.png>)

The output gives a list of all S3 buckets associated with the flaws profile.

The `level4-1156739cfb264ced6de514971a4bef68.flaws.cloud` subdomain that is among the listed outputs gives us access to the next level, which is Level 4 when we click on it.

![Level 3 Completed](<attachments/flAWS challenge/Completed level 3.png>)

## Level 4

For the next level, we need to get access to the web page running on an EC2 at `4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud`

A snapshot was made of that EC2 shortly after nginx was setup on it.

You can snapshot the disk volume of an EC2 as a backup. In this case, the snapshot was made public, but you'll need to find it.

To do this, first we need the account ID, which we can get using the AWS key from the previous level: `aws --profile flaws sts get-caller-identity`

![AWS Account ID](<attachments/flAWS challenge/AWS account ID.png>)

Using that command also tells you the name of the account, which in this case is named `backup` (`arn:aws:iam::975426262029:user/backup`). The backups this account makes are snapshots of EC2s.

Next, discover the snapshot: `aws --profile flaws ec2 describe-snapshots --owner-id 975426262029 --region us-west-2`

![EBS Snapshot](<attachments/flAWS challenge/EBS Snapshot.png>)

We specify the owner ID just to filter the output.
The output shows details of the EBS snapshot owned by the AWS account `975426262029` in the `us-west-2` region.

- `Description: “”` – no description was provided for this snapshot.
- `Encrypted: false` – this snapshot is not encrypted
- `OwnerId: “975426262029”` – AWS account ID that owns the snapshot.
- `Progress: “100%”` – indicates the snapshot creation process is complete.
- `SnapshotId: “snap-0b49342abd1bdcb89”` – The unique identifier for the snapshot.
- `StartTime: “2017-02-28T01:35:12+00:00”` – The date and time when the snapshot creation was initiated.
- `State: “completed”` – current state of the snapshot. In this case, it is completed.
- `VolumeId: “vol-04f1c039bc13ea950”` – the ID of the EBS volume from which the snapshot was created.
- `VolumeSize: 8` – the size of the EBS volume (in GiB) from which the snapshot was created.
- `Tags: "Key": "Name", "Value": "flaws backup 2017.02.27"` – the snapshot has a tag with the key Name and the value flaws backup 2017.02.27.
- `StorageTier: “standard”` – the storage tier for the snapshot, which in this case is standard.

This snapshot appears to be a backup labeled as "flaws backup 2017.02.27," created from an 8 GiB EBS volume.
This snapshot is in `us-west-2`.

Now that you know the snapshot ID, you're going to want to mount it. You'll need to do this in your own AWS account.
First, use this command to create a volume using the snapshot: `aws --profile cli-olick ec2 create-volume --availability-zone us-west-2a --region us-west-2 --snapshot-id snap-0b49342abd1bdcb89`

![Creating volume using snapshot](<attachments/flAWS challenge/Creating volume using snapshot.png>)

The output indicates that a new EBS volume is being created from the snapshot `snap-0b49342abd1bdcb89` in the `us-west-2a` availability zone.

- `Iops: 100` – the number of I/O operations per second (IOPS) that the volume supports. This is typical for a general-purpose SSD (gp2) volume.
- `VolumeType: “gp2”` – the type of the EBS volume. In this case, it is a general-purpose SSD (gp2).
- `MultiAttachEnabled: false` – indicates that multi-attach is not enabled for this volume. Multi-Attach allows multiple instances to attach to a volume concurrently.

![Created EBS Volume on AWS console](<attachments/flAWS challenge/Created EBS Volume on AWS console.png>)

The above screenshot confirms that the creation of the new EBS volume is complete.

Now in the console you can create an EC2 in the us-west-2 region and in the storage options, choose the volume you just created.

- Navigate to the **EC2 Dashboard** and click on **Launch instance**.
- Input the name of the instance, **flaws_instance**.
- Then, choose **Amazon Linux** under Application and OS Images.
- Under Key pair (login), select **Create new key pair**.
- Name your Key pair, leave everything else as they are, then click on **Create key pair**.
- Once you click on **Create key pair**, the key pair will automatically download to your machine.
- Scroll down to the **Configure storage** section, click on **Add new volume**.
- Then click on **Advanced**.
- Then select **Yes** under the **Delete on termination** dropdown list.
- Copy the **Snapshot ID** from the terminal and paste it on the **Snapshot** dropdown section, wait till loading is complete and ***DO NOT*** leave the page till it’s done. (This takes about 2 to 3 minutes).
- Then click on **Launch instance**.

After launch is successfully initiated, you can click on the provided **Instance ID** to confirm the instance is there.

![Launched Instance](<attachments/flAWS challenge/Launched instance.png>)

Take note of the **Public IPv4 address**, as this will be needed to SSH into the EC2.

To find the syntax of the username, Google search **‘aws manage system users on your Linux instance’**. This is significant since it is part of the syntax that will be used to SSH into the EC2.
For Amazon Linux, the default username is `ec2-user`.

SSH in with this command: `ssh -i flaws-key-pair.pem ec2-user@18.246.191.117`

The output gives an error message that indicates that the permissions on the private key file (`flaws-key-pair.pem`) are too open. SSH requires that your private key file is only accessible by the owner, meaning it should not be readable or writable by anyone else.

The following command can be run to change the permissions to `400` (readable only by the owner): `chmod 400 flaws-key-pair.pem`

- `chmod`: Change mode, a command used to change the file system modes or permissions of files and directories.
- `400`: The permission set for the file. The first digit `4` means the owner has read permission. The second and third digits `0` mean that neither the group nor others have any permissions.
- `flaws-key-pair.pem`: The file whose permissions are being changed.

Setting the file permissions to `400` ensures that only the file owner can read the file, and no one else can read, write or execute it. This is required by SSH for security reasons, to prevent unauthorized access to your private key.

After changing the permissions, try connecting to the EC2 instance again using the SSH command: `ssh -i flaws-key-pair.pem ec2-user@18.246.191.117`

![SSH to EC2 instance](<attachments/flAWS challenge/SSH to EC2 instance.png>)

Successfully connected to the EC2 instance using SSH. Now logged in as the `ec2-user` on the Amazon Linux 2023 instance.

We'll need to mount this extra volume by running: `lsblk`

The `lsblk` command is used to list information about all available block devices on your system. Block devices are storage devices that use a block-oriented access method, such as hard drives, SSDs and USB drives. This command is useful for viewing details about disks and partitions.

From the output, we have two block devices attached to our EC2 instance:

`nvme0n1`: The root disk (8GiB), with partitions:

- `nvme0n1p1`: The root partition mounted at ‘/’
- `nvme0n1p127`: A 1MiB partition (typically used for system metadata or reserved purposes)
- `nvme0n1p128`: A 10MiB partition mounted at /boot/efi (used for EFI boot)
- `xvdb`: An 8GiB disk with a single partition:
- `nvme1n1p1`: A partition that is not yet mounted.

The `file -s` command is used to display the type of a file or device. When applied to a block device like `/dev/nvme1n1p1`, it can show information about the filesystem on that device.

`sudo file -s /dev/nvme1n1p1`

The output indicates that the partition is formatted with the ext4 filesystem and has some specific details including UUID (Universally Unique Identifier) and Volume Name.

Next, we mount it using the following command: `sudo mount /dev/nvme1n1p1 /mnt`

Run the `lsblk` command again to confirm if it has been mounted successfully.

![Mount volume](<attachments/flAWS challenge/Mount volume.png>)

The output indicates you’ve successfully mounted `/dev/nvme1n1` to `/mnt`.

Once you've attached the volume, you'll want to look around for something that might tell you the password.
You can do that by navigating to the /mnt directory using this command:

- `cd /mnt` to navigate to mount.
- `ls` lists the contents of the /mnt directory.
- `cd home` to navigate into the home directory.
- `ls` lists the contents of the home directory.
- `cd ubuntu` to navigate into the ubuntu directory.
- `ls` lists the contents of the ubuntu directory.

![Navigating mounted directory](<attachments/flAWS challenge/Navigating mounted directory.png>)

In the ubuntu user's home directory is the file: `/home/ubuntu/setupNginx.sh`

View contents of the setupNginx.sh using this command: `cat setupNginx.sh`

![View contents of file in mounted directory](<attachments/flAWS challenge/View contents of file in mounted directory.png>)

The `setupNginx.sh` script contains a single command that is using `htpasswd` to create or update a user in an Nginx password file.

- `htpasswd`: is a command-line utility used to create and update the flat-files used to store usernames and password pairs for basic authentication of HTTP users.
- `-b`: this option tells `htpasswd` to use batch mode, where the password is passed on the command line rather than being prompted for. This is less secure because the password will be visible in the command history and process list.
- `/etc/nginx/.htpasswd`: this is the file where the username and password will be stored. If the file does not exist, `htpasswd` will create it.
- `flaws`: this is the username being created or updated in the password file.
- `nCP8xigdjpjyiXgJ7nJu7rw5Ro68iE8M`: this is the password for the user `flaws`.

This creates the basic HTTP auth user: `htpasswd -b /etc/nginx/.htpasswd flaws nCP8xigdjpjyiXgJ7nJu7rw5Ro68iE8M`

That is the username and password that will be used to sign in. Enter those at `4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud`

![Fill in credentials](<attachments/flAWS challenge/Fill in credentials.png>)

After entering those credentials to sign in, it takes us to Level 5.

![Level 4 completed](<attachments/flAWS challenge/Completed level 4.png>)

## Level 5

This EC2 has a simple HTTP only proxy on it.
On cloud services, including AWS, the IP 169.254.169.254 is magical. It's the metadata service.
cURL this address from the terminal: `curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/`

The output is a list of dates representing the versions of the EC2 instance metadata service. Each date corresponds to a version of the metadata available. The latest indicates the most recent version available.
Attach the latest metadata path to the base URL to access it as shown: `curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/`

The output shows categories of metadata available for the EC2 instance:

- `dynamic`: contains dynamic instance metadata that can change during the lifetime of the instance. This usually includes information related to instance identity documents and scheduled events.
- `meta-data`: contains static metadata about the instance, such as instance ID, instance type, AMI ID and network information. This is the most commonly accessed category for obtaining instance-specific details

Attach meta-data to the URL by running this command: `curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/`

The output provides detailed list of metadata available under the meta-data category for your EC2 instance. `iam/` contains information about the IAM roles associated with the instance.

1. Get into `iam/` by running this command:
`curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/iam/`

2. Then into security-credentials/ by running this command:
`curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/iam/security-credentials/`

3. Then get into flaws using this command:
`curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/iam/security-credentials/flaws/`

![Leaked credentials](<attachments/flAWS challenge/Leaked credentials.png>)

You should have found AWS credentials provided by the IAM role of the EC2.

Configure a new profile with these credentials: `aws configure --profile level5`

Note that you need to specify the Token, otherwise you’ll get an InvalidAccessKeyId error.

The credentials allow us to list the contents of the level 6 bucket.
Use the following command to look in the bucket:
`aws s3 ls level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud --profile level5`

![View level 6 bucket](<attachments/flAWS challenge/View level 6 bucket.png>)

The output gives `ddcc78ff/` and `index.html`
The output shows the contents of the S3 bucket `level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud`.

- `PRE ddcc78ff/`: indicates a directory (or prefix) named ddcc78ff within the S3 bucket. S3 doesn’t have a hierarchical file system like traditional file systems, but uses prefixes to mimic directories.
- `871 index.html`: indicates a file named index.html within the S3 bucket, and date and time shows when the file was last modified. 871 is the size of the files in bytes.

As you explore `http://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud/ddcc78ff/` you access Level 6.

![Level 5 completed](<attachments/flAWS challenge/Completed level 5.png>)

## Level 6

For this final challenge, you're getting a user access key that has the SecurityAudit policy attached to it. We see what else it can do and what else we might find in this AWS account.

- Access key ID: `AKIA.....`
- Secret: `S2I...+dZ4ps+u`

Create a profile and enter the credentials above using the command: `aws configure --profile level6`

The SecurityAudit group can get a high-level overview of the resources in an AWS account, but it's also useful for looking at IAM policies.
First, find out who you are: `aws --profile level6 iam get-user`

The output provides details about the IAM user named Level6:

- `Path: ‘/’`: is a way to group users in a hierarchy. In this case, the user is at the root level (‘ /’).
- `UserName`: the name of the IAM user.
- `UserId`: the unique identifier for the IAM user.
- `Arn`: Amazon Resource Name, uniquely identifies the IAM user.
- `CreateDate`: date and time when the IAM user was created.

Now that you know your username is Level6, you can find out what policies are attached to it with the command: `aws --profile level6 iam list-attached-user-policies --user-name Level6`

![View policies attached to profile](<attachments/flAWS challenge/View policies attached to profile.png>)

Now you know that you have two policies attached:

- `“MySecurityAudit”`
- `“list_apigateways”`

Once you know the ARN for the policy you can get its version ID:
`aws --profile level6 iam get-policy --policy-arn arn:aws:iam::975426262029:policy/list_apigateways`

Now that you have the ARN and the version ID, you can see what the actual policy is:
`aws --profile level6 iam get-policy-version  --policy-arn arn:aws:iam::975426262029:policy/list_apigateways --version-id v4`

The policy grants permission to perform the `GET` action on all API Gateway resources in the `us-west-2` region. The `Allow` effect means that the specified action (`apigateway:GET`) is permitted on the given resource.
This tells us using this policy we can call `"apigateway:GET"` on `"arn:aws:apigateway:us-west-2::/restapis/*"`

Using this, it should lead you in the direction of where the hidden resource is.
You now know that you have the ability to GET "arn:aws:apigateway:us-west-2::/restapis/*"
The API gateway in this case is used to call a lambda function, but you need to figure out how to invoke it.

The SecurityAudit policy lets you see some things about lambdas:
`aws --region us-west-2 --profile level6 lambda list-functions`

![Listed lambda functions](<attachments/flAWS challenge/Listed lambda functions.png>)

From the output, this Lambda function, named `Level6` is an AWS Lambda function written in Python 2.7. The function assumes the `Level6` IAM role and uses `zip` as the deployment package type. Logging is enabled with a log group `/aws/lambda/Level6`, and the function does not have advanced features like SnapStart enabled.

That tells you there is a function named “Level6”, and the SecurityAudit also lets you run:
`aws --region us-west-2 --profile level6 lambda get-policy --function-name Level6`

![View policy](<attachments/flAWS challenge/View policy.png>)

From the output, this policy allows the AWS API Gateway service (`apigateway.amazonaws.com`) to invoke the Lambda function `Level6`. The invocation is restricted to requests originating from a specific API Gateway (`arn:aws:execute-api:us-west-2:975426262029:s33ppypa75/*/GET/level6`).

This setup is typical for securing Lambda functions that are triggered by API Gateway, ensuring that only authorized API Gateway endpoints can invoke the function.

This tells you about the ability to execute `arn:aws:execute-api:us-west-2:975426262029:s33ppypa75/*/GET/level6\`

That "s33ppypa75" is a rest-api-id, which you can then use with that other attached policy:
`aws --profile level6 --region us-west-2 apigateway get-stages --rest-api-id "s33ppypa75"`

![View API gateway](<attachments/flAWS challenge/View API gateway.png>)

From the output, the API gateway with `rest-api-id “s33ppypa75”` has a single stage named `“Prod”`. This stage does not have caching enabled, does not have specific method settings and does not have tracing enabled. The stage was created and last updated on `2017-02-27`. The `deploymentId` for this stage is `“8gppiv”`.

That tells you the stage name is “Prod”. Lambda functions are called using that rest-api-id, stage name, region and resource as: `https://s33ppypa75.execute-api.us-west-2.amazonaws.com/Prod/level6`

Clicking on the link takes you to the following URL, which tells you to visit another one which marks the end of the challenge.

![View API URL](<attachments/flAWS challenge/View API URL.png>)

![Level 6 Completed](<attachments/flAWS challenge/Completed level 6.png>)

## Conclusion

In Level 1, we were able to list the contents of the S3 bucket *flaws.cloud* without requiring authentication. This means that the bucket allows public access to its contents without requiring authentication. That is a flaw that has been introduced by the owner that allows "Everyone" to have "List" permissions, since by default, S3 buckets are private and secure when they are created. The point of introduction of the flaw can be seen in the image below:

In Level 2, we were able to list the contents of another S3 bucket using our own authenticated AWS account. This was possible due to loose permissions by the bucket's owner. Similar to opening permissions to "Everyone", people accidentally open permissions to "Any Authenticated AWS User". They might mistakenly think this will only be users of their account, when in fact it means anyone that has an AWS account. This mistake can be avoided by opening permissions to specific AWS users only. This image shows where the flaw was introduced by the bucket's owner.

Note: This screenshot is from the web console in 2017. This setting can no longer be set in the web console, but the SDK and third-party tools sometimes allow it.
To view that public access has been blocked to all in the web console, navigate to Amazon S3 > Buckets > [Bucket_Name] > Permissions

In Level 3, we were able to access the access keys that were not deleted from the git file that was in the S3 bucket. People often leak AWS keys and then try to cover up their mistakes without revoking the keys. You should always revoke any AWS keys (or any secrets) that could have been leaked or were misplaced. Roll your secrets early and often. Rolling secrets means that you revoke the keys (i.e. delete them from the AWS account) and generate new ones. Another interesting issue this level has exhibited, although not that worrisome, is that you can't restrict the ability to list only certain buckets in AWS, so if you want to give an employee the ability to list some buckets in an account, they will be able to list them all. The key you used to discover this bucket can see all the buckets in the account. You can't see what is in the buckets, but you'll know they exist. Similarly, be aware that buckets use a global namespace meaning that bucket names must be unique across all customers, so if you create a bucket named `merger_with_company_Y` or something that is supposed to be secret, it's technically possible for someone to discover that bucket exists.

In Level 4, we see that AWS allows you to make snapshots of EC2's and databases (RDS). The main purpose for that is to make backups, but people sometimes use snapshots to get access back to their own EC2's when they forget the passwords. This also allows attackers to get access to things. Snapshots are normally restricted to your own account, so a possible attack would be an attacker getting access to an AWS key that allows them to start/stop and do other things with EC2's and then uses that to snapshot an EC2 and spin up an EC2 with that volume in your environment to get access to it. Like all backups, you need to be cautious about protecting them.

In Level 5, we learned that the IP address 169.254.169.254 is special in cloud environments. Cloud providers like AWS, Azure, Google, and DigitalOcean use it to let their resources (like virtual machines) get information about themselves, called metadata. For example, Google requires an extra step where the request must include a Metadata-Flavor: Google header and won't accept requests with an X-Forwarded-For header. AWS has a newer, more secure version of this service called IMDSv2, which needs special headers and a challenge-response process, but not all AWS accounts use this new version yet. If you can send a HTTP request to 169.254.169.254 from an EC2 instance, you'll likely receive information that the owner wouldn't want to be exposed.

In Level 6, we see that it is common to give people and entities read-only permissions such as the SecurityAudit policy. The ability to read your own and others' IAM policies can really help an attacker figure out what exists in your environment and look for weaknesses and mistakes. This mistake can be avoided by not handing out any permissions liberally, even permissions that only let you read metadata or know what your permissions are.
