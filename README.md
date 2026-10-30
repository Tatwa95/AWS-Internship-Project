# AWS VPC Peering Lab

A CloudFormation project for a private IPv4 connection between two AWS VPCs, with a public bastion for SSH administration. It follows the EWB Courses VPC Peering workbook and uses phased deployment so the pre-peering baseline can be tested first.

## Architecture

- VPC A: `10.10.0.0/16`
  - Public subnet: `10.10.1.0/24` with a bastion at `10.10.1.10`
  - Private subnet: `10.10.2.0/24` with App-A at `10.10.2.10`
  - Internet gateway and separate public/private route tables
- VPC B: `10.20.0.0/16`
  - Private subnet: `10.20.1.0/24` with App-B at `10.20.1.10`
  - No internet gateway
- VPC peering and forward/return routes are created in Phase 2.
- Security groups restrict SSH to the current administrator `/32` or bastion `/32`, and ICMP Echo Request to the peer host `/32`.
- Amazon Linux 2023, small `t3.micro` instances, no NAT gateway.

## Deploy with CloudFormation

1. Use a dedicated lab account and choose one AWS Region (the template defaults to `ap-south-1`). Check current pricing and account budget before deployment.
2. Create an EC2 key pair named `ewb-lab` (or provide another existing key-pair name). Download and protect the private `.pem` file locally. Never commit it to GitHub.
3. Find your current public IPv4 and provide it as `x.x.x.x/32` for `BastionSshCidr`.
4. In CloudFormation, create a stack using `stack.yaml`. Set `StudentName`, `BastionSshCidr`, and `KeyName`; leave `EnablePeering` as `false` for Phase 1.
5. Wait for stack creation. SSH to the bastion, then App-A through the bastion. From App-A run `ping -c 4 10.20.1.10` and record the expected pre-peering failure.
6. Update the same stack and set `EnablePeering` to `true`. Wait for the update to complete. Verify the peering connection and routes, then run pings in both directions between the private IPs. Verify App-B SSH through the bastion.
7. For the isolation exercise, temporarily remove only App-B's inbound ICMP Echo Request rule from `10.10.2.10/32`, verify App-A ping fails, then restore the exact rule and verify recovery. Keep SSH allowed throughout.

Do not use the private IPs as ports or expose SSH/ICMP to `0.0.0.0/0`. The private hosts should have no public IPv4 addresses.

## Cleanup

Delete the CloudFormation stack after saving coursework evidence. Verify the stack is `DELETE_COMPLETE`, the lab instances are terminated, and no lab volumes, public IP allocations, VPCs, peering connections, or other tagged resources remain. Review the account's billing page because usage reporting can lag. Keep or remove the separate key pair according to your own key-management needs.

## Recorded validation

The student deployment was completed in phases and then cleaned up. During the recorded baseline check, the peering resource already existed but both inter-VPC routes were removed; App-A sent four ping packets to App-B and received none. This confirms that traffic failed without routes, but it is not a pre-peering baseline. After restoring the routes, both ping directions received four replies with no loss. SSH to the bastion and both private hosts succeeded. During the controlled rule-removal check, App-A ping failed while App-B's Echo Request rule was absent and recovered after restoration. The AWS stack is now deleted; these results describe the completed historical run, not a currently running lab.

## Repository safety

This public source copy contains no private keys, local SSH config, AWS account identifiers, or personal resource IDs. Before publishing any report or screenshots separately, remove credentials, local file paths, account IDs, and any IP addresses you do not want public. Do not commit `*.pem`, `*.tfstate*`, `*.tfplan`, `.terraform/`, `.env`, `ssh_config`, or `known_hosts`.

