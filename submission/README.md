# EWB AWS VPC Peering - Submission Evidence

This folder contains the project report and supporting evidence for the CloudFormation lab in the repository root. The report covers the two-VPC architecture, private peering routes, bastion-based SSH, ICMP verification, workbook questions, and cleanup.

## Files

- `TatwaShahare_EWB_VPC_Peering_Report.pdf` - formatted report with six AWS Console screenshots.
- `submission-report.md` - editable report.
- `verification-results.txt` - captured SSH and ping results and cleanup record.
- `evidence/` - six authentic AWS Console screenshots as JPEGs, with AWS account identifiers and public IP addresses redacted.
- `../stack.yaml` - CloudFormation source template.

## Evidence notes

The route-disabled baseline was recorded while the peering connection existed and both private inter-VPC routes were removed. This demonstrates that routing is required; it is not a pre-peering baseline. With routes restored, both private ping directions succeeded. A controlled removal of App-B's ICMP Echo Request rule caused ping to fail, and restoring the rule restored connectivity. SSH through the bastion succeeded. The temporary stack was deleted after testing.

The AWS Console captures are genuine screenshots with personal/account-specific identifiers and public IPs obscured for public sharing. The original terminal screenshot was unavailable; the report does not represent text output as a terminal image. The PDF workbook itself is not included.

## Reproduce

Follow the root [`README.md`](../README.md) to deploy and test the lab. Review current AWS pricing and clean up the stack after use. Never publish private keys, SSH configuration, AWS credentials, or unredacted account screenshots.

