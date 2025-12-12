# The-Merchandise-Vault--Active-Directory-Infrastructure

## Project Overview

This project demonstrates the design and implementation of an on-premises Active Directory environment for **The Merchandise Vault**, a global retail and e-commerce company with operations across three geographic regions: New York (Headquarters), London (EMEA), and Singapore (APAC).

The infrastructure supports 30 employees across multiple departments and incorporates enterprise security best practices including role-based access control, Group Policy management, and administrative delegation.

---

## Business Scenario

The Merchandise Vault is a mid-sized retail company specializing in merchandise sales through both online and physical retail channels. With a growing global presence, the company required a centralized identity management solution that:

- Supports employees across three international locations
- Enforces consistent security policies while allowing regional flexibility
- Enables role-based access to resources based on job function
- Provides secure delegation of administrative tasks to regional IT teams
- Establishes a foundation for future hybrid cloud integration

---

## Environment Details

| Component | Details |
|-----------|---------|
| Domain Name | TheMerchandiseVault.local |
| Domain Controller | DC01-NYC (New York) |
| Forest/Domain Functional Level | Windows Server 2022 |
| Total Users | 30 |
| Geographic Locations | New York, London, Singapore |

---

## Organizational Unit Structure

```
TheMerchandiseVault.local
├── Builtin
├── Computers
├── Disabled Accounts
├── Domain Controllers
│   └── DC01-NYC
├── ForeignSecurityPrincipals
├── Groups
├── Locations
│   ├── London
│   │   ├── Computers
│   │   ├── Groups
│   │   └── Users
│   ├── New York
│   │   ├── Computers
│   │   ├── Groups
│   │   └── Users
│   └── Singapore
│       ├── Computers
│       ├── Groups
│       └── Users
├── Managed Service Accounts
├── Service Accounts
└── Users
```

### Design Rationale

A **geographic OU structure** was chosen for the following reasons:

1. **Regional GPO Application** — Different locations may require different policies based on local compliance requirements, time zones, or operational needs
2. **Delegated Administration** — Regional IT staff can be granted administrative permissions scoped to their location only
3. **Scalability** — New locations can be added as additional OUs without restructuring
4. **Clear Organization** — Intuitive structure that mirrors the company's operational reality

---

## User Distribution

| Location | Department | Count | Example Roles |
|----------|------------|-------|---------------|
| New York | IT | 4 | IT Administrator, Helpdesk, Security Analyst |
| New York | Finance | 4 | CFO, Accountant, Billing Specialist |
| New York | HR | 3 | HR Director, HR Generalist, Recruiter |
| New York | Executive | 3 | CEO, COO, VP Operations |
| London | Sales | 4 | Sales Manager, Sales Representatives |
| London | Warehouse | 4 | Warehouse Manager, Inventory Clerks |
| Singapore | Shipping | 4 | Shipping Manager, Shipping Clerks |
| Singapore | Sales | 4 | Regional Sales Manager, E-commerce Specialists |

**Total: 30 Users**

---

## Security Groups

### Location-Specific Groups

| Group Name | Purpose |
|------------|---------|
| SG-NYC-IT-Admins | Full administrative access to NYC resources |
| SG-NYC-Helpdesk | Password reset and account unlock for NYC users |
| SG-NYC-Finance | Access to financial systems and shared drives |
| SG-LON-Warehouse | Access to inventory management systems |
| SG-LON-Sales | Access to CRM and sales reporting |
| SG-SG-Shipping | Access to shipping and logistics systems |
| SG-SG-Sales | Access to APAC sales resources |

### Global Groups

| Group Name | Purpose |
|------------|---------|
| SG-Global-Executives | Cross-location access for executive leadership |
| SG-Global-IT-Admins | Enterprise-wide IT administrative access |
| SG-Global-Managers | Access to management reporting across all locations |

---

## Group Policy Objects

### Domain-Level Policies

| GPO Name | Settings | Linked To |
|----------|----------|-----------|
| Default Password Policy | 12+ characters, complexity enabled, 90-day expiration, 3 failed attempts lockout, 10-minute lockout duration | Domain |
| Global Audit Policy | Audit logon success/failure, account management, policy changes | Domain |

### Location-Level Policies

| GPO Name | Settings | Linked To |
|----------|----------|-----------|
| NYC-Workstation-Policy | Screen lock after 5 minutes, prevent control panel access for non-IT | Locations/New York |
| LON-Workstation-Policy | Screen lock after 5 minutes, regional settings for UK | Locations/London |
| SG-Workstation-Policy | Screen lock after 5 minutes, regional settings for Singapore | Locations/Singapore |

---

## Delegation of Control

| Location | Delegated Group | Permissions Granted |
|----------|-----------------|---------------------|
| New York | SG-NYC-Helpdesk | Reset passwords, unlock accounts, read user properties |
| London | SG-LON-IT-Support | Reset passwords, unlock accounts, read user properties |
| Singapore | SG-SG-IT-Support | Reset passwords, unlock accounts, read user properties |

This delegation follows the **principle of least privilege** — helpdesk staff can only manage users within their assigned region.

---

## Service Accounts

| Account | Purpose | OU Location |
|---------|---------|-------------|
| svc-backup | Automated backup operations | Service Accounts |
| svc-sql | SQL Server database access | Service Accounts |
| svc-monitoring | System monitoring agent | Service Accounts |

