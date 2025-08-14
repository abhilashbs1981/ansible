# EMS503 Secondary Environment Enablement Documentation

## Secondary Environment Enablement Redundancy (GR) Implementation Guide

### Document Information
- Document Title: EMS503 Secondary Environment Enablement Documentation
- Project: Secondary Environment Enablement Redundancy (GR)
- Environment: EMS503
- Date: [Current Date]
- Author: [Author Name]
- Review Status: [Draft/In Review/Approved]

## Preface

This document outlines the implementation of a Secondary Environment Enablement Redundancy (GR) enabled setup for the EMS503 environment. The solution addresses the requirement to deploy both primary and secondary NEMS instances within the same inventory structure while enabling independent deployment control.

### Key Features
- Independent Deployment Control: Maintains proper separation of configurations, vault secrets, and Kubernetes contexts
- Flexible Targeting: Primary and secondary NEMS instances are differentiated through dedicated host groups (ems and ems_secondary)
- Ansible Integration: Enables flexible deployment scenarios through Ansible's host targeting mechanisms using the -l flag

## Overview

This document covers the comprehensive changes made to both environment and playbook configurations to support deployment of both primary and secondary EMS instances within the same environment using the -l flag for precise control.

## 1. Secondary NEMS - Environment Changes

### 1.1 Reference Environment
- Environment: environments/envs/ems503
- Playbook: playbook/ems-charts-deploy.yml

### 1.2 Scope
The implementation scope covers:
- Environment: environments/envs/ems503
- Playbook(s): playbook/ems-charts-deploy.yml

### 1.3 What Changed

#### 1.3.1 Inventory Structure
- Separate Secondary EMS Host/Group: Inventory now models a separate secondary EMS host/group alongside the existing primary
- Dedicated Host Variables: A dedicated host_vars directory for the secondary EMS was created by cloning primary host vars (including its vault file) to maintain identical behavior initially
- Shared Group Variables: group_vars/access40 remain common to both EMS hosts via the inventory's children relationship

#### 1.3.2 Files Modified for Environment

**Updated Files:**
- environments/envs/ems503/ems503.inventory (Modified inventory structure)

**Added Files:**
- environments/envs/ems503/host_vars/ems503-secondary/vars (Secondary EMS variables)
- environments/envs/ems503/host_vars/ems503-secondary/vault (Secondary EMS vault file)

### 1.4 Inventory Structure (Effective State)

#### 1.4.1 Host Variables

**Primary EMS (Existing)**
- environments/envs/ems503/host_vars/ems503/vars
- environments/envs/ems503/host_vars/ems503/vault (vault-encrypted)

**Secondary EMS (New - Identical to Primary)**
- environments/envs/ems503/host_vars/ems503-secondary/vars
- environments/envs/ems503/host_vars/ems503-secondary/vault (vault-encrypted)

#### 1.4.2 Group Variables (Existing)
**Shared for Both EMS Instances via Inventory Children Mapping**
- environments/envs/ems503/group_vars/access40/vars
- environments/envs/ems503/group_vars/access40/vault
- environments/envs/ems503/group_vars/access40/vault-pod-common

