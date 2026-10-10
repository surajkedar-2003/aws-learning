# AWS EC2

I've been working with EC2 in production for deploying and running applications. I've launched instances, configured security groups, assigned Elastic IPs, attached EBS volumes, set up IAM roles for S3 access, and used CloudWatch to monitor servers.

I knew how to do these things from my work, but I wanted to understand why we configure them and what happens when something goes wrong. Here are my notes from learning EC2.

## 1. Launching an EC2 Instance

When I launch an instance, I need to choose an AMI, instance type, storage, and network settings.

An AMI gives me the starting image for the server, usually with an operating system installed. The instance type decides how much CPU, memory, and networking capacity the server gets. I need to choose one based on the application I'm running.

I also went through user data. It lets me run a script when an instance launches. For example, I can use it to install packages or prepare the server instead of doing every setup step manually.

## 2. Security Groups and Networking

I've configured security groups for my production servers, mainly to allow the ports required by the applications.

One thing I wanted to understand better was how security groups work with the rest of the network. They control allowed inbound and outbound traffic, and they're stateful, so response traffic for an allowed connection is automatically permitted.

But opening a port isn't enough to make an application accessible. I also need to check the VPC, subnet, route table, internet gateway, and public IP configuration. The application itself must be running and listening on the expected port.

## 3. Elastic IP

I've assigned Elastic IPs to servers that need a fixed public IP address.

An automatically assigned public IP can change when an instance is stopped and started. An Elastic IP stays allocated to my AWS account until I release it, so I can associate it with a supported resource when needed.

I also looked at how this relates to domain configuration. The domain and IP address are separate things. DNS records determine where the domain points, so having an Elastic IP doesn't automatically configure the domain.

Elastic IPs can also cost money, which is something to keep in mind when deciding whether a static IP is necessary.

## 4. EBS Volumes and Snapshots

I've attached EBS volumes to EC2 instances when additional storage was needed.

I wanted to understand what happens after attaching a volume. On Linux, I first need to identify the disk. Depending on the situation, I may need to create a filesystem and mount it to a directory before I can store files on it.

These are the commands I use to check storage:

- `lsblk` — lists the available disks and block devices.
- `lsblk -f` — shows filesystem information.
- `df -h` — shows disk space usage.

I also looked at what happens when an EBS volume is resized. Increasing its size doesn't always make the extra space available to the filesystem straight away. The partition or filesystem may need to be extended as well.

Snapshots are useful for keeping a point-in-time backup of a volume. I can use one to create another volume when recovering data or setting up storage again.

Before terminating an instance, I need to check its volume settings. Some volumes are deleted along with the instance, while others remain in the account.

## 5. IAM Roles and S3 Access

I've used IAM roles when applications running on EC2 need to access S3.

Instead of keeping AWS access keys in the application code, I can attach an IAM role to the instance. The application can then obtain temporary credentials through the Instance Metadata Service.

The permissions in the role decide what the application can access. I should only grant the permissions it actually needs.

I also covered instance metadata and IMDSv2. The metadata service provides information about the instance, and IMDSv2 uses a session token when making metadata requests.

## 6. CloudWatch and Server Troubleshooting

I've used CloudWatch to monitor EC2 instances and set up alarms.

It provides metrics such as CPU utilisation and network traffic. I can configure alarms to notify me when a metric crosses a threshold.

One thing worth remembering is that EC2 monitoring doesn't automatically provide every operating system metric. If I want to monitor memory or filesystem usage, I generally need to configure the CloudWatch Agent.

I also need to remember that an EC2 instance can be running even when the application isn't working. When that happens, I check the server instead of relying only on the instance status.

These commands are useful when investigating a problem:

- `top` — check running processes and resource usage.
- `free -h` — check memory usage.
- `df -h` — check available disk space.
- `ss -tulpn` — check listening ports and sockets.
- `systemctl status` — check whether a service is running.

If an application isn't responding, I can use these commands to check the process, memory, disk space, and listening ports. I also need to look at the application logs and AWS network configuration to narrow down the issue.

## 7. Deploying Applications and Configuring Domains

I've deployed applications on EC2, installed dependencies, and configured domains to make them accessible.

During deployment, I need to make sure the required runtime is installed and the application starts correctly. I also check which port the application uses and whether the necessary traffic is allowed through the security group.

I've worked with Nginx as a reverse proxy. It receives requests from users and forwards them to the application running on the server.

Domain configuration is another part of the setup. DNS records need to point to the correct endpoint, and HTTPS needs a properly configured TLS certificate.

I also revisited the difference between stopping and terminating an instance. Stopping shuts down the server but generally keeps its EBS-backed storage. Terminating removes the instance permanently, and its root volume may also be deleted depending on the settings.

Stopping an instance doesn't mean every AWS charge stops. For example, EBS volumes that remain allocated can continue to incur charges.

## Final Notes

Most of these topics came up in my production work. I wanted to understand them better instead of just remembering which settings to change.

These notes cover what I've gone through so far. I'll add more as I work with EC2 and come across configurations or issues that are worth documenting.