# AWS VPC Peering Lab - Deployment and Verification

**Student:** Tatwa Shahare  

**Region:** Asia Pacific (Mumbai), `ap-south-1`  

**Deployment:** Temporary CloudFormation stack `ewb-vpc-peering-lab`, created 2026-10-31 01:37 IST and deleted 2026-10-31 01:48 IST  

**Final AWS status:** `DELETE_COMPLETE`; lab stack resources have been removed

## Current status

The temporary stack was deployed, tested, and deleted. To capture a route-disabled baseline, both private inter-VPC routes were temporarily removed while the peering connection existed. App-A pinged App-B at `10.20.1.10`: 4 packets sent, 0 received (100% loss). After restoring both routes, reciprocal private-IP pings returned 4/4 replies. A controlled App-B ICMP rule removal caused ping to fail; restoring the rule returned 4/4 replies. SSH through the bastion worked for both private hosts.

## 1. Architecture

```mermaid

flowchart LR

  Laptop[Student laptop]

  subgraph A[VPC A 10.10.0.0/16]

    Bastion[Bastion 10.10.1.10 public IPv4 [redacted public IPv4]]

    AppA[App-A 10.10.2.10 private]

    IGW[Internet Gateway]

  end

  subgraph B[VPC B 10.20.0.0/16]

    AppB[App-B 10.20.1.10 private]

  end

  Laptop -->|SSH TCP 22 from [redacted public IPv4]/32| Bastion

  Bastion -->|ProxyJump SSH| AppA

  Bastion -->|ProxyJump SSH over peering| AppB

  AppA <-->|ICMP over VPC peering| AppB

  Bastion --> IGW

```

VPC A and VPC B use non-overlapping CIDRs. The bastion is the only EC2 instance with a public IPv4 address. App-A and App-B use private addresses; the peering connection and routes carry traffic between the VPCs. VPC B has no internet gateway. The bastion SSH ingress rule was limited to the public IPv4 detected for the student's network at deployment time (`[redacted public IPv4]/32`).

## 2. Deployed resources

| Resource | ID / address | Result |

|---|---|---|

| CloudFormation stack | `ewb-vpc-peering-lab` | `DELETE_COMPLETE` |

| VPC A (`10.10.0.0/16`) | `[redacted AWS resource ID]` | Temporary; removed after tests |

| VPC B (`10.20.0.0/16`) | `[redacted AWS resource ID]` | Temporary; removed after tests |

| Bastion | `[redacted AWS resource ID]`, private `10.10.1.10`, public `[redacted public IPv4]` | SSH verified, then terminated |

| App-A | `[redacted AWS resource ID]`, private `10.10.2.10` | SSH verified, then terminated |

| App-B | `[redacted AWS resource ID]`, private `10.20.1.10` | SSH verified, then terminated |

| VPC peering | `[redacted AWS resource ID]` | Active during tests; removed by stack cleanup |

| Public subnet A | `[redacted AWS resource ID]` / `10.10.1.0/24` | Temporary; removed after tests |

| Private subnet A | `[redacted AWS resource ID]` / `10.10.2.0/24` | Temporary; removed after tests |

| Private subnet B | `[redacted AWS resource ID]` / `10.20.1.0/24` | Temporary; removed after tests |

| Route table public A | `[redacted AWS resource ID]` | Temporary; removed after tests |

| Route table private A | `[redacted AWS resource ID]` | Temporary; removed after tests |

| Route table private B | `[redacted AWS resource ID]` | Temporary; removed after tests |

| App-A security group | `[redacted AWS resource ID]` | Temporary; removed after tests |

| App-B security group | `[redacted AWS resource ID]` | Temporary; removed after tests |

| Bastion security group | `[redacted AWS resource ID]` | Temporary; removed after tests |

| SSH key pair | `standalone-lab-key` | Imported key pair; private key remains local |

