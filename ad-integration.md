# Active Directory Integration

This page documents the attempt to integrate osTicket with the existing Active Directory domain (`TestNet.Domain`) using LDAP authentication.

---

## Overview

In a real enterprise helpdesk environment, agents and users authenticate against Active Directory rather than maintaining separate credentials in the ticketing system. osTicket supports this through its LDAP Authentication and Lookup plugin, which connects to AD over LDAP (Lightweight Directory Access Protocol) — the standard protocol for querying directory services.

**Planned architecture:**

| Component | Detail |
|---|---|
| Protocol | LDAP (port 389) |
| Directory | Active Directory on TestNet.Domain |
| DC01 | 192.168.10.1 — primary LDAP target |
| DC02 | 192.168.10.2 — secondary DC |
| Bind account | CN=osadmin,CN=Users,DC=TestNet,DC=Domain |
| Search base | DC=TestNet,DC=Domain |

---

## What Was Completed

### Step 1 — Download and install the LDAP plugin

The LDAP plugin is not bundled with osTicket v1.18 — it must be downloaded separately from the osTicket plugins repository.

```bash
cd /tmp
wget https://github.com/osTicket/osTicket-plugins/archive/refs/heads/master.zip
unzip master.zip -d osticket-plugins
sudo cp -r /tmp/osticket-plugins/osTicket-plugins-master/auth-ldap \
           /var/www/html/osticket/include/plugins/
sudo chown -R www-data:www-data /var/www/html/osticket/include/plugins/
```

**Why copy to the plugins folder:** osTicket scans `include/plugins/` on startup and makes any plugin found there available for installation in the Admin Panel.

### Step 2 — PHP 8.5 compatibility fix

Ubuntu 26.04 ships with PHP 8.5, which enforces stricter object-oriented rules than the plugin was written for. The plugin required patching to add missing abstract method implementations before it would load without errors.

See [Troubleshooting — Issue 5](troubleshooting.md#issue-5--ldap-plugin-incompatible-with-php-85) for the full breakdown of what was fixed and why.

### Step 3 — Install and enable the plugin

In osTicket Admin Panel → **Manage → Plugins → Add New Plugin:**

- LDAP Authentication and Lookup plugin appeared after copying to the plugins folder
- Clicked **Install**
- Enabled the plugin via the checkbox → **More → Enable**

### Step 4 — Create an instance and configure AD settings

Admin Panel → **Manage → Plugins → LDAP Authentication and Lookup → Instances → Add New Instance**

**Instance settings:**
| Field | Value |
|---|---|
| Name | TestNet Active Directory |
| Status | Enabled |

**Config tab — AD connection settings:**
| Field | Value |
|---|---|
| Default Domain | TestNet.Domain |
| DNS Servers | 192.168.10.1 |
| Search User | CN=osadmin,CN=Users,DC=TestNet,DC=Domain |
| Password | (bind account password) |
| Search Base | DC=TestNet,DC=Domain |
| LDAP Schema | Automatically Detect |

**Why these settings:**

**Default Domain** — the AD domain users belong to. With this set, users can log in with just their username (`jsmith`) rather than the full UPN (`jsmith@testnet.domain`).

**DNS Servers** — points to DC01 so the plugin can resolve `TestNet.Domain` to find the AD server. This is needed because the plugin uses DNS SRV records to autodiscover domain controllers.

**Search User (Bind DN)** — the account osTicket uses to connect to AD and run LDAP queries. In production this would be a dedicated service account with read-only access to AD. The Distinguished Name (DN) format `CN=name,CN=container,DC=domain,DC=tld` is how LDAP identifies objects in the directory tree.

**Search Base** — the point in the AD directory tree where LDAP searches begin. `DC=TestNet,DC=Domain` means search the entire domain, which ensures all users in any OU can be found.

---

## Known Limitation — PHP 8.5 / Net_LDAP2 Incompatibility

The configuration was saved successfully, however the plugin cannot fully function due to a compatibility issue between the `Net_LDAP2` library (used by the plugin for LDAP communication) and PHP 8.5.

`Net_LDAP2` depends on the PEAR framework which was deprecated and removed in PHP 8.0. This is an upstream issue with the osTicket plugins repository that requires the plugin to be rewritten to use a modern LDAP library.

**Error encountered:**
```
Call to undefined method Net_LDAP2::PEAR()
```

**Impact:** Domain user authentication via LDAP does not work. osTicket falls back to local authentication (username/password stored in the osTicket database).

**Resolution path:** This issue will be resolved when either:
- The osTicket team updates the plugin for PHP 8.x compatibility, or
- The lab is rebuilt on Ubuntu 24.04 LTS (PHP 8.2) where the plugin works correctly

Full troubleshooting detail: [Troubleshooting — Issue 5](troubleshooting.md#issue-5--ldap-plugin-incompatible-with-php-85)

---

## How AD Integration Works (Conceptual)

Even though the plugin could not connect in this environment, understanding what it does is important for the portfolio context.

When a user logs into osTicket with AD integration working:

1. User enters their AD username and password on the osTicket login page
2. osTicket passes the credentials to the LDAP plugin
3. The plugin connects to DC01 on port 389 using the bind account
4. It searches the directory under `DC=TestNet,DC=Domain` for a user matching the username
5. It attempts to authenticate (bind) using the user's credentials
6. If successful, AD returns the user's attributes (name, email, group memberships)
7. osTicket maps those attributes to an agent or user account
8. The user is logged in without needing a separate osTicket password

This means:
- Agents use their existing Windows domain credentials
- Password changes in AD automatically apply to osTicket
- When a user is disabled in AD, they immediately lose osTicket access
- No password sprawl — one set of credentials for the whole environment

---

➡️ [Continue to Configuration](configuration.md)