**Note:** Service accounts are configured with strong passwords and "Password never expires" is enabled with compensating controls (regular password rotation via scheduled task).

---

## Security Hardening Measures

1. **Default Administrator Account** — Renamed and disabled
2. **Guest Account** — Disabled
3. **Inactive Account Policy** — Accounts inactive for 90+ days are moved to Disabled Accounts OU
4. **Fine-Grained Password Policy** — IT Admins require 15+ character passwords with 60-day expiration
5. **Audit Logging** — All authentication events logged to Event Viewer

---

## Validation and Testing

The following tests were performed to validate the configuration:

- [ ] User can log in successfully with correct credentials
- [ ] User is locked out after 5 failed password attempts
- [ ] GPO applies correctly (verified with `gpresult /r`)
- [ ] Helpdesk can reset passwords for users in their location only
- [ ] Helpdesk cannot reset passwords for users in other locations
- [ ] Screen lock activates after 10 minutes of inactivity
- [ ] Audit logs capture logon events in Event Viewer

---

## Screenshots
<img width="1024" height="768" alt="VirtualBox_DC01-NYC_12_12_2025_11_56_40" src="https://github.com/user-attachments/assets/d9a2a7cc-5b9a-49c6-8fc5-7f1806786a83" />
Description: OU Structure in Active Directory Users and Computers

---
<img width="1024" height="768" alt="VirtualBox_DC01-NYC_12_12_2025_12_20_46" src="https://github.com/user-attachments/assets/2f9198dc-0a10-46f7-a0ce-6dd967760ddf" />

---
<img width="1024" height="768" alt="VirtualBox_DC01-NYC_12_12_2025_12_21_50" src="https://github.com/user-attachments/assets/59e44d68-4e94-48ac-8282-df3e607c7ce0" />

Description: User properties showing department and title

---
<img width="1024" height="768" alt="VirtualBox_DC01-NYC_12_12_2025_12_30_33" src="https://github.com/user-attachments/assets/4fc8f90f-528e-4a73-90fc-2c7ddf0c4adc" />

---
<img width="1024" height="768" alt="VirtualBox_DC01-NYC_12_12_2025_12_31_53" src="https://github.com/user-attachments/assets/26fcb82a-4d4e-44e4-8ed4-9b6f32d2ddf8" />

---
<img width="1024" height="768" alt="VirtualBox_DC01-NYC_12_12_2025_12_32_57" src="https://github.com/user-attachments/assets/3dc83581-b4a8-4d53-aaff-3ee87129366b" />

Description: Security group membership

---
<img width="1024" height="768" alt="VirtualBox_DC01-NYC_12_12_2025_14_13_50" src="https://github.com/user-attachments/assets/a3e63bd5-8ced-4e77-9b61-b25aae3cb8a3" />

<img width="1024" height="768" alt="VirtualBox_DC01-NYC_12_12_2025_13_04_56" src="https://github.com/user-attachments/assets/d65305ec-b963-4d00-b67b-dfc715be585a" />

<img width="1024" height="768" alt="VirtualBox_DC01-NYC_12_12_2025_13_09_20" src="https://github.com/user-attachments/assets/e63eb991-775d-4f39-87c0-eb0dc1a77249" />

<img width="1024" height="768" alt="VirtualBox_DC01-NYC_12_12_2025_13_12_16" src="https://github.com/user-attachments/assets/6dc9f3e2-8bd3-47f6-83dd-a1377a2cf76a" />

<img width="1024" height="768" alt="VirtualBox_DC01-NYC_12_12_2025_13_16_15" src="https://github.com/user-attachments/assets/378f37e7-9741-4a58-9506-89aa0c99df18" />

<img width="1024" height="768" alt="VirtualBox_DC01-NYC_12_12_2025_13_22_30" src="https://github.com/user-attachments/assets/982cba44-1630-464d-a93d-df1bbef14582" />

Description: Group policies

---
<img width="1024" height="768" alt="VirtualBox_DC01-NYC_12_12_2025_13_52_54" src="https://github.com/user-attachments/assets/d84a086e-663d-4818-b182-6ebb37733af3" />

 Delegation of Control Wizard completion

---

<img width="1024" height="768" alt="VirtualBox_DC01-NYC_12_12_2025_13_48_15" src="https://github.com/user-attachments/assets/df3f5d9e-934e-412a-a29c-6b530366d38d" />

Description: gpresult output showing applied policies

---

## Future Enhancements

This on-premises Active Directory environment serves as the foundation for:

- **Hybrid Identity Integration** — Azure AD Connect synchronization with Microsoft Entra ID
- **Conditional Access** — Cloud-based access policies for hybrid users
- **Azure AD Join** — Device management through Microsoft Intune
- **Single Sign-On** — Seamless access to cloud applications

See the companion project: [The Merchandise Vault — Hybrid Identity Infrastructure](https://github.com/Nkd-hue/Enterprise-Hybrid-Identity-Infrastructure)

---

## Technologies Used

- Windows Server 2022
- Active Directory Domain Services (AD DS)
- Group Policy Management Console (GPMC)
- Active Directory Users and Computers (ADUC)


---

## Author

**Nadia Kroduah**  
Identity and Access Management Professional  
[LinkedIn](www.linkedin.com/in/nadia-kroduah)

---

## License

This project is for educational and portfolio demonstration purposes.
