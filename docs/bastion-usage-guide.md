# Bastion Service Usage Guide

Complete guide for using OCI Bastion service to securely connect to your private instances.

## Overview

**Bastion OCID:** `ocid1.bastion.oc1.iad.amaaaaaaofvcz2qaqfsdnnjspl5ws5vs4nqvrn62rt34pljrobufrahtru2a`  
**Status:** Creating (will be ACTIVE shortly)  
**Target Subnet:** Private-App-Subnet (10.0.10.0/24)  
**Max Session TTL:** 3 hours (10800 seconds)

## Prerequisites

1. **SSH Key Pair:** You need an SSH key pair on your local machine
2. **Target Instance:** A compute instance in the private subnet
3. **OCI CLI:** Configured and authenticated

## Step 1: Generate SSH Keys (If You Don't Have One)

```powershell
# Create .ssh directory if it doesn't exist
if (!(Test-Path ~\.ssh)) { New-Item -ItemType Directory -Path ~\.ssh }

# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -f ~\.ssh\oci_bastion_key -C "your-email@example.com"

# This creates two files:
# - ~\.ssh\oci_bastion_key (private key)
# - ~\.ssh\oci_bastion_key.pub (public key)
```

## Step 2: Wait for Bastion to be ACTIVE

Check the bastion status:

```powershell
$bastionId = "ocid1.bastion.oc1.iad.amaaaaaaofvcz2qaqfsdnnjspl5ws5vs4nqvrn62rt34pljrobufrahtru2a"

C:\Users\gleti\bin\oci.exe bastion bastion get --bastion-id $bastionId --query "data.""lifecycle-state""
```

Wait until it shows **ACTIVE** (usually takes 2-5 minutes).

## Step 3: Create a Bastion Session

### Option A: Managed SSH Session (Recommended)

This is the easiest method - OCI manages the SSH connection for you.

```powershell
# Set variables
$bastionId = "ocid1.bastion.oc1.iad.amaaaaaaofvcz2qaqfsdnnjspl5ws5vs4nqvrn62rt34pljrobufrahtru2a"
$instanceId = "<YOUR-INSTANCE-OCID>"
$publicKeyPath = "$env:USERPROFILE\.ssh\oci_bastion_key.pub"

# Create managed SSH session
C:\Users\gleti\bin\oci.exe bastion session create-managed-ssh `
  --bastion-id $bastionId `
  --key-type PUB `
  --ssh-public-key-file $publicKeyPath `
  --target-resource-id $instanceId `
  --target-os-username opc `
  --display-name "my-session" `
  --session-ttl 10800
```

**Important:** 
- For Oracle Linux: use `opc` as username
- For Ubuntu: use `ubuntu` as username
- For CentOS: use `centos` as username

### Option B: Port Forwarding Session

Use this if you need to access services running on the instance (like databases).

```powershell
# Create port forwarding session
C:\Users\gleti\bin\oci.exe bastion session create-port-forwarding `
  --bastion-id $bastionId `
  --key-type PUB `
  --ssh-public-key-file $publicKeyPath `
  --target-resource-id $instanceId `
  --target-port 22 `
  --display-name "port-forward-session" `
  --session-ttl 10800
```

## Step 4: Get Session Details

After creating a session, get the connection string:

```powershell
# List all sessions
C:\Users\gleti\bin\oci.exe bastion session list --bastion-id $bastionId

# Get specific session details
$sessionId = "<SESSION-OCID-FROM-ABOVE>"
C:\Users\gleti\bin\oci.exe bastion session get --session-id $sessionId
```

## Step 5: Connect via SSH

### For Managed SSH Session:

The output from Step 4 will include an SSH command. Copy and paste it:

```powershell
# Example command (yours will be different):
ssh -i ~\.ssh\oci_bastion_key -o ProxyCommand="ssh -i ~\.ssh\oci_bastion_key -W %h:%p -p 22 ocid1.bastionsession.oc1.iad.xxx@host.bastion.us-ashburn-1.oci.oraclecloud.com" -p 22 opc@<PRIVATE-IP>
```

### For Port Forwarding Session:

```powershell
# Start port forwarding
ssh -i ~\.ssh\oci_bastion_key -N -L 2222:<INSTANCE-PRIVATE-IP>:22 -p 22 ocid1.bastionsession.oc1.iad.xxx@host.bastion.us-ashburn-1.oci.oraclecloud.com

# In another terminal, connect to localhost
ssh -i ~\.ssh\oci_bastion_key -p 2222 opc@localhost
```

## Complete Example Workflow

Here's a full example from start to finish:

```powershell
# 1. Set your variables
$bastionId = "ocid1.bastion.oc1.iad.amaaaaaaofvcz2qaqfsdnnjspl5ws5vs4nqvrn62rt34pljrobufrahtru2a"
$prodComp = "ocid1.compartment.oc1..aaaaaaaaz4r2gcj4uehwhpda5h7crighwcdreppuatluiqvy2sxmda7amcga"
$privateSubnet = "ocid1.subnet.oc1.iad.aaaaaaaak2bae27mheowacne5pgz3kylrulpjnumxrg56hylwdty5wxjq5ta"

# 2. Check bastion status
C:\Users\gleti\bin\oci.exe bastion bastion get --bastion-id $bastionId

# 3. Launch a test instance (if you don't have one)
# Get availability domain
$ad = (C:\Users\gleti\bin\oci.exe iam availability-domain list --compartment-id $prodComp --query "data[0].name" --raw-output)

# Get Oracle Linux image
$imageId = (C:\Users\gleti\bin\oci.exe compute image list `
  --compartment-id $prodComp `
  --operating-system "Oracle Linux" `
  --sort-by TIMECREATED `
  --sort-order DESC `
  --limit 1 `
  --query "data[0].id" `
  --raw-output)

