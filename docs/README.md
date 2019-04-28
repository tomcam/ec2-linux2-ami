## Creating an Amazon Linux 2 instance and configuring it to run Golang programs

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

When it's ready, click the **Connect** button between **Launch Instance** and **Actions** 
on your dashboard.

## Connecting with SSH

This section is based on [Connecting to Your Linux Instance Using SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)

## Not covered properly

* Passwords are just dead. It's all .PEM files.
* PEM files seem to contain both public and private keys
* An overview of using SSH
* How to deal with stopping an instance, then trying to get back in (SSH fingerprint error)
* Create an SSH directory `mkdir -p ~/.ssh`

### Notes

* A warning says `Rules with source of 0.0.0.0/0 allow all IP addresses to access your instance. We recommend setting security group rules to allow access from known IP addresses only.` But isn't that exactly what I want for a website?

* Be sure to use [Billing Preferences](https://console.aws.amazon.com/billing/home?#/preferences) to notify you
of cost overruns.

Based on:
* [Getting Started with Amazon EC2 Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
* [Connecting to Your Linux Instance Using SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)