**Note:** If secondary requires different defaults, either override in host_vars/ems503-secondary/* or create environments/envs/ems503/group_vars/ems_secondary/.

### 1.5 How Ansible Resolves Variables (Summary)

**Primary Target:**
- Target: ems
- Host: ems503
- Variable Loading Order: host_vars/ems503/* + group_vars/access40/* + inventory line vars

**Secondary Target:**
- Target: ems_secondary
- Host: ems503-secondary
- Variable Loading Order: host_vars/ems503-secondary/* + group_vars/access40/* + inventory line vars

**Important:** Host vars take precedence over group vars.

## 2. Secondary EMS - Playbook Changes

### 2.1 Overview
This section covers the changes made to the following playbooks to support deployment of both primary and secondary EMS instances:
- playbook/ems-charts-deploy.yml
- playbook/ems-charts-purge.yml

### 2.2 Playbook Structure Changes

**Before (Original):**
- Target Groups: Single EMS group
- Host Targeting: Fixed targeting
- Kubeconfig Generation: Single config file

**After (Updated):**
- Target Groups: ems,ems_secondary groups
- Host Targeting: Flexible with -l flag
- Kubeconfig Generation: Separate config per EMS instance

**Note:** These changes apply to both ems-charts-deploy.yml and ems-charts-purge.yml.

### 2.3 Control Methods

#### 2.3.1 Primary Method: Host Targeting with -l Flag

**Command: -l ems**
- Target: Primary EMS hosts only
- Description: Targets only primary EMS hosts

**Command: -l ems_secondary**
- Target: Secondary EMS hosts only
- Description: Targets only secondary EMS hosts

**Command: -l ems,ems_secondary**
- Target: Both EMS groups
- Description: Targets both primary and secondary EMS hosts

### 2.4 Kubeconfig File Generation

#### 2.4.1 File Pattern
Each EMS instance generates its own kubeconfig file using the following pattern:

{{ workdir }}/.kube/{{ inventory_dir | basename }}-{{ inventory_hostname }}.config

#### 2.4.2 Generated Files
- Primary EMS: ems503-ems503.config
- Secondary EMS: ems503-ems503-secondary.config

### 2.5 Deployment Commands

#### 2.5.1 Deploy Everything (Default - Both EMS Instances)
- Executes: All plays (Prepare + Deploy EMS) for both groups
- Result: Both EMS instances are properly prepared and deployed
- Generates: Both kubeconfig files

#### 2.5.2 Primary EMS Only
- Executes: Prepare + Deploy EMS for primary EMS only
- Result: Only primary EMS is deployed
- Generates: Primary config only

#### 2.5.3 Secondary EMS Only
- Executes: Prepare + Deploy EMS for secondary EMS only
- Result: Only secondary EMS is deployed
- Generates: Secondary config only

#### 2.5.4 Both EMS Instances (Explicit)
- Executes: Prepare + Deploy EMS for both groups
- Result: Both EMS instances are deployed
- Generates: Both kubeconfig files

### 2.6 Purge Commands

**Default:**
- Target: Both EMS instances
- Result: Purges both primary and secondary

**Primary Only:**
- Target: Primary EMS
- Result: Purges only primary EMS

**Secondary Only:**
- Target: Secondary EMS
- Result: Purges only secondary EMS

**Both Explicit:**
- Target: Both EMS instances
- Result: Purges both EMS instances

### 2.7 Technical Implementation

#### 2.7.1 Environment Variables
- Kubeconfig Setting: Each play sets K8S_AUTH_KUBECONFIG to the appropriate kubeconfig file
- Cluster Context: Kubernetes operations use the correct cluster context for each EMS instance

#### 2.7.2 Host Targeting
- Group Targeting: Both plays now target ems,ems_secondary groups
- Flag Control: Use -l flag to limit execution to specific host groups
- Precise Control: Provides exact control over which EMS instances are processed

### 2.8 Benefits

1. Full Control: Deploy primary, secondary, or both independently
2. Simple Control: Use -l flag for precise host targeting
3. Consistent Behavior: Same deployment logic for both instances
4. Separate Kubeconfig: Each EMS operates on its own cluster context
5. Clear Separation: Each EMS has its own host group

## 3. Operational Procedures

### 3.1 Optional Checks

#### 3.1.1 Show Inventory Graph
```
ansible-inventory -i environments/envs/ems503/inventory --graph | cat
```

#### 3.1.2 Dry-Run (Secondary)
```
# Example dry-run command for secondary EMS
ansible-playbook -i environments/envs/ems503/inventory playbook/ems-charts-deploy.yml -l ems_secondary --check
```

### 3.2 POD Considerations

**Important:** No changes were made to POD inventory. If PODs link to EMS via a variable (for example ems_fqdn), keep them pointing to the primary (ems503) for GR-enabled setups.

### 3.3 Vault and Secrets Management

#### 3.3.1 Current State
- Secondary Secrets: Currently reuses the same secrets as primary by design (identical host_vars and shared group_vars/access40/vault)

#### 3.3.2 Future Customization
To diverge secondary secrets later, edit with vault:
```
ansible-vault edit environments/envs/ems503/host_vars/ems503-secondary/vault
```

## 4. Deployment Examples

### 4.1 Common Deployment Scenarios

#### 4.1.1 Deploy Both EMS Instances
```
# Deploy both primary and secondary EMS
ansible-playbook -i environments/envs/ems503/inventory playbook/ems-charts-deploy.yml
```

#### 4.1.2 Deploy Primary EMS Only
```
# Deploy only primary EMS
ansible-playbook -i environments/envs/ems503/inventory playbook/ems-charts-deploy.yml -l ems
```

#### 4.1.3 Deploy Secondary EMS Only
```
# Deploy only secondary EMS
ansible-playbook -i environments/envs/ems503/inventory playbook/ems-charts-deploy.yml -l ems_secondary
```

### 4.2 Purge Operations

#### 4.2.1 Purge Both EMS Instances
```
# Purge both primary and secondary EMS
ansible-playbook -i environments/envs/ems503/inventory playbook/ems-charts-purge.yml
```

#### 4.2.2 Purge Specific EMS Instance
```
# Purge only secondary EMS
ansible-playbook -i environments/envs/ems503/inventory playbook/ems-charts-purge.yml -l ems_secondary
```

## 5. Troubleshooting

### 5.1 Common Issues

#### 5.1.1 Kubeconfig Generation Issues
- Problem: Kubeconfig files not generated correctly
- Solution: Verify inventory structure and host group definitions
- Check: Ensure ems and ems_secondary groups are properly defined

#### 5.1.2 Variable Resolution Issues
- Problem: Secondary EMS using primary EMS variables
- Solution: Verify host_vars/ems503-secondary/ directory structure
- Check: Ensure secondary host variables are properly configured

### 5.2 Validation Commands

```
# Validate inventory structure
ansible-inventory -i environments/envs/ems503/inventory --list

# Check host group membership
ansible-inventory -i environments/envs/ems503/inventory --graph

# Verify variable resolution
ansible-inventory -i environments/envs/ems503/inventory --host ems503-secondary
```

## 6. Appendices

### Appendix A: File Structure
```
environments/envs/ems503/
├── ems503.inventory
├── host_vars/
│   ├── ems503/
│   │   ├── vars
│   │   └── vault
│   └── ems503-secondary/
│       ├── vars
│       └── vault
└── group_vars/
    └── access40/
        ├── vars
        ├── vault
        └── vault-pod-common
```

### Appendix B: Command Reference

**Command: -l ems**
- Description: Target primary EMS only
- Use Case: Primary EMS operations

**Command: -l ems_secondary**
- Description: Target secondary EMS only
- Use Case: Secondary EMS operations

**Command: -l ems,ems_secondary**
- Description: Target both EMS instances
- Use Case: Full deployment/purge

### Appendix C: Change Log

**Version 1.0**
- Date: [Date]
- Author: [Author]
- Changes: Initial implementation of secondary EMS support

## Document Control

- Last Updated: [Date]
- Next Review: [Date + 3 months]
- Distribution: Technical Team, Operations Team, DevOps Engineers
- Classification: Internal Use

---

This document provides comprehensive guidance for implementing and operating the EMS503 secondary environment enablement redundancy setup. For questions or clarifications, contact the DevOps team.
