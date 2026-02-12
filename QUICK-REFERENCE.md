# Quick Reference Guide

Quick commands and OCIDs for your OCI infrastructure.

## Environment Details

**Tenancy OCID:**
```
ocid1.tenancy.oc1..aaaaaaaawrxbb3qlbs2gu5otpirdfgnh3x7exfw7l5i7lkm4vy3stuuux4qa
```

**User OCID:**
```
ocid1.user.oc1..aaaaaaaa36cn7qynlucsakvyolhuanovhh2o7j3sojqrdp7i6vtj5pl4zfkq
```

**Region:** us-ashburn-1

## Compartments

| Name | OCID |
|------|------|
| Production | `ocid1.compartment.oc1..aaaaaaaaz4r2gcj4uehwhpda5h7crighwcdreppuatluiqvy2sxmda7amcga` |
| Development | `ocid1.compartment.oc1..aaaaaaaahshfmgcjj27flugw3i55hv4wmnu2vodlaorum66b2qx4s6fo5yja` |
| Shared-Services | `ocid1.compartment.oc1..aaaaaaaamn2sntcy67ykangyy6foczrvrzc5lzn7mgs222xktzfvdgdi3z5a` |

## Network Resources

### Production VCN
**OCID:** `ocid1.vcn.oc1.iad.amaaaaaaofvcz2qaokz6cqfhuntyg5e6yqzsuj4dr2ukkzjfbh3doynegppq`  
**CIDR:** 10.0.0.0/16

### Subnets
- **Public:** 10.0.1.0/24
- **Private App:** 10.0.10.0/24
- **Private DB:** 10.0.20.0/24

## Quick Commands

### List Resources

```bash
# Set variables for easy reuse
export TENANCY_OCID="ocid1.tenancy.oc1..aaaaaaaawrxbb3qlbs2gu5otpirdfgnh3x7exfw7l5i7lkm4vy3stuuux4qa"
export PROD_COMP="ocid1.compartment.oc1..aaaaaaaaz4r2gcj4uehwhpda5h7crighwcdreppuatluiqvy2sxmda7amcga"
export VCN_OCID="ocid1.vcn.oc1.iad.amaaaaaaofvcz2qaokz6cqfhuntyg5e6yqzsuj4dr2ukkzjfbh3doynegppq"

# List all compartments
oci iam compartment list --all

# List VCNs in Production
oci network vcn list --compartment-id $PROD_COMP

# List subnets
oci network subnet list --compartment-id $PROD_COMP --vcn-id $VCN_OCID

# List compute instances
oci compute instance list --compartment-id $PROD_COMP

# List groups
oci iam group list --compartment-id $TENANCY_OCID

# List policies
oci iam policy list --compartment-id $TENANCY_OCID
```

### PowerShell Variables

```powershell
$tenancyOcid = "ocid1.tenancy.oc1..aaaaaaaawrxbb3qlbs2gu5otpirdfgnh3x7exfw7l5i7lkm4vy3stuuux4qa"
$prodComp = "ocid1.compartment.oc1..aaaaaaaaz4r2gcj4uehwhpda5h7crighwcdreppuatluiqvy2sxmda7amcga"
$vcnOcid = "ocid1.vcn.oc1.iad.amaaaaaaofvcz2qaokz6cqfhuntyg5e6yqzsuj4dr2ukkzjfbh3doynegppq"
```

## Common Tasks

### Launch a Compute Instance

```bash
# List available shapes
oci compute shape list --compartment-id $PROD_COMP

# List available images
oci compute image list --compartment-id $PROD_COMP --operating-system "Oracle Linux"

# Get subnet OCID (replace with your subnet name)
SUBNET_OCID=$(oci network subnet list --compartment-id $PROD_COMP --display-name "Private-App-Subnet" --query "data[0].id" --raw-output)

# Launch instance (adjust parameters as needed)
oci compute instance launch \
  --availability-domain "<AD-NAME>" \
  --compartment-id $PROD_COMP \
  --shape "VM.Standard.E2.1.Micro" \
  --subnet-id $SUBNET_OCID \
  --image-id "<IMAGE-OCID>" \
  --display-name "my-instance" \
  --assign-public-ip false
```

### Create Security List Rule

```bash
# Get security list OCID
SL_OCID=$(oci network security-list list --compartment-id $PROD_COMP --vcn-id $VCN_OCID --query "data[0].id" --raw-output)

# Update security list (example: allow SSH from specific IP)
oci network security-list update \
  --security-list-id $SL_OCID \
  --ingress-security-rules '[{"source": "1.2.3.4/32", "protocol": "6", "tcpOptions": {"destinationPortRange": {"min": 22, "max": 22}}}]'
```

### Cost Monitoring

```bash
# View current budget
oci budgets budget list --compartment-id $TENANCY_OCID

# Get cost usage (last 30 days)
oci usage-api usage-summary request-summarized-usages \
  --granularity DAILY \
  --tenant-id $TENANCY_OCID \
  --time-usage-started $(date -u -d '30 days ago' +%Y-%m-%dT%H:%M:%S.000Z) \
  --time-usage-ended $(date -u +%Y-%m-%dT%H:%M:%S.000Z)
```

## Web Console URLs

- **OCI Console:** https://cloud.oracle.com/
- **Compartments:** https://cloud.oracle.com/identity/compartments
- **VCN:** https://cloud.oracle.com/networking/vcns
- **Compute Instances:** https://cloud.oracle.com/compute/instances
- **Cost Analysis:** https://cloud.oracle.com/usage/cost-analysis

## Support and Documentation

- **OCI Documentation:** https://docs.oracle.com/en-us/iaas/
- **OCI CLI Docs:** https://docs.oracle.com/en-us/iaas/tools/oci-cli/latest/
- **Support:** https://cloud.oracle.com/support

## Troubleshooting

### CLI Not Working
```bash
# Verify configuration
oci setup config

# Test authentication
oci iam region list

# Check API key fingerprint
cat ~/.oci/config
```

### Can't See Resources
- Check you're in the correct region
- Verify compartment selection in console
- Ensure user has proper permissions

### Network Issues
- Verify security lists allow traffic
- Check route tables for gateway routes
- Ensure instances are in correct subnet

---

**Last Updated:** 2026-02-12
