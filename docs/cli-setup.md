# OCI CLI Setup Guide

Complete guide for installing and configuring the Oracle Cloud Infrastructure CLI.

## Installation

### Windows (PowerShell)

```powershell
# Install OCI CLI
powershell -NoProfile -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.ps1'))"

# Add to PATH (automatically done by installer)
# Verify installation
oci --version
```

### Linux/macOS

```bash
bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"
```

## Configuration

### 1. Generate API Keys (Without Passphrase)

Create a Python script to generate keys:

```python
# save as generate_oci_keys.py
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.backends import default_backend
import hashlib

# Generate private key
private_key = rsa.generate_private_key(
    public_exponent=65537,
    key_size=2048,
    backend=default_backend()
)

# Serialize private key (without encryption)
private_pem = private_key.private_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PrivateFormat.TraditionalOpenSSL,
    encryption_algorithm=serialization.NoEncryption()
)

# Serialize public key
public_key = private_key.public_key()
public_pem = public_key.public_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PublicFormat.SubjectPublicKeyInfo
)

# Calculate fingerprint
public_der = public_key.public_bytes(
    encoding=serialization.Encoding.DER,
    format=serialization.PublicFormat.SubjectPublicKeyInfo
)
fingerprint = hashlib.md5(public_der).hexdigest()
fingerprint_formatted = ':'.join([fingerprint[i:i+2] for i in range(0, len(fingerprint), 2)])

# Write keys
with open('oci_api_key.pem', 'wb') as f:
    f.write(private_pem)

with open('oci_api_key_public.pem', 'wb') as f:
    f.write(public_pem)

print(f"Keys generated successfully!")
print(f"Fingerprint: {fingerprint_formatted}")
print(f"\nPublic key to upload to OCI Console:")
print(public_pem.decode('utf-8'))
```

Run the script:

```bash
pip install cryptography
python generate_oci_keys.py
```

### 2. Upload Public Key to OCI Console

1. Log into OCI Console
2. Click **Profile** → **User Settings**
3. Scroll to **API Keys** section
4. Click **Add API Key**
5. Select **Paste Public Key**
6. Paste the public key from `oci_api_key_public.pem`
7. Click **Add**
8. Verify the fingerprint matches

### 3. Configure OCI CLI

Create or edit `~/.oci/config`:

```ini
[DEFAULT]
user=<your-user-ocid>
fingerprint=<your-api-key-fingerprint>
tenancy=<your-tenancy-ocid>
region=us-ashburn-1
key_file=<path-to-your-private-key>
```

**Example:**

```ini
[DEFAULT]
user=ocid1.user.oc1..aaaaaaaa36cn7qynlucsakvyolhuanovhh2o7j3sojqrdp7i6vtj5pl4zfkq
fingerprint=85:91:fe:23:1a:0e:ae:85:b2:ab:cc:e6:ed:81:ac:89
tenancy=ocid1.tenancy.oc1..aaaaaaaawrxbb3qlbs2gu5otpirdfgnh3x7exfw7l5i7lkm4vy3stuuux4qa
region=us-ashburn-1
key_file=C:\Users\gleti\.oci\oci_api_key.pem
```

### 4. Test Configuration

```bash
# Test authentication
oci iam region list

# List compartments
oci iam compartment list --all

# Check current user
oci iam user get --user-id <your-user-ocid>
```

## PowerShell Profile Setup (Windows)

Add OCI CLI to your PowerShell profile for automatic loading:

```powershell
# Create profile if it doesn't exist
if (!(Test-Path $PROFILE)) {
    New-Item -Path $PROFILE -ItemType File -Force
}

# Add OCI CLI to PATH
Add-Content -Path $PROFILE -Value @"

# OCI CLI Configuration
`$env:PATH += ';C:\Users\<your-username>\bin'
"@
```

## Common Commands

### IAM Operations

```bash
# List groups
oci iam group list --compartment-id <tenancy-ocid>

# Create group
oci iam group create --name "MyGroup" --description "Description" --compartment-id <tenancy-ocid>

# Create policy
oci iam policy create --compartment-id <compartment-ocid> --name "MyPolicy" --description "Description" --statements '["Allow group MyGroup to manage all-resources in compartment MyCompartment"]'
```

### Networking

```bash
# List VCNs
oci network vcn list --compartment-id <compartment-ocid>

# Create VCN
oci network vcn create --cidr-block "10.0.0.0/16" --compartment-id <compartment-ocid> --display-name "MyVCN"

# List subnets
oci network subnet list --compartment-id <compartment-ocid> --vcn-id <vcn-ocid>
```

### Compute

```bash
# List compute shapes
oci compute shape list --compartment-id <compartment-ocid>

# List images
oci compute image list --compartment-id <compartment-ocid>

# Launch instance
oci compute instance launch --availability-domain <ad-name> --compartment-id <compartment-ocid> --shape <shape-name> --subnet-id <subnet-ocid> --image-id <image-ocid>
```

## Troubleshooting

### Authentication Errors

**Error:** `NotAuthenticated` or `The required information to complete authentication was not provided`

**Solutions:**
1. Verify API key is uploaded to OCI Console
2. Check fingerprint matches in config file
3. Ensure user OCID is correct
4. Verify region is correct (use home region if unsure)

### Permission Errors

**Error:** `Authorization failed or requested resource not found`

**Solutions:**
1. Check user is in appropriate group
2. Verify policies grant required permissions
3. Ensure compartment OCID is correct

### File Permission Warnings

On Windows, you may see warnings about file permissions. These can be ignored or fixed:

```bash
oci setup repair-file-permissions --file ~/.oci/config
oci setup repair-file-permissions --file ~/.oci/oci_api_key.pem
```

## Best Practices

1. **Never share private keys** - Keep `oci_api_key.pem` secure
2. **Use separate keys per environment** - Dev, staging, production
3. **Rotate keys regularly** - Every 90 days minimum
4. **Use profiles** - Configure multiple profiles in config file for different environments
5. **Limit API key permissions** - Use least privilege principle

## Additional Resources

- [OCI CLI Documentation](https://docs.oracle.com/en-us/iaas/tools/oci-cli/latest/)
- [OCI CLI Command Reference](https://docs.oracle.com/en-us/iaas/tools/oci-cli/latest/oci_cli_docs/)
- [API Key Management](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm)
