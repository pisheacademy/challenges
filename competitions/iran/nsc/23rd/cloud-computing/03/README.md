# Challenge 03 — Know every way in, and every way out

This is the third challenge. The rules are the same as before. The difference is that this time all three questions are about the network: the VPC, and how a packet gets in and out of it.

In the first challenge you told me what a Region and an Availability Zone are. In the second, you decided who is allowed to do what. Now I want to see where the packets go. Draw a network before you build it. Read a set of rules and know which packet gets through. Give a private server a way out without giving the internet a way in. A server with a perfect IAM role is still in trouble if its database port is open to the world. An app with every permission it needs still does nothing if its packets have no route out of the subnet. The network does exactly what you configured, not what you meant.

Three questions. Write the answers in Persian, on paper, in your own words. A short answer that is yours beats a long one you copied from a page. If I can see you transcribed a document instead of explaining it, I will treat it as not yours.

At the end of each answer, on the same page, write the references you actually used. A doc title and a link is enough. If you used nothing, write that. Do not leave the bottom of the answer empty.

## How you submit

Write it by hand. Take a photo of each answer. The photo has to be readable: light on the page, the whole page in the frame, your handwriting big enough that I can read the Persian and the references without zooming until it breaks.

Open a pull request with those photos.

In this folder, `competitions/iran/nsc/23rd/cloud-computing/03/`, create a new folder. The folder name is your GitHub username, the same spelling, the same caps. Put the photos in that folder. Name them by question so I do not have to guess:

- `01.jpg` for the first answer
- `02.jpg` for the second
- `03.jpg` for the third

If one answer needs a second page, name it `01-2.jpg`, and so on. The references stay at the end of that answer, not in a separate file.

If your username is `ali-cloud`, I should see something like:

`competitions/iran/nsc/23rd/cloud-computing/03/ali-cloud/01.jpg`

One person, one folder, one pull request. Do not edit this README. Do not put files in someone else's folder. Do not type the answer into GitHub. I want the paper.

## 1. Draw it before you build it

You are building the network for a small web app in one Region. There is a load balancer that the internet talks to, two app servers behind it, and a database. Only the load balancer faces the internet. Nobody on the internet should be able to reach the app servers or the database directly. And you know the rule from the first challenge: one Availability Zone can have a bad day, and the app has to stay up.

One more fact. The office network already uses `10.0.0.0/16`, and next year someone will connect the office to this VPC with a VPN.

Pick a CIDR block for the VPC, and tell me why that one. Then draw the VPC: every subnet, the Availability Zone it lives in, its CIDR, and what runs in it. Mark which subnets are public and which are private, and put the internet gateway where it belongs. Draw it by hand, on the paper. Boxes and lines are enough, but label everything. I should not have to guess what a box is.

Then two short ones:

- A subnet is not public because someone named it `public-a`. What actually makes a subnet public? And what else does a server in that subnet need before someone on the internet can reach it?
- The smallest subnet AWS lets you create is a `/28`. That is 16 addresses. How many of them can you give to your own resources, and where did the rest go?

## 2. Follow the packet

Someone else built this VPC. Its CIDR is `10.20.0.0/16`. It has two subnets, both in the same Availability Zone, and each subnet has its own route table.

| Subnet | CIDR | Network ACL |
|---|---|---|
| `public-a` | `10.20.1.0/24` | the default one, which allows all traffic in and out |
| `private-a` | `10.20.11.0/24` | its own, shown below |

Route table of `public-a`:

| Destination | Target |
|---|---|
| `10.20.0.0/16` | `local` |
| `0.0.0.0/0` | `igw-01` |

Route table of `private-a`:

| Destination | Target |
|---|---|
| `10.20.0.0/16` | `local` |

Network ACL of `private-a`:

| Direction | Rule | Protocol | Ports | Source or destination | Action |
|---|---|---|---|---|---|
| Inbound | 100 | TCP | 5432 | `10.20.1.0/24` | Allow |
| Inbound | `*` | All | All | `0.0.0.0/0` | Deny |
| Outbound | 100 | TCP | 5432 | `10.20.1.0/24` | Allow |
| Outbound | `*` | All | All | `0.0.0.0/0` | Deny |

Three servers run in it. `db-1` runs PostgreSQL on port 5432.

| Server | Subnet | Private IP | Public IP | Security group |
|---|---|---|---|---|
| `web-1` | `public-a` | `10.20.1.10` | `198.51.100.20` | `sg-web` |
| `report-1` | `public-a` | `10.20.1.20` | none | `sg-report` |
| `db-1` | `private-a` | `10.20.11.5` | none | `sg-db` |

The security groups. All three keep the default outbound rule, which allows all traffic to anywhere.

| Security group | Inbound rules |
|---|---|
| `sg-web` | TCP 443 from `0.0.0.0/0` |
| `sg-report` | none |
| `sg-db` | TCP 5432 from `sg-web` |

For each case, tell me yes or no. If it is a yes, tell me what lets it through. If it is a no, tell me where it stops. If more than one thing would stop it, name every one of them.

1. Someone on the internet connects to `198.51.100.20` on port 443.
2. `report-1` connects to `db-1` on port 5432.
3. `web-1` connects to `db-1` on port 5432.
4. `db-1` runs `apt update`, which downloads package lists from the Ubuntu mirrors on the internet.

Same as last time: the yes or no is the easy part. The reason is where the points are.

The app needs the third case to work. Make it work with the smallest change you can, and write the exact rule: direction, rule number, protocol, ports, source or destination, allow or deny.

Then tell me, in two or three sentences of your own, how a security group and a network ACL are different. I want more than "one is for the server and one is for the subnet".

## 3. Private, but not cut off

Bring back the app from the second challenge, the one that uploads files to S3. It uses a role now, no keys in a config file. It runs on the two app servers from question 1, in the private subnets, one server in each Availability Zone.

The servers run Ubuntu, and they still need to reach outside the VPC for two jobs: uploading files to an S3 bucket in the same Region, and downloading security updates from the Ubuntu mirrors on the internet. Assume the private route tables have only the `local` route for now, so the servers can do neither.

A teammate says: "Easy. Move them to the public subnet and give them public IPs." Tell me why we are not doing that.

You have two other ways to build the road: a NAT gateway, and a gateway VPC endpoint for S3. For each one, tell me where it sits, what it adds to the route table, and what you pay for it. Then decide. Which road do the uploads take, and which road do the updates take? Why not send everything down one road?

When you are done, write the route table of one private subnet. A packet to S3 and a packet to an Ubuntu mirror leave the same server. Tell me which route each one takes, and how the route table picks it.

Last part. You built one NAT gateway, in zone A, and the private subnets in both zones send their internet traffic to it. Zone A has a bad day. What happens to the server in zone B? Which of its two jobs keeps working, and which one stops? Then change the design so that losing one zone does not cut off the other.