The stack also created the three subnets, three route tables and subnet associations, one Internet Gateway for VPC A, three security groups, and routes for internet access from the public subnet and peering traffic in both VPCs (plus VPC A's public route table).

## 3. Routes and security

| Route table | Associated subnet | Required routes |

|---|---|---|

| Public A | `10.10.1.0/24` | Local VPC route; `0.0.0.0/0` to the VPC A Internet Gateway; `10.20.0.0/16` to `[redacted AWS resource ID]` |

| Private A | `10.10.2.0/24` | Local VPC route; `10.20.0.0/16` to `[redacted AWS resource ID]` |

| Private B | `10.20.1.0/24` | Local VPC route; `10.10.0.0/16` to `[redacted AWS resource ID]` |

- Bastion: inbound TCP 22 only from `[redacted public IPv4]/32`.

- App-A: inbound TCP 22 from bastion `10.10.1.10/32`; ICMP Echo request from App-B `10.20.1.10/32`.

- App-B: inbound TCP 22 from bastion `10.10.1.10/32`; ICMP Echo request from App-A `10.10.2.10/32`.

- No inbound SSH or ICMP rule is open to `0.0.0.0/0`.

## 4. Verification performed

The locally stored SSH private key authenticated successfully; the key itself is not included. The bastion and both private hosts were reachable by SSH using ProxyJump. With both private peer routes removed, App-A to App-B ping failed. After restoring routes, App-A to App-B and App-B to App-A each received four replies (0% loss). See `verification-results.txt` for captured results.

A ping initiated from the bastion directly to the application hosts received no replies. That is expected with the configured host-specific peer ICMP rules; the successful peer-to-peer tests are the VPC peering check.

**Route-disabled baseline:** the VPC peering resource existed, but both private inter-VPC routes were temporarily removed. App-A sent four pings to App-B and received zero replies. This confirms route dependence, not a pre-creation/no-peering baseline.

### ICMP isolation exercise

### Controlled ICMP failure and recovery

Temporarily removed only App-B’s inbound ICMP Echo Request rule from `10.10.2.10/32`, leaving SSH TCP 22 from the bastion intact. App-A’s four pings to App-B received no replies (100% loss). Restored the Echo Request rule and reran the ping; all four replies returned. AWS confirmed the rule is present after restoration. The restored rule ID is `[redacted AWS resource ID]`.

## 5. AWS Console screenshots

Six authentic AWS Console screenshots captured during this temporary deployment and cleanup are included in this folder:

- `aws-console-stack-outputs.jpg` - live stack outputs and addresses.

- `aws-console-active-route.jpg` - active private peering route.

- `aws-console-icmp-restored.jpg` - App-B security group after restoring Echo Request.

- `aws-console-stack-cleanup.jpg` - `DELETE_COMPLETE` status.

- `aws-console-volumes-empty.jpg` - no matching lab volumes.

- `aws-console-active-stacks-empty.jpg` - no active CloudFormation stacks.

Terminal command output is preserved in `verification-results.txt`. An original terminal screenshot was unavailable, so none is represented as an original image.

## 6. Traffic path

App-A sends to App-B's private address `10.20.1.10`. The route table for App-A's subnet sends `10.20.0.0/16` across the peering connection. App-B's security group permits Echo requests from App-A's `/32`. The return route in VPC B sends `10.10.0.0/16` back through peering. For SSH, the laptop first reaches the bastion's public address; `ProxyJump` then connects from the bastion to the private host, so the private host sees the bastion's private IP as the SSH source.

## 7. Key and connection

The imported EC2 key pair is `standalone-lab-key`. Its private `.pem` file remains on the student computer and is not included here. The included `ssh_config` contains historical connection details and will not connect now because the stack was deleted.

From this project folder:

```powershell

ssh -F .\ssh_config ewb-bastion

ssh -F .\ssh_config ewb-app-a

ssh -F .\ssh_config ewb-app-b

```

## 8. Workbook questions

1. **Why are return routes needed?** Packets need a path back to the originating host; a forward route alone does not complete a two-way exchange.

2. **Why does App-B allow SSH from `10.10.1.10/32`?** ProxyJump opens the second SSH connection from the bastion, which uses that private source address.

3. **Why does active peering alone not ensure connectivity?** Both VPC route tables, security group rules, network ACLs, host addressing and instance state affect reachability.

4. **Can VPC B use VPC A's Internet Gateway through peering?** No. VPC peering is not transitive and does not share internet gateway access.

## 9. Cleanup and cost

The user set a $50 budget and asked for cleanup after completion. CloudFormation shows `ewb-vpc-peering-lab` as `DELETE_COMPLETE`, and the active stack list is empty. EC2 Volumes search for the lab name returned no matches. Stack-managed resources were removed by CloudFormation. The standalone key pair `standalone-lab-key` is not part of the stack and was intentionally retained. No claim is made here about checking every possible account-wide resource category after deletion.

## 10. Submission checklist

- Student name is Tatwa Shahare. Batch is omitted per the student request.

- This report contains the architecture, resource IDs, routes, security rules, recorded test results, explanations, and cleanup status.

- Six authentic AWS Console screenshots are included in the report and folder. Terminal command output is preserved in verification-results.txt; an original terminal screenshot was unavailable.

- 

- Cleanup evidence: CloudFormation showed `DELETE_COMPLETE`; the active stack list showed zero stacks; no matching lab volumes were found.


