# GitHub Actions Workflows for Terraform

This is a sample repository that shows how to build GitHub Actions workflows to manage infrastructure with Terraform. 

## Architecture

<img width="2159" alt="GitHub Actions CICD for Terraform" src="https://user-images.githubusercontent.com/1248896/189254453-439dd558-fc6c-4377-b01c-d5e54cc49403.png">

## Dataflow

1. Create a new branch and check in the needed Terraform code modifications.
2. Create a Pull Request (PR) in GitHub once you're ready to merge your changes into your environment.
3. A GitHub Actions workflow will trigger to ensure your code is well formatted. In addition, a Terraform Plan analysis should run to generate a preview of the changes that will happen in your Azure environment.
4. Once appropriately reviewed, the PR can be merged into your main branch.
5. Another GitHub Actions workflow will trigger from the main branch and execute the changes using Terraform.
6. A regularly scheduled GitHub Action workflow should also run to look for any configuration drift in your environment and create a new issue if changes are detected.


## Workflows

1. [**Terraform Unit Test**](.github/workflows/tf-validate.yml)

    This workflow is designed to be run on every commit and is composed of a set of unit tests on the infrastructure code. It runs [terraform fmt]( https://www.terraform.io/cli/commands/fmt) to ensure the code is properly linted and follows terraform best practices. Next it performs [terraform validate](https://www.terraform.io/cli/commands/validate) to check that the code is syntactically correct and internally consistent.

2. [**Terraform Plan / Apply**](.github/workflows/tf-plan-apply.yml)

    This workflow runs on every pull request and on each commit to the main branch. The plan stage of the workflow is used to understand the impact of the IaC changes on the Azure environment by running [terraform plan](https://www.terraform.io/cli/commands/plan). This report is then attached to the PR for easy review. The apply stage runs after the plan when the workflow is triggered by a push to the main branch. This stage will take the plan document and [apply](https://www.terraform.io/cli/commands/apply) the changes after a manual review has signed off if there are any pending changes to the environment.

3. [**Terraform Drift Detection**](.github/workflows/tf-drift.yml)

    This workflow runs on a periodic basis to scan your environment for any configuration drift (i.e. changes made outside of terraform). If any drift is detected a GitHub Issue is raised to alert the maintainers of the project.

## Getting Started

To use these workflows in your environment several prerequiste steps are required:

1. **Create GitHub Environments**

    The workflows utilizes GitHub Environments to store the azure identity information and setup an appoval process for deployments. Create 2 environments: `production-readonly` and `production-readwrite` by following these [insturctions](https://docs.github.com/actions/deployment/targeting-different-environments/using-environments-for-deployment#creating-an-environment). On the `production-readwrite` environment setup a protection rule and add any required approvers you want that need to sign off on production deployments. You can also limit the environment to your main branch. Detailed instructions can be found [here](https://docs.github.com/actions/deployment/targeting-different-environments/using-environments-for-deployment#creating-an-environment).

2. **Setup Azure Identity**: 

    An Azure Active Directory application is required that has permissions to deploy within your Azure subscription. Create a separate application for the `production-readonly` and `production-readwrite` environments and give them the appropriate permissions in your Azure subscription. Next setup the federated credentials to allow the GitHub environments to utilize the identity using OIDC. See the [Azure documentation](https://docs.microsoft.com/azure/developer/github/connect-from-azure?tabs=azure-portal%2Clinux#use-the-azure-login-action-with-openid-connect) for detailed instructions. Make sure to set the Enitity Type to `Environment` and use the appropriate environment name for the GitHub name.


3. **Add GitHub Secrets**

    For each GitHub Environment create the following secrets for the respective Azure Identity:

    - _AZURE_CLIENT_ID_ : The application (client) ID of the app registration in Azure
    - _AZURE_TENANT_ID_ : The tenant ID of Azure Active Directory where the app registration is defined.
    - _AZURE_SUBSCRIPTION_ID_ : The subscription ID where the app registration is defined.

    Instuructions to add the secrets to the environment can be found [here](https://docs.github.com/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-an-environment).
    
4. **Configure Terraform State Location**

    Terraform utilizes a [state file](https://www.terraform.io/language/state) to store information about the current state of your managed infrastructure and associated configuration. This file will need to be persisted between different runs of the workflow. The recommended approach is to store this file within an Azure Storage Account or other similiar remote backend. The [Terraform backend block]() `TODO: add link` will need to be configured to point to an appropriate location where your workflow has permissions. Normally this location would be created manually or via a separate workflow.

5. **Activate the Workflows**

    In each workflow file uncomment the top trigger section to enable the workflows to run automatically.
