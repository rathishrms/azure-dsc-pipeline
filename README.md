## Description
The contents of this repo provide an example of how to insert Azure DSC workflow into a pipeline, end-to-end.

This also uses DSC Composite module, to compartmentalize your DSC code into managable sections.

**Note**: on the initial run, this set of pipelines will likely take over 30 minutes to complete.

## Prerequisites
- An Azure Subscription in which to deploy resources
- An Azure KeyVault that will be used to generate certificates
- An Azure Storage Account with a container, to store composite module zip
- An Azure DevOps organization you can create pipelines in
- An Azure Service Principal with the following RBAC: (so that it can itself create new service principals)
    - must be "Application Administrator" on the Azure AD tenant
    - must be "Owner" on the subscription
    - must have appropriate rights to an access policy on the KeyVault to generate and retrieve Certificates
- An Azure DevOps Service Connection linked to the Service Principal above

## Deployment
Assuming you're connecting to a GitHub repository containing this code:
- Populate the "dsc_parameters.ps1" file with values as you desire
- Within Azure DevOps, create a new Pipeline, and link it to your GitHub (creating a service connection using OATH login)
- Select an existing yaml file to create the pipeline from
    - Do this twice for "ModuleDeploy-pipeline.yml" and "azure-pipelines.yml"
    - Don't forget to rename your pipelines in DevOps portal, to something meaningful
- Manually run the "ModuleDeploy-pipeline.yml" pipeline

## Results
- ModuleDeploy-pipeline.yml pipeline runs and
    - creates automation account if it doesn't exist
    - uploads DSC composite module to automation account
- azure-pipelines.yml pipeline runs and:
    - creates automation account if it doesn't exist
    - imports/updates Az.Accounts module
    - imports/updates remaining modules identified in parameters
    - creates new automation runas account (and required service principal) if it doesn't exist
    - performs a 'first-time' run of the "Update-AutomationAzureModulesForAccount" runbook (because automation account is created with out-of-date default modules)
    - imports DSC configuration
    - compiles DSC configuration against configuration data