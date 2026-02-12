# OCI Infrastructure Setup

Complete documentation for Oracle Cloud Infrastructure (OCI) initial setup following best practices.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Setup Steps](#setup-steps)
- [Configuration Details](#configuration-details)
- [Cost Estimation](#cost-estimation)
- [Next Steps](#next-steps)
- [Troubleshooting](#troubleshooting)

## 🎯 Overview

This repository contains the complete setup documentation for a production-ready OCI environment, including:

- Identity and Access Management (IAM)
- Network architecture with VCN and subnets
- Security baseline configuration
- Cost management and budgeting
- Logging and monitoring

**Setup Date:** February 12, 2026  
**Region:** us-ashburn-1 (US East - Ashburn)  
**Organization:** The-Nohvech-Group

## 🏗️ Architecture

```
OCI Tenancy
├── Production Compartment
│   └── Production VCN (10.0.0.0/16)
│       ├── Public Subnet (10.0.1.0/24)
│       ├── Private App Subnet (10.0.10.0/24)
│       └── Private DB Subnet (10.0.20.0/24)
├── Development Compartment
└── Shared-Services Compartment
```

### Network Components

- **Internet Gateway:** Public internet access for public subnet
- **NAT Gateway:** Outbound internet for private subnets
- **Service Gateway:** Access to OCI services (Object Storage, etc.)

## ✅ Prerequisites

- OCI Account with administrator access
- OCI CLI installed and configured
- Python 3.10+ (for key generation scripts)
- Git (for version control)

## 🚀 Setup Steps

### 1. Security Configuration

- ✅ Root account MFA enabled
- ✅ API keys configured (passphrase-free for automation)
- ✅ Security logging enabled

### 2. Identity and Access Management

**Groups Created:**
- `OCI-Admins` - Full administrative access
- `OCI-Developers` - Development environment access
- `OCI-Auditors` - Read-only access for compliance

**Policies:**
- `OCI-Admins-Policy` - Manage all resources in tenancy
- `OCI-Developers-Policy` - Manage compute, networking, read all resources
- `OCI-Auditors-Policy` - Inspect and read all resources

### 3. Compartment Structure

```
Root
├── Production (OCID: ocid1.compartment.oc1..aaaaaaaaz4r2gcj4uehwhpda5h7crighwcdreppuatluiqvy2sxmda7amcga)
├── Development (OCID: ocid1.compartment.oc1..aaaaaaaahshfmgcjj27flugw3i55hv4wmnu2vodlaorum66b2qx4s6fo5yja)
└── Shared-Services (OCID: ocid1.compartment.oc1..aaaaaaaamn2sntcy67ykangyy6foczrvrzc5lzn7mgs222xktzfvdgdi3z5a)
```

### 4. Network Configuration

**Production VCN Details:**
- **Name:** Production-VCN
- **OCID:** ocid1.vcn.oc1.iad.amaaaaaaofvcz2qaokz6cqfhuntyg5e6yqzsuj4dr2ukkzjfbh3doynegppq
- **CIDR:** 10.0.0.0/16
- **DNS Label:** prodvcn
- **Domain:** prodvcn.oraclevcn.com

**Subnets:**

| Name | CIDR | Type | Purpose |
|------|------|------|---------|
| Public-Subnet | 10.0.1.0/24 | Public | Load balancers, Bastion |
| Private-App-Subnet | 10.0.10.0/24 | Private | Application servers |
| Private-DB-Subnet | 10.0.20.0/24 | Private | Database servers |

### 5. Security Baseline

- **Logging:** Security-Logs log group created
- **Vulnerability Scanning:** Weekly scans configured for Sunday
- **Cloud Guard:** Available for threat detection

### 6. Cost Management

- **Budget:** $100 USD monthly alert configured
- **Monitoring:** Cost analysis enabled

## 📊 Configuration Details

See detailed configuration files:
- [IAM Configuration](docs/iam-config.md)
- [Network Configuration](docs/network-config.md)
- [Security Configuration](docs/security-config.md)
- [OCI CLI Setup](docs/cli-setup.md)

## 💰 Cost Estimation

**Free Tier Resources:**
- 2 AMD-based compute instances (1/8 OCPU, 1GB RAM each)
- 2 Block Volumes (100GB total)
- 10GB Object Storage
- Load Balancer (10 Mbps)

**Estimated Monthly Cost (Beyond Free Tier):**
- Additional compute instances: ~$5-10 per instance
- Network egress: Variable based on usage
- Database services: Starting at $15/month

## 🎯 Next Steps

1. **Deploy Bastion Service** - Secure SSH access
2. **Launch Compute Instances** - Deploy workloads
3. **Configure Load Balancer** - Distribute traffic
4. **Set Up Database** - MySQL, PostgreSQL, or Oracle DB
5. **Implement Backup Strategy** - Automated backups
6. **Configure Monitoring** - Alarms and notifications

## 🔧 Troubleshooting

### OCI CLI Authentication Issues

If you encounter authentication errors:

```bash
# Verify OCI CLI configuration
oci --version
oci iam region list

# Check config file
cat ~/.oci/config

# Verify API key fingerprint matches in OCI Console
```

### Network Connectivity Issues

- Verify route tables are configured for gateways
- Check security lists allow required traffic
- Ensure instances are in correct subnet

## 📚 Resources

- [OCI Documentation](https://docs.oracle.com/en-us/iaas/)
- [OCI CLI Reference](https://docs.oracle.com/en-us/iaas/tools/oci-cli/latest/)
- [OCI Best Practices](https://docs.oracle.com/en/solutions/oci-best-practices/)
- [OCI Free Tier](https://www.oracle.com/cloud/free/)

## 📝 License

This documentation is provided as-is for reference purposes.

## 👥 Contributors

- Infrastructure Team - The-Nohvech-Group
- Setup Date: February 12, 2026

---

**Note:** This is a living document. Update as infrastructure evolves.