# Launch instance
C:\Users\gleti\bin\oci.exe compute instance launch `
  --availability-domain $ad `
  --compartment-id $prodComp `
  --shape "VM.Standard.E2.1.Micro" `
  --subnet-id $privateSubnet `
  --image-id $imageId `
  --display-name "test-instance" `
  --assign-public-ip false `
  --ssh-authorized-keys-file $env:USERPROFILE\.ssh\oci_bastion_key.pub

# 4. Get the instance OCID from the output above
$instanceId = "<INSTANCE-OCID>"

# 5. Create bastion session
C:\Users\gleti\bin\oci.exe bastion session create-managed-ssh `
  --bastion-id $bastionId `
  --key-type PUB `
  --ssh-public-key-file $env:USERPROFILE\.ssh\oci_bastion_key.pub `
  --target-resource-id $instanceId `
  --target-os-username opc `
  --display-name "test-session" `
  --session-ttl 10800

# 6. Get session details
$sessionId = "<SESSION-OCID-FROM-ABOVE>"
C:\Users\gleti\bin\oci.exe bastion session get --session-id $sessionId

# 7. Use the SSH command from the output
# ssh -i ~\.ssh\oci_bastion_key -o ProxyCommand="..." -p 22 opc@<IP>
```

## Troubleshooting

### Session Not Connecting

1. **Check Bastion Status:**
   ```powershell
   C:\Users\gleti\bin\oci.exe bastion bastion get --bastion-id $bastionId
   ```
   Should show `lifecycle-state: ACTIVE`

2. **Check Session Status:**
   ```powershell
   C:\Users\gleti\bin\oci.exe bastion session get --session-id $sessionId
   ```
   Should show `lifecycle-state: ACTIVE`

3. **Verify Security List:**
   Ensure your security list allows SSH (port 22) from the bastion subnet to your instance.

### SSH Key Issues

```powershell
# Verify key permissions (Windows)
icacls $env:USERPROFILE\.ssh\oci_bastion_key

# Key should only be accessible by you
# Fix if needed:
icacls $env:USERPROFILE\.ssh\oci_bastion_key /inheritance:r
icacls $env:USERPROFILE\.ssh\oci_bastion_key /grant:r "$env:USERNAME`:F"
```

### Session Expired

Sessions have a maximum TTL (3 hours by default). Create a new session if expired:

```powershell
# Delete old session
C:\Users\gleti\bin\oci.exe bastion session delete --session-id $sessionId

# Create new session
C:\Users\gleti\bin\oci.exe bastion session create-managed-ssh ...
```

## Best Practices

1. **Use Managed SSH:** Easier and more secure than port forwarding
2. **Limit Session TTL:** Set the minimum time needed
3. **Use Specific CIDR:** Instead of 0.0.0.0/0, use your office/home IP
4. **Delete Sessions:** Clean up unused sessions
5. **Audit Logs:** Review bastion session logs regularly
6. **Separate Keys:** Use different SSH keys for different environments

## Viewing Session Logs

```powershell
# List all sessions for audit
C:\Users\gleti\bin\oci.exe bastion session list --bastion-id $bastionId --all

# Get session lifecycle events
C:\Users\gleti\bin\oci.exe bastion session get --session-id $sessionId
```

## Quick Reference Commands

```powershell
# Bastion Commands
# =================

# List bastions
C:\Users\gleti\bin\oci.exe bastion bastion list --compartment-id $prodComp

# Get bastion details  
C:\Users\gleti\bin\oci.exe bastion bastion get --bastion-id $bastionId

# Update bastion (e.g., change max sessions)
C:\Users\gleti\bin\oci.exe bastion bastion update --bastion-id $bastionId --max-sessions-allowed 50

# Delete bastion
C:\Users\gleti\bin\oci.exe bastion bastion delete --bastion-id $bastionId


# Session Commands
# ================

# Create managed SSH session
C:\Users\gleti\bin\oci.exe bastion session create-managed-ssh --bastion-id $bastionId ...

# Create port forwarding session
C:\Users\gleti\bin\oci.exe bastion session create-port-forwarding --bastion-id $bastionId ...

# List sessions
C:\Users\gleti\bin\oci.exe bastion session list --bastion-id $bastionId

# Get session
C:\Users\gleti\bin\oci.exe bastion session get --session-id $sessionId

# Delete session
C:\Users\gleti\bin\oci.exe bastion session delete --session-id $sessionId
```

## Security Considerations

- **No Public IPs:** Instances don't need public IPs
- **Time-Limited:** Sessions automatically expire
- **Auditable:** All sessions are logged
- **Fine-Grained Control:** Can restrict by CIDR, TTL, and max sessions
- **No Bastion Host Maintenance:** Fully managed by OCI

## Resources

- [OCI Bastion Documentation](https://docs.oracle.com/en-us/iaas/Content/Bastion/home.htm)
- [Bastion Best Practices](https://docs.oracle.com/en-us/iaas/Content/Bastion/Tasks/managingbastions.htm)

---

**Last Updated:** 2026-02-12
