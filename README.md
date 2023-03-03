# Custom Azure Policies for Azure Kubernetes Service

Contains custom Azure Policies to reflect the Gatekeeper [policy library](https://open-policy-agent.github.io/gatekeeper-library/website/)

## Constraint templates currently covered by Azure Policies

- [Allowed Repos](https://open-policy-agent.github.io/gatekeeper-library/website/validation/allowedrepos)
- [Automount Service Account Token for Pod](https://open-policy-agent.github.io/gatekeeper-library/website/validation/automount-serviceaccount-token)
- [Block Services with type LoadBalancer](https://open-policy-agent.github.io/gatekeeper-library/website/validation/block-loadbalancer-services)
- [Block NodePort](https://open-policy-agent.github.io/gatekeeper-library/website/validation/block-nodeport-services)
- [Container Limits](https://open-policy-agent.github.io/gatekeeper-library/website/validation/containerlimits/)
- [Container Requests](https://open-policy-agent.github.io/gatekeeper-library/website/validation/containerrequests)
- [External IPs](https://open-policy-agent.github.io/gatekeeper-library/website/validation/externalip) 
- [Replica Limits](https://open-policy-agent.github.io/gatekeeper-library/website/validation/replicalimits)
- [Required Labels](https://open-policy-agent.github.io/gatekeeper-library/website/validation/requiredlabels)
- [Required Probes](https://open-policy-agent.github.io/gatekeeper-library/website/validation/requiredprobes/)

## Important information

- 💡 As of now, defining custom Azure Policies for the AKS Policy Add-on is in public preview (22. Feb 2023) 
- 💡 The custom Azure Policies are pointing to constraint templates hosted on this repository. If you're not happy with this setup, you'd need to rehost the templates and rewrite the `templateInfo.URL` property in each policy
- 💡 It can take up to 30 minutes until an Azure Policy assignment or changes to an assignment become active, so be patient!

## Prerequisits

- Azure Policy Add-on for AKS ([installation instructions](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/policy-for-kubernetes#install-azure-policy-add-on-for-aks))

## Usage

Here is an example on how an Azure Policy definition can be created and assigned. 

### Create definitions
```PowerShell
New-AzPolicyDefinition -Name $(New-Guid) -Policy .\policies\container-requests.json
```

### Create assignment 

#### Option 1: Using the Azure Portal 

1. Open the `Policy` blade 
2. Navigate to `Definitions`
3. Select the desired definition (either filter by category `Kubernetes` or use the `Search` field) 
4. Click `Assign`
5. Select the subscription holding your AKS cluster, or narrow it down to overarching resource group 
6. Enter your desired parameters
7. Finish with `Review + Create`

#### Option 2: Using Azure PowerShell 

List assignable policies and make a note of the `Name` which is a GUID. 

```PowerShell
$subscription = Get-AzSubscription -SubscriptionName '<SUBSCRIPTION>'

Get-AzPolicyDefinition -Custom -SubscriptionId $subscription.Id | Select-Object -ExpandProperty 'Properties' Name | Select-Object Name,DisplayName,Description | Format-List

Name        : f0c59b01-a6ab-4c22-bfc7-f7786e03e545
Description : Requires containers to have memory and CPU requests set and constrains requests to be within the specified maximum values.
DisplayName : Gatekeeper Library: Container Requests
```

Define a hash-map holding the parameter for the Azure Policy. Here the resource group `rg-demo` holds the AKS cluster. 
```
$parameters = @{
    'effect' = 'Deny';
    'cpu' = '250m';
    'memory' = '64Mi';
}
$policyName = 'f0c59b01-a6ab-4c22-bfc7-f7786e03e545'
$resourceGroup 'rg-demo'

$policy = Get-AzPolicyDefinition -Name $policyName
$scope = $(Get-AzResourceGroup -Name $resourceGroup).ResourceId
$displayName = $(Get-AzPolicyDefinition -Name $policyName | Select-Object -ExpandProperty 'Properties).DisplayName

New-AzPolicyAssignment -PolicyDefinition $policy -PolicyParameterObject $parameters -scope $scope -Name 'gatekeeper-require-container-requests' -DisplayName $displayName
```

## Checklist for adding new policies

- In the Azure Policy definition, does *apiGroups* and *kinds* section match the examples provided by Gatekeeper? 
    - e.g. https://open-policy-agent.github.io/gatekeeper-library/website/validation/requiredprobes#examples
- In the Azure Policy definition, does the `templateInfo.URL` point to a valid constraint template?

## Further reading

- [Create and assign a custom policy definition](https://learn.microsoft.com/en-us/azure/aks/use-azure-policy#create-and-assign-a-custom-policy-definition)
- [Azure Policy definition structure](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/definition-structure)
