# Creating an Amazon Linux 2 instance and configuring it to run Golang programs

## TODO
* Learn how to get SSL certificate



Shows how create a simple Amazon Linux 2 running Golang, using the default Go web server,
in a simple web app.

* Launch a new t2.micro instance
* Create root user credentials
* Create a new user with sudo privileges
* Install Go
* Configure the built-in firewall (security group) to serve web pages and accept ssh connections
* Connect your website to a domain name
* Get an SSL certificate for the domain name
* Compile and install a tiny web app in Go
* Make the web app visible to the world

This is mostly for me because I'm bad at following multiple web pages linking all
over the place. It's MacOS-oriented, assuming a bash shell.

This section is based on [Getting Started with Amazon EC2 Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)

* Browse to the [EC2 Dashboard](https://console.aws.amazon.com/ec2/)

* Choose **Launch Instance**.

* Choose **Amazon Linux 2 AMI (HVM), SSD Volume Type**, then choose **Select**.

* Choose **t2.micro Free tier eligble**

* Choose **Review and Launch**, taking you to the **Review Instance Launch** page.

It has created a security group, called something like `launch-wizard 1`.

* Choose **Edit Security Groups**

The **Configure Security Group** page appears.

Choose *Select an existing security group*, then add a checkmark to `launch-wizard-1` or whatever it's called.

* Choose **Review and Launch**

The **Review Instance Launch** page appears.

A warning appears looking something like this:

```
Improve your instances' security. Your security group, 
launch-wizard-1, is open to the world.

Your instances may be accessible from any IP address. 
We recommend that you update your security group rules to a
llow access from known IP addresses only.

You can also open additional ports in your security group to 
facilitate access to the application or service you're running, 
e.g., HTTP (80) for web servers. Edit security groups
```

## TODO:

* Research the above message. Does it mean that my security group itself is open
to the world, or does it mean my security group is describing that 
the world is allowed to visit my website?

## TODO:
* Learn what 0.0.0.0/0 means for Source. I assume it means everyone. What's the `/0` or `/32`appended?
* Chart below: The online help for Source should explain what `0.0.0.0/0` means.


| Type              | Protocol | Port Range   | Source             | Description |
|-------------------|----------|--------------|--------------------|-------------|
| HTTP              | TCP      | 80           | 0.0.0.0/0          |             |
| HTTP              | TCP      | 80           | ::/0               |             |
| Custom TCP Rule   | TCP      | 8080         | 0.0.0.0/0          |             |
| Custom TCP Rule   | TCP      | 8080         | ::/0               |             |
| SSH               | TCP      | 22           | 173.82.199.34/32   |             |


* Choose **Launch**.

A **Select an existing key pair or create a new key pair** dialog appears.

* If you don't have one, choose **Create a new key pair** and follow instructions carefully (https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html for more details).

* I already have one so I choose it, then choose it.

* Check the "I acknowledge" box, then choose **Launch Instances**

The **Launch Status** page appears with a message saying
**Your instances are now launching**, with a link to the instance being
created.

## Collect and type  the following information

As soon as you need the following information, type it into a document
file of some kind. 
**In the case of passwords, type them before actually entering them, then 
copy to the system clipboard** and paste them in when asked for
a password. That way you don't type in the wrong thing, because password fields hide
what you type. Plus you have to do it twice.


| Note this             |  Write the value here      |
| --------------------- | -------------------------- |
| Instance name         |                            |
| Instance ID           |                            |
| Public DNS            | ec2-50-51-232-66.us-east-1.compute.amazonaws.com            |
| ssh connection string | ssh -i "sshkey.pem" ec2-user@ec2-50-51-232-66.us-east-1.compute.amazonaws.com
| default username      | ec2-user                   |
| password              |                            |
| IAM username          | Administrator              |
| password              |                            |
| PEM file              | ~/.ssh/sshkey.pem          |
| Access key ID         |                            |
| Secret access key     |                            |
| Location of AWS secret access key file | ~/.aws/config |
| Regin name            |                            |

## Visit the AWS Management Console to get your ssh connection string

* Go the AWS dashboard and under **Compute**, choose **EC2**.

* In the navigation pane, choose **Instances** under **INSTANCES**.

A page with your instance(s) will appear.

* Choose the row with the right Instance ID, then choose **Connect**.

A **Connect To Your Instance** dialog appears. 

* Next to **I would like to connect with** choose **A standalone SSH client**.

You're given a number of directions. *Read them carefully and follow them!* 
Among the most important you'll see that you need to change permissions
for the sshkey file. So change appropriately; here's an example. 

* In your local terminal program, run:

```
# Replace this with the filename and
# directory you created for your PEM file.
chmod 400 ~/.ssh/sshkey.pem 
```

The **Connect To Your Instance** dialog also has the
exact command you need to get access to your instance with SSH.
It will look something like this:

```
# -i specifies the file pathname of the keyfile
ssh -i "sshkey.pem" ec2-user@ec2-50-51-232-66.us-east-1.compute.amazonaws.com
```

### Note

The example they give may not work. Make sure you add the direct path to 
the PEM file after the `-i`. For example, on my system it would be

```
# -i specifies the file pathname of the keyfile
ssh -i "~/.ssh/sshkey.pem" ec2-user@ec2-50-51-232-66.us-east-1.compute.amazonaws.com
```
## I think hackernoon has it right. This is where to set up an Elastic IP

[https://hackernoon.com/getting-a-free-ssl-certificate-on-aws-a-how-to-guide-6ef29e576d22](https://hackernoon.com/getting-a-free-ssl-certificate-on-aws-a-how-to-guide-6ef29e576d22)

* From the AWS dashboard, choose **Services**, then search for `certificate`.

The AWS Certificate Manager appears.

* Under **Provision Certificates**, choose **Get started**.

The **Request a certificate** page appears.

* Choose **Request a public certificate**, then the **Request certificate** button.

You're on the **Add domain names** step.

* Under **Domain name**, fill in the domain name itself, for example, `example.com`.

* If you also have another variation, such as `www.example.com`, you can choose **Add another name to this certificate**
and include it.

* Choose **Next**.

The **Select validation method** page appears. This is where you must prove you own the domain, or at least
have control over it. That requires either modifying the DNS records or answering from one of the contact emails
given for the domain name. In this example we'll use DNS validation, because most domain names use
privacy shielding now.

* Choose **Review**.

A page appears, summarizing the changes you plan to make.

You're given the choice to export the DNS values to a file. 

* Choose **Export DNS configuration to a file**.

* If you used Route 53 to register your domain, you'll see a button that says **Create the record in Route 53**. Otherwise...

### Add a CNAME record for each alias

You need to log into your DNS provider (usually your domain name registrar, such as GoDaddy or Namecheap)
and find


## Configuring the AWS CLI

### Note/Amazon doc defect



A number of the following commands use the AWS CLI, or command line interface. 
You can't just start typing the commands, however, because they're cryptographically
signed. It turns out that you first have to
the [Configure the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html).
That's good, but the docs so far haven't mentioned it and the errors I was getting didn't point me
in that direction. I wanted to configure using the CLI using the command line (makes it easier to automate later).
I wanted to do this via the command line. As near as I can tell from [Creating Your First IAM Admin User and Group](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html), the second part of that page called "Creating an IAM User and Group (AWS CLI)" can't be implemented via the command line right after creating an instance. That's because it requires secrets.

## Create an Administrators group

* Go the [IAM Console](https://console.aws.amazon.com/iam/home?#home).

* From the navigation pane choose **Users**, under **Dashboard**.

* You'll see that there are no users defined yet.

*Add user* | *Delete user* 

| User name | Groups | Access key age | Password age | Last activity | MFA |
|-----------|--------|----------------|--------------|---------------|-----|
| *There are no IAM users. Learn more |

* Choose **Add user**.

The **Add user** page appears.

* Under Set user details, put `Administrator` for User name.

* Look for the checkbox next to **AWS Management Console access** and make sure
it's checked.

* At Console password, make a password choice. We'll choose Custom password. Type the password
down in your notes, then copy and paste it in.

* Uncheck **Require password reset** because, c'mon, you can handle this. You already wrote
down your password, you clever person.

* Choose **Next: Permissions**.

The **Set permissions** page appears.

* Choose **Add user to group**.

* Under **Get started with groups**, choose **Create group**.

A big **Create group** dialog appears with a long list of policy names.

* For **Group name**, type `Administrators`.

* Choose the **Filter policies** dropdown, and under **POLICY TYPE**, make sure you check **AWS managed - job function**.

The number of policies has been reduced (you set a filter). 

* In the **Policy name** column, find **AdministratorAccess** and make sure you check it.

* Choose **Create group**.

The new group appears:

*Create group* | *Refresh* 

| Groups         | Attached policies   |
|----------------|---------------------|
| Administrators | AdministratorAccess |

* Choose **Next: Tags**

This is a small operation so we don't need tags, which help organize large operations.

* Choose **Next: Review**

You're shown a summary of the work you've just 
done creating the group.

* Choose **Create user**

A page appears showing a link for the newly created role to sign in. There's
a long link looking something like this:

```
Administrator https://2649932634109.signin.aws.amazon.com/console
```

* Click the link to log in. You'll be taken to an **aws** login page 
with fields containing **Account ID or alias**, **IAM user name**, and **Password**.

**Account ID or alias**

| 2649932634109         |           
| --------------------- |

**IAM username**

| Administrator         |           
| --------------------- |

**Password**

| *************         |           
| --------------------- |

Note the new account ID, which is just a big long number. It's the same
number that started the URL you were just issued.

* Log in using the username you just created (`Administrator` in this example) 
and custom password you created a few steps back.

## Create your IAM access keys

* Visit the [IAM Console](https://console.aws.amazon.com/iam/home?#home) at [https://console.aws.amazon.com/iam/home?#home](https://console.aws.amazon.com/iam/home?#home) and in the navigation pane under **Dashboard**, choose **Users**.

* Choose the IAM username you just created, **Administrator** in this example.

A **Summary** page appears, with tabs like **Permissions**, **Groups**, **Tags**, and **Security Credentials**.

* Choose the **Security Credentials** tab, then under **Access keys** choose **Create access key**.

A **Create access key** dialog appears. It says:

```
This is the only time that the secret access keys can be viewed or downloaded. You cannot recover them later. However, you can create new access keys at any time.
```

Under it is a button that says **Download .csv file** and
Two strings (sets of numbers and letters) have been created for you:

| Access key ID  | Secret access key   |
|----------------|---------------------|
| AF4355111FADFZ | ************** [Show]() |

* As the explanation says, you get one chance to view the secret access key.
Do not give it to anyon outside your group.

* Choose **Show**, then copy the secret access key and paste it into your notes.

* Choose **Access key ID** and copy and paste it into your notes too.

* Choose Download .csv file and save it on your machine. The best place is
to create the directory `~/.aws/` and name the file `config`. 
Depending on your operating system and shell, you might do something like:

```
# Create the hidden aws directory.
mkdir ~/.aws

# Copy the downloaded keys to that directory
cp ~/Downloads/accessKeys.csv ~/.aws

# Rename the key file
mv ~/.aws/accessKeys.csv ~/.aws/config
```

* Double-check that you saved the file and the secret access key, then choose **Close**.

## Configure AWS CLI options

* At the prompt enter `aws configure --profile Administrator`

```
aws configure --profile Administrator
```

You're asked a series of questions. 

* Fill them in appropriately,
replacing the values shown here with the ones you've recorded:

```
AWS Access Key ID [None] AF4355111FADFZ
AWS Secret Access Key [None]ZDFAFDFDAFWWFDSAFA2AF$++_
Default region name [None]use-east-2
```

* Accept defaults for the other values.

A file named `credentials` containing these values has been written to  `~/.aws` directory.

## Connecting with SSH

When it's ready, click the **Connect** button between **Launch Instance** and **Actions** 
on your dashboard.

This section is based on [Connecting to Your Linux Instance Using SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)


## Set up your firewall (configure routes for VPC)

* Open the [EC console](https://console.aws.amazon.com/ec2/).

* In the navigation pane under **INSTANCES**, choose **Instances**.

* Choose the right instance under **Instance ID**.

* Choose the **Description** tab, then find **VPC ID** and choose that.

The VPC console appears for that VPC.

* Choose link under **Main Route table**

The route table's configuration page appears.

* Choose the **Routes** tab.

* Choose **Edit routes**.

There should be one with the Destination `0.0.0.0/0` and Target with an igw number.

### TODO: Figure out what to do if there's no such destination; see https://aws.amazon.com/premiumsupport/knowledge-center/ec2-linux-ssh-troubleshooting/?icmpid=support_rt_kc_articles

## Install go

* Update your system and install Go:
```
sudo yum update -y
sudo yum install -y golang
```

### Update $GOPATH

* Open your bash profile:

```
vim ~/.bash_profile
```

* Append these lines at the bottom of the file:

```
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin:/usr/local/go/bin
```

* Save the file and exit, then create the recommended Go workspace:

```
mkdir -p $HOME/go/{bin,src}
```

* Then create a test project:

```
mkdir -p $HOME/go/src/test
cd $HOME/go/src/test
```

## Install and configure Git

You can create a simple Go program if it uses only the standard library. 
But any packages on GitHub require that you install and configure Git.
For example, getting the Gorilla Mux package using `go get` as shown
here requires Git:

```
go get -u github.com/gorilla/mux
# Results in the following error :
# go: missing Git command. See https://golang.org/s/gogetcmd
```

### Download Git

Many Go repositories are on [GitHub](http://github.com), of course, so get an account there. It's free.

* From your remote machine, run `sudo yum install git`:

```
sudo yum install git
```

* Configure Git with your username and email address. Obviously replace the ones shown here with your own.

```
# Use your own name and email here, duh!
git config --global user.name "Tom Campbell"
git config --global user.email "tomcampbell@example.com"
```

## Getting an SSL certificate


## TODO: I think I need to cover
* [Creating Your First IAM Admin User and Group](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html). Reason: I wanted to get the fingerprint and to be able to script things like configuring the firewall. That requires AWS CLI access, which requires IAM secrets, which requires the creation of an IAM user, which requires the creation of a group.

### Get the instance fingerprint

### NOTE

* First have to start at the AWS Console, so to go [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

* Pretty sure this has to be preceded by [Creating Your First IAM Admin User and Group](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html) to get the AWS CLI signed, then configuring the CLI [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

aws ec2 get-console-output --instance-id

## Areas I need to improve

* Somewhere early note the region like use-east-2 or whatever
* Tradition in Linux is to create a new user immediately. Near as I can tell they've
done that already by creating ec2-user, right? And if that's true does it pose
a security threat since most people could guess the username? Or is the theory that
requiring a PEM file pretty much obviates that need?
* Passwords are just dead. It's all .PEM files.
* PEM files seem to contain both public and private keys
* An overview of using SSH
* How to deal with stopping an instance, then trying to get back in (SSH fingerprint error)
* Create an SSH directory `mkdir -p ~/.ssh`
* AWS-CLI [describe-instances](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html)
* Getting [ssh](http://www.openssh.com/)

### Notes

* A warning says `Rules with source of 0.0.0.0/0 allow all IP addresses to access your instance. We recommend setting security group rules to allow access from known IP addresses only.` But isn't that exactly what I want for a website?
* Be sure to use [Billing Preferences](https://console.aws.amazon.com/billing/home?#/preferences) to notify you
of cost overruns.
* [Region List](https://docs.aws.amazon.com/general/latest/gr/rande.html)

### Amazon doc defects

* [(Optional) Get the Instance Fingerprint](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connection-prereqs.html#connection-prereqs-fingerprint) gives incomplete instructions for getting the instance ID fingerprint.
They give the example `aws ec2 get-console-output --instance-id instance_id` but when I did it I got the error `You must specify a region. You can also configure your region by running "aws configure".` I found an example of specifying a region on [Stack Overflow](https://stackoverflow.com/questions/29166957/error-you-must-specify-a-region-when-running-command-aws-ecs-list-container-inst) which looked more like this: `aws ec2 get-console-output --instance-id instance_id --region us-east-1`. When I tried it I got the error `Unable to locate credentials. You can configure credentials by running "aws configure"`. So I think that passage is missing a whole section.  I think it needs to be precded. And getting the fingerprint probably isn't optional, because I'm sure I'm not the only one who ran into the fingerprint problem by just stopping an instance.
* I wanted to create the IAM User and Group using the AWS CLI as detailed on [Creating Your First IAM Admin User and Group](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html). The docs don't say it very well but you're supposed to do something like this to create a group named Admins:

```
aws iam create-group --group-name Administrators
```

However when I did it I got the error `Unable to locate credentials. You can configure credentials by running "aws configure".`
* **Blocking bug:** in the VPC route table part (second half) of [How do I troubleshoot problems connecting to my EC2 Linux instance using SSH?](https://aws.amazon.com/premiumsupport/knowledge-center/ec2-linux-ssh-troubleshooting/?icmpid=support_rt_kc_articles) it says "In the navigation pane, choose Route Tables and then select your VPC route table from the list" I have no idea what my VPC route table is. That's why I came to this article. 

## Troubleshooting
* I tried to run the command `aws ec2 get-console-output --instance-id i-065aa3e3f23ae9859 --region us-west-1`. I returned the error `Unable to locate credentials. You can configure credentials by running "aws configure".` I already did so.


## Based on:
* [Getting Started with Amazon EC2 Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
* [Connecting to Your Linux Instance Using SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)
* [Creating Your First IAM Admin User and Group](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html) to get the AWS CLI signed
* [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html), but [Setting up Amazon Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/setting-up-route-53.html#setting-up-create-iam-user) 
has a good summary of creating your first IAM user and group
* [Identities (Users, Groups, and Roles)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id.html)
* [AWS Tasks that Require Account Root User](https://docs.aws.amazon.com/general/latest/gr/aws_tasks-that-require-root.html)

## Related
* [Billing](https://console.aws.amazon.com/billing/home#/) (requires login)
* [How do I make sure I don't incur charges when I'm using the AWS Free Tier?](https://aws.amazon.com/premiumsupport/knowledge-center/free-tier-charges/?icmpid=support_rt_kc_articles)
* [Best Practices for Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-best-practices.html?icmpid=docs_support_createcase_category_amazon-elastic-compute-cloud-linux_general-guidance)
* [How do I troubleshoot problems connecting to my EC2 Linux instance using SSH?](https://aws.amazon.com/premiumsupport/knowledge-center/ec2-linux-ssh-troubleshooting/?icmpid=support_rt_kc_articles)
* [Deploying a Go application on AWS EC2](https://hackernoon.com/deploying-a-go-application-on-aws-ec2-76390c09c2c5)
* [How to set up an EC2 instance with Github, Node.js and PostgreSQL](https://medium.com/digitalcrafts/how-to-set-up-an-ec2-instance-with-github-node-js-and-postgresql-e363cb771826)
* [Getting a Free SSL Certificate on AWS a How-To Guide](https://hackernoon.com/getting-a-free-ssl-certificate-on-aws-a-how-to-guide-6ef29e576d22)

