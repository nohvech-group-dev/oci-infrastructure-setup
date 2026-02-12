# IAM Configuration

Identity and Access Management setup for OCI infrastructure.

## Groups

### OCI-Admins
- **OCID:** ocid1.group.oc1..aaaaaaaal4aolicdpgy4le34of4krdrtxp4rxaohbefdrui7kbqnv7kv6xsq
- **Purpose:** Full administrative access to all resources
- **Members:** Add administrator users here
- **Created:** 2026-02-12

### OCI-Developers  
- **Purpose:** Development environment access
- **Permissions:**
  - Manage compute instances
  - Manage virtual networks
  - Read all resources
- **Members:** Development team members

### OCI-Auditors
- **Purpose:** Read-only access for compliance and auditing
- **Permissions:**
  - Inspect all resources
  - Read all resources
- **Members:** Compliance and audit team

## Policies

### OCI-Admins-Policy

```
Allow group OCI-Admins to manage all-resources in tenancy
```

**Scope:** Tenancy-wide  
**Purpose:** Grants full administrative access to all resources

### OCI-Developers-Policy

```
Allow group OCI-Developers to manage instance-family in tenancy
Allow group OCI-Developers to manage virtual-network-family in tenancy
Allow group OCI-Developers to inspect compartments in tenancy
Allow group OCI-Developers to read all-resources in tenancy
```

**Scope:** Tenancy-wide  
**Purpose:** Allows developers to manage compute and networking resources

### OCI-Auditors-Policy

```
Allow group OCI-Auditors to inspect all-resources in tenancy
Allow group OCI-Auditors to read all-resources in tenancy
```

**Scope:** Tenancy-wide  
**Purpose:** Read-only access for auditing

## Best Practices

1. **Least Privilege:** Only grant minimum required permissions
2. **Group-Based Access:** Always assign users to groups, not individual policies
3. **Regular Reviews:** Review group memberships quarterly
4. **MFA Enforcement:** Require MFA for all administrative users
5. **Separate Accounts:** Use separate accounts for different roles

## Adding Users

### Via OCI Console

1. Navigate to **Identity → Domains → Default → Users**
2. Click **Create User**
3. Fill in user details
4. Add user to appropriate groups
5. Enable MFA for the user

### Via OCI CLI

```bash
# Create user
oci iam user create --name "john.doe" --description "John Doe - Developer" --email "john.doe@example.com"

# Get user OCID
USER_OCID=$(oci iam user list --name "john.doe" --query "data[0].id" --raw-output)

# Add to group
GROUP_OCID=$(oci iam group list --name "OCI-Developers" --query "data[0].id" --raw-output)
oci iam group add-user --user-id $USER_OCID --group-id $GROUP_OCID
```

## Policy Examples

### Compartment-Specific Policy

```
Allow group OCI-Developers to manage instance-family in compartment Development
Allow group OCI-Developers to manage virtual-network-family in compartment Development
```

### Resource-Type Specific

```
Allow group Database-Admins to manage database-family in compartment Production
Allow group Network-Team to manage virtual-network-family in tenancy
```

### Conditional Policies

```
Allow group Developers to manage instance-family in compartment Development where request.region = 'us-ashburn-1'
```

## Security Recommendations

1. **Rotate API Keys:** Every 90 days
2. **Monitor Audit Logs:** Regular review of IAM changes
3. **Federated Identity:** Consider integrating with IdP (Azure AD, Okta)
4. **Service Accounts:** Use dedicated users for automation
5. **Break-Glass Account:** Maintain emergency access account

## Resources

- [OCI IAM Documentation](https://docs.oracle.com/en-us/iaas/Content/Identity/home.htm)
- [Policy Reference](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/policyreference.htm)
- [Common Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/commonpolicies.htm)
