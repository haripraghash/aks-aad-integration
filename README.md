# aks-aad-integration
Steps involved in creating an AKS cluster integrated with Azure Active DIrectory(AAD)
# Prequisites

1. Azure Subcription
2. Access to Azure AD and permissions
3. AZ CLI installed
4. Kubectl installed


# Create an Azure Active Directory App Registration - For AKS server
Integrating AKS with AAD involves creating 2 AAD app registrations. One representing the server and another one for the client.

`az login`

`AAD_AKS_SERVER_APP="AKSAADServerApp"`

`#Create server app registration`

`az ad app create --display-name=$AAD_AKS_SERVER_APP --reply-urls "https://$AAD_AKS_SERVER_APP"`

`#Set the groupMembershipClaims value to All in manifest`

`az ad app update --id $SERVER_APP_ID --set groupMembershipClaims=All`


Make a note of the app id returned above
`
SERVER_APP_ID=<app id of the server app>
#Create a secret
az ad app credential reset --id $SERVER_APP_ID

#Make a note of the password in the output returned above
SERVER_APP_PASSWORD=<password from above>

`
