# ⚡ EC2 Appliances

**Your AWS appliances. One tap away.**

It's 11 pm, you're nowhere near your laptop, and a server needs to go up, or down, *now*.
Opening the AWS console on a phone screen is no fun. **EC2 Appliances** puts your fleet in your pocket:
see what is running, tick the boxes, tap **Start** or **Stop**. Done.

### 📲 [Download the latest APK](https://github.com/vladislavsloboder/ec2-app-releases/releases/latest/download/ec2-appliances.apk)

*Android only. Older versions are on the [Releases](https://github.com/vladislavsloboder/ec2-app-releases/releases) page.*

---

## What you get

- 🖥️ **Your whole fleet at a glance:** every instance with its **name**, live **status** and a **checkbox**
  (`running`, `stopped`, `pending`, `stopping`)
- ✅ **Bulk control:** *Select all*, *Unselect all*, then **Start** or **Stop** every selected instance at once
- 🛡️ **No accidents:** each action asks for confirmation and lists exactly which instances it will touch
- 🔄 **Always current:** refreshes every 10 seconds, on pull-to-refresh and whenever you come back to the app
- 🌗 **Light and dark mode** that follow your phone

## How it works

```
 ┌──────────┐      HTTPS, signed on the phone      ┌───────────────┐
 │ Your     │ ───────────────────────────────────▶ │  AWS EC2 API  │
 │ phone    │ ◀─────────────────────────────────── │               │
 └──────────┘                                      └───────────────┘
        nothing in between: no backend, no proxy, no extra server
```

The app talks **directly to AWS**. There is no middleman server that could see, store or leak your data.

---

## Get started

### 1. Prepare AWS (one time)

**a) Create a dedicated IAM user**
*IAM → Users → Create user*, for example `ec2-phone-app`, **without** console access.

**b) Give it only what it needs**
This policy lets the app list all instances, but **start and stop only instances tagged `Type=appliance`**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:DescribeInstances",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": ["ec2:StartInstances", "ec2:StopInstances"],
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": { "StringEquals": { "aws:ResourceTag/Type": "appliance" } }
    }
  ]
}
```

**c) Create an access key**
On the user: *Security credentials → Create access key* (use case: *Application running outside AWS*).
Note the **Access Key ID** and the **Secret Access Key**. The secret is shown only once.

**d) Tag your instances**
- `Name`: the display name in the app (without it, the instance ID is shown)
- `Type` = `appliance`: required for Start and Stop under the policy above

### 2. Install the app

1. Open the download link above **on your Android phone** and download `ec2-appliances.apk`.
2. Open the file. When Android asks, **allow installation from this source**.
3. Tap **Install**.

New versions install the same way, normally as an update over the old one.

> 🔍 **Tip:** every APK on the Releases page shows a SHA-256 checksum. Compare it with your download,
> for example with `sha256sum ec2-appliances.apk`.

### 3. Connect and go

1. Open **EC2 Appliances** and enter your **Access Key ID**, **Secret Access Key** and **Region**
   (for example `eu-central-1`). Optionally add a **tag filter** such as `Type=appliance` to show only those instances.
2. Tap **Connect**. The app instantly checks your keys, region and permissions.
3. Tap instances to select them, then hit **Start** or **Stop** and confirm.
4. Watch the status flip from `pending` to `running` (or `stopping` to `stopped`) on its own.
5. Done for the day? Tap **Sign out** (top right). The secret key is wiped from the phone.

**If something goes wrong**

| Message | What it means |
|---|---|
| Invalid credentials | Access Key ID or secret is wrong, or the key was deactivated |
| No permission for this action | The IAM policy is missing, or the instance doesn't have the tag `Type=appliance` |
| AWS not reachable | No internet connection |
| Timeout | AWS did not answer within 15 seconds |

---

## 🔒 Security

**Built in**

- **Direct and encrypted:** the app only talks to the AWS EC2 API over HTTPS.
- **Your secret never leaves the phone:** every request is signed on the device (AWS Signature Version 4).
  AWS receives the signature, never the secret key itself.
- **Locked away:** credentials are kept in the operating system's secure storage
  (Android Keystore / iOS Keychain), not in plain files.
- **No server to hack:** the key exists only on each user's own phone, not at the developer and not in any cloud
  service of ours, because there is none.
- **Confirm before acting:** Start and Stop always show the affected instances first.
- **Sign out means gone:** the secret key is deleted from the device.
- **Clean repository:** this repo holds only finished builds, no source code and no credentials.
  The app itself contains no keys. Everybody enters their own.

**Your part**

- **Dedicated IAM user, minimal rights**, as in the policy above. Never put root or admin keys into the app.
- **Lost or sold your phone?** Deactivate and delete the access key in *IAM → Users → Security credentials* right away.
- **Rotate keys regularly.** AWS access keys do not expire on their own.
- **Lock your phone** with a PIN or biometrics. The app has no lock of its own, so anyone who can unlock the phone can use the app.
- **Download only from here** and compare the checksum.

**Know the limits**

- A **long-lived access key** is stored on the phone, which is why minimal permissions matter.
- The APK is signed with a standard developer key. It does **not** come from the Google Play Store,
  so Android shows its usual unknown-source warnings during installation.
- One AWS account and one region per sign-in.
- No push notifications. The status is refreshed by polling every 10 seconds.

---

*Built for people who run real servers and don't want to open a laptop to flip a switch.*

