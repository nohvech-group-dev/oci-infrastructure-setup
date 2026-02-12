# Next Steps

Now that your OCI infrastructure foundation is set up, here are the recommended next steps to deploy and manage workloads.

## 🎯 Immediate Next Steps

### 1. ✅ Bastion Service (IN PROGRESS)
**Status:** Creating  
**Purpose:** Secure SSH access to private instances

The Bastion service is being deployed to the Private-App-Subnet. Once active, you'll be able to SSH into instances without exposing them to the internet.

**How to use:**
```bash
# Create a session (after deploying an instance)
oci bastion session create-managed-ssh \
  --bastion-id <BASTION-OCID> \
  --key-type PUB \
  --ssh-public-key-file ~/.ssh/id_rsa.pub \
  --target-resource-id <INSTANCE-OCID> \
  --target-os-username opc \
  --display-name "my-session"

# Connect using the SSH command provided in the session details
```

### 2. Deploy Your First Compute Instance

#### Option A: Using Free Tier (Recommended for testing)

**Available in Free Tier:**
- 2x VM.Standard.E2.1.Micro (AMD) - 1/8 OCPU, 1GB RAM
- Or 4x VM.Standard.A1.Flex (Arm) - Up to 4 OCPUs, 24GB RAM

```bash
# Set variables
PROD_COMP="ocid1.compartment.oc1..aaaaaaaaz4r2gcj4uehwhpda5h7crighwcdreppuatluiqvy2sxmda7amcga"
PRIVATE_SUBNET="ocid1.subnet.oc1.iad.aaaaaaaak2bae27mheowacne5pgz3kylrulpjnumxrg56hylwdty5wxjq5ta"

# List availability domains
oci iam availability-domain list

# Get Oracle Linux image
IMAGE_ID=$(oci compute image list \
  --compartment-id $PROD_COMP \
  --operating-system "Oracle Linux" \
  --sort-by TIMECREATED \
  --sort-order DESC \
  --limit 1 \
  --query "data[0].id" \
  --raw-output)

# Launch instance
oci compute instance launch \
  --availability-domain "<AD-NAME>" \
  --compartment-id $PROD_COMP \
  --shape "VM.Standard.E2.1.Micro" \
  --subnet-id $PRIVATE_SUBNET \
  --image-id $IMAGE_ID \
  --display-name "app-server-01" \
  --assign-public-ip false \
  --ssh-authorized-keys-file ~/.ssh/id_rsa.pub
```

#### Option B: Using Terraform (Infrastructure as Code)

Create `main.tf`:
```hcl
terraform {
  required_providers {
    oci = {
      source  = "oracle/oci"
      version = "~> 5.0"
    }
  }
}

provider "oci" {
  region = "us-ashburn-1"
}

resource "oci_core_instance" "app_server" {
  availability_domain = "<AD-NAME>"
  compartment_id      = var.compartment_id
  shape               = "VM.Standard.E2.1.Micro"
  
  create_vnic_details {
    subnet_id        = var.subnet_id
    assign_public_ip = false
  }
  
  source_details {
    source_type = "image"
    source_id   = var.image_id
  }
  
  metadata = {
    ssh_authorized_keys = file("~/.ssh/id_rsa.pub")
  }
  
  display_name = "app-server-01"
}
```

### 3. Configure Security Rules

Update security lists to allow necessary traffic:

```bash
# Get security list OCID
VCN_ID="ocid1.vcn.oc1.iad.amaaaaaaofvcz2qaokz6cqfhuntyg5e6yqzsuj4dr2ukkzjfbh3doynegppq"
SL_OCID=$(oci network security-list list \
  --compartment-id $PROD_COMP \
  --vcn-id $VCN_ID \
  --query "data[0].id" \
  --raw-output)

# Allow SSH from Bastion subnet
oci network security-list update \
  --security-list-id $SL_OCID \
  --ingress-security-rules '[
    {
      "source": "10.0.10.0/24",
      "protocol": "6",
      "tcpOptions": {
        "destinationPortRange": {"min": 22, "max": 22}
      }
    }
  ]'
```

### 4. Set Up Load Balancer (Optional)

If you're deploying web applications:

```bash
# Create load balancer in public subnet
PUBLIC_SUBNET="ocid1.subnet.oc1.iad.aaaaaaaadwjp3ngjvfhlroqoaoitdc4ajgxs4jtwydk3ib5antigqyj4ah6q"

oci lb load-balancer create \
  --compartment-id $PROD_COMP \
  --display-name "app-lb" \
  --shape-name "flexible" \
  --subnet-ids "[\"$PUBLIC_SUBNET\"]" \
  --shape-details '{"minimumBandwidthInMbps": 10, "maximumBandwidthInMbps": 100}' \
  --is-private false
```

### 5. Configure Monitoring and Alarms

```bash
# Create alarm for instance CPU usage
oci monitoring alarm create \
  --compartment-id $PROD_COMP \
  --display-name "High-CPU-Alert" \
  --metric-compartment-id $PROD_COMP \
  --namespace "oci_computeagent" \
  --query-text "CpuUtilization[1m].mean() > 80" \
  --severity "WARNING" \
  --destinations '["<TOPIC-OCID>"]'
```

## 📋 Recommended Setup Sequence

1. **Week 1: Core Infrastructure**
   - ✅ Bastion service
   - ✅ First compute instance
   - ✅ Security rules configuration
   - ✅ Monitoring setup

2. **Week 2: Application Deployment**
   - Deploy application stack
   - Configure load balancer
   - Set up database (if needed)
   - Implement backup strategy

3. **Week 3: Operations**
   - Configure automated backups
   - Set up log aggregation
   - Implement CI/CD pipeline
   - Document runbooks

4. **Week 4: Optimization**
   - Review costs and optimize
   - Performance testing
   - Security hardening
   - Disaster recovery planning

## 🛠️ Additional Services to Consider

### Database Services
- **Autonomous Database:** Fully managed, auto-scaling
- **MySQL Database Service:** Managed MySQL with HeatWave
- **PostgreSQL:** Coming soon to OCI

### Container Services
- **OKE (Kubernetes Engine):** Managed Kubernetes
- **Container Instances:** Serverless containers
- **Container Registry:** Private Docker registry

### Serverless
- **Functions:** Event-driven serverless
- **API Gateway:** Managed API gateway
- **Events:** Event-driven automation

### DevOps
- **DevOps Service:** CI/CD pipelines
- **Resource Manager:** Terraform automation
- **Artifacts:** Package repository

## 🔐 Security Hardening

### 1. Enable OS Management
```bash
# Enable OS Management for patching
oci os-management managed-instance-group create \
  --compartment-id $PROD_COMP \
  --display-name "production-servers"
```

### 2. Implement Backup Strategy
```bash
# Create backup policy
oci bv volume-backup-policy create \
  --compartment-id $PROD_COMP \
  --display-name "daily-backups" \
  --schedules '[{
    "backupType": "INCREMENTAL",
    "period": "ONE_DAY",
    "retentionSeconds": 2592000
  }]'
```

### 3. Enable Cloud Guard Targets
```bash
# Enable Cloud Guard for compartment
oci cloud-guard target create \
  --compartment-id $PROD_COMP \
  --display-name "Production-Target" \
  --target-resource-type COMPARTMENT \
  --target-resource-id $PROD_COMP
```

## 📊 Cost Optimization Tips

1. **Use Free Tier:** Maximize free tier resources first
2. **Right-size Instances:** Start small, scale as needed
3. **Reserved Instances:** Consider for long-term workloads
4. **Auto-scaling:** Implement for variable workloads
5. **Budget Alerts:** Monitor and optimize regularly

## 📚 Learning Resources

- [OCI Architecture Center](https://docs.oracle.com/solutions/)
- [OCI Free Tier](https://www.oracle.com/cloud/free/)
- [OCI Terraform Provider](https://registry.terraform.io/providers/oracle/oci/latest/docs)
- [OCI GitHub Examples](https://github.com/oracle/oci-quickstart)

## 🆘 Need Help?

- **Documentation:** https://docs.oracle.com/en-us/iaas/
- **Forums:** https://community.oracle.com/cloud
- **Support:** https://cloud.oracle.com/support
- **GitHub Issues:** Report infrastructure code issues

---

**Last Updated:** 2026-02-12
