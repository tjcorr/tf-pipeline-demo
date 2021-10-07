# tf-pipeline-demo

This is a sample repository that shows how to build a GitHub Actions workflow to manage infrastructure with Terraform. It includes:

- A workflow that runs terraform fmt and validate on every commit
- A workflow that runs terraform plan/apply
 - Runs plans on PRs against main
 - Writes the output of the plan as a comment on the PR
 - For commits to main run a plan follow by apply if there are any changes. 
 - Require approvals before running terraform apply
