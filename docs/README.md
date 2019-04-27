* Browse to the [EC2 Dashboard](https://console.aws.amazon.com/ec2/)

* Choose **Launch Instance**.

* Choose **Amazon Linux 2 AMI (HVM), SSD Volume Type**, then choose **Select**.

* Choose **t2.micro Free tier eligble**

* Choose **Review and Launch**, taking you to the **Review Instance Launch** page.

It has created a security gropu, called something like `launch-wizard 1`.

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
| HTTP | TCP        | TCP      | 80           | 0.0.0.0/0          |             |
| HTTP | TCP        | TCP      | 80           | ::/0               |             |
| Custom TCP Rule   | TCP      | 8080         | 0.0.0.0/0          |             |
| Custom TCP Rule   | TCP      | 8080         | ::/0               |             |
| SSH               | TCP      | 22           | 73.83.199.86/32    |             |

Based on:
* [Getting Started with Amazon EC2 Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)

