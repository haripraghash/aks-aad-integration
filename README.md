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

`#!/bin/bash


ENV_SHORT_NAME='dev'
AAD_SCOPE='Scope'
AAD_ROLE='Role'
SERVER_APP_NAME=aksaad${ENV_SHORT_NAME}serverapp
USER_READ_ALL_DELEGATED='a154be20-db9c-4678-8ab7-66f6cc099a59'
DIRECTORY_READ_ALL_DELEGATED='06da0dbc-49e2-44d2-8312-53f166ab848a'
DIRECTORY_READ_ALL_APPLICATION='7ab1d382-f21e-4acd-a863-ba3e13f7da61'
MICROSOFT_GRAPH_GUID='00000003-0000-0000-c000-000000000000'


az ad app create --reply-urls https://$SERVER_APP_NAME --display-name $SERVER_APP_NAME --password $SERVER_APP_PASSWORD
SERVER_APP_ID=$(az ad app list --output json | jq -r --arg appname $SERVER_APP_NAME '.[]| select(.displayName==$appname) |.appId')  
az ad app update --id $SERVER_APP_ID --set groupMembershipClaims=All
az ad app permission add --id $SERVER_APP_ID --api $MICROSOFT_GRAPH_GUID --api-permissions $USER_READ_ALL_DELEGATED=$AAD_SCOPE $DIRECTORY_READ_ALL_DELEGATED=$AAD_SCOPE $DIRECTORY_READ_ALL_APPLICATION=$AAD_ROLE

az ad app permission admin-consent --id $SERVER_APP_ID

#Client Application

CLIENT_APP_ID=$(az ad app create --display-name "${SERVER_APP_NAME}-Client" --native-app --reply-urls "https://${SERVER_APP_NAME}-Client" --query appId -o tsv)
SERVER_OAUTH_PERMISSION_ID=$(az ad app show --id $SERVER_APP_ID --query "oauth2Permissions[0].id" -o tsv)

az ad app permission add --id $CLIENT_APP_ID --api $SERVER_APP_ID --api-permissions $SERVER_OAUTH_PERMISSION_ID=Scope
#az ad app permission grant --id $CLIENT_APP_ID --api $SERVER_APP_ID
az ad app permission admin-consent --id $CLIENT_APP_ID

echo server_app_id = $SERVER_APP_ID 
echo server_app_secret = $SERVER_APP_PASSWORD
echo client_app_id = $CLIENT_APP_ID

az aks create -g aks-cluster-resgrp -n hari-aks --aad-server-app-id $SERVER_APP_ID --aad-server-app-secret $SERVER_APP_PASSWORD --aad-client-app-id $CLIENT_APP_ID --node-count 1 --location northeurope -k 1.15.7 -a monitoring -a http_application_routing
