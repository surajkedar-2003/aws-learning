# AWS VPC

I started learning VPC to understand how networking works in AWS. I wanted to know how EC2 instances communicate with each other, how resources get internet access, and how we keep application servers and databases private.

I began with IP addressing and CIDR because understanding these concepts makes the rest of VPC networking easier. After that, I learned about subnets, route tables, Internet Gateways, NAT Gateways, Security Groups, and Network ACLs.

## VPC Architecture

This is a basic architecture of how I understand networking for a web application in AWS.

![architecture diagram](<AWS VPC Web Application Architecture Diagram.png>)
The public subnet contains resources that need to receive public traffic or provide outbound internet access. The private application subnet contains the backend, while the database stays in a private subnet.

The route tables control where traffic goes. Security Groups and Network ACLs control which traffic is allowed.

*Note: This is a conceptual diagram. The route tables and security rules still need to be configured, and the load balancer and database require their own appropriate settings.*

## What I Learned

### 1. VPC, IP Addressing and CIDR

A VPC is a logically isolated virtual network in AWS. When creating one, we choose an IP address range and divide it into subnets according to the needs of our application.

I learned that CIDR notation determines the size of an IP range. For IPv4, the total number of addresses is calculated using `2^(32 - prefix length)`.

| CIDR | Total IPv4 addresses | AWS-usable addresses per subnet |
|---|---:|---:|
| `10.0.0.0/16` | 65,536 | Depends on the subnet size |
| `10.0.1.0/24` | 256 | 251 |
| `10.0.1.0/28` | 16 | 11 |

AWS reserves five IPv4 addresses in each subnet. For example, a `/24` subnet has 256 total addresses, but only 251 are available for use.

I also learned that subnets within the same VPC cannot have overlapping IPv4 CIDR ranges.

### 2. Public and Private Subnets

A subnet is considered public when its associated route table has a route to an Internet Gateway. For example:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | Local |
| `0.0.0.0/0` | Internet Gateway |

A private subnet does not have a direct route to an Internet Gateway. It can still access the internet through a NAT Gateway if the required routing is configured.

A public route alone does not make an EC2 instance accessible from the internet. Direct public IPv4 connectivity also requires suitable public IP addressing and security rules.

### 3. Route Tables and Routes

A route table tells AWS where to send network traffic based on its destination. Every VPC has a local route that allows communication within its VPC address range, subject to security and other networking rules.

I also learned about the **longest prefix match** rule: when multiple routes match a destination, AWS selects the most specific matching route.

For example, consider these routes:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | Local |
| `10.0.2.0/24` | VPC peering connection |
| `0.0.0.0/0` | Internet Gateway |

If traffic is going to `10.0.2.15`, the `/24` route is more specific than the `/16` route, so AWS selects the VPC peering route, assuming it is a valid route for that destination.

Here is an example of how route tables are used in an application:

| Subnet | Route table | Destination | Target |
|---|---|---|---|
| Public | Public RT | `10.0.0.0/16` | Local |
| Public | Public RT | `0.0.0.0/0` | Internet Gateway |
| Private App | Private RT | `10.0.0.0/16` | Local |
| Private App | Private RT | `0.0.0.0/0` | NAT Gateway |
| Private DB | Database RT | `10.0.0.0/16` | Local |

These are example routes. The database subnet does not need a default route to the internet unless its requirements call for one.

### 4. Internet Gateway and NAT Gateway

An Internet Gateway is attached to a VPC and enables internet connectivity for resources with the required addressing, routes, and security configuration.

A NAT Gateway is placed in a subnet. A public NAT Gateway normally uses an Elastic IP and allows private resources to initiate outbound internet connections without allowing unsolicited inbound connections through the NAT Gateway.

The typical routing is:

- Public subnet: `0.0.0.0/0` points to the Internet Gateway.
- Private application subnet: `0.0.0.0/0` points to the NAT Gateway.
- NAT Gateway's public subnet: has a route to the Internet Gateway.

NAT Gateways have a cost, so they should be removed when they are no longer needed for practice.

### 5. Security Groups and Network ACLs

I learned that AWS provides two different layers of network traffic filtering.

| Feature | Security Group | Network ACL |
|---|---|---|
| Applied at | Resource network interface | Subnet |
| Traffic handling | Stateful | Stateless |
| Rules | Allow rules only | Allow and deny rules |
| Rule evaluation | All applicable rules are considered together | First matching numbered rule |
| Response traffic | Automatically allowed for tracked connections | Must be allowed by the relevant outbound rules |

For example, a backend Security Group can allow traffic on the application port only from the load balancer's Security Group. The database Security Group can allow database traffic only from the backend Security Group.

This helps limit access instead of allowing every resource to communicate freely.

## Key Takeaways

- A VPC defines the network address range, while subnets divide it into smaller networks.
- A `/24` subnet has 256 total IPv4 addresses, of which AWS reserves five.
- Route tables determine traffic destinations; the most specific matching route takes priority.
- Internet Gateways provide a path to the internet, while NAT Gateways support outbound internet access for private resources.
- Security Groups are stateful and resource-level; Network ACLs are stateless and subnet-level.

## Hands-on / Next Steps

Today, I focused on understanding the theory. I have not completed the practical labs yet.

- [ ] Create a custom VPC with a suitable CIDR range.
- [ ] Create public and private subnets in different Availability Zones.
- [ ] Configure an Internet Gateway and public route table.
- [ ] Launch an EC2 instance in a public subnet and test connectivity.
- [ ] Create a NAT Gateway and configure a private subnet's route table.
- [ ] Test outbound internet access from a private EC2 instance.
- [ ] Configure Security Groups and test allowed and blocked connections.
- [ ] Document the results with screenshots and troubleshooting notes.

After completing each lab, I will update this checklist with the actual results and add screenshots where they help explain the configuration.
