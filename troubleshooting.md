# Troubleshooting Log

This log documents issues encountered during the lab build and how they were resolved. Real-world IT environments always involve troubleshooting — documenting these problems and solutions is part of professional practice.

---

## Issue 1 — Ubuntu VM network autoconfiguration failed

**Phase:** 1 — VM Setup

**Issue:** During Ubuntu Server installation, the network configuration screen showed `eth0` as disabled with "autoconfiguration failed". The VM could not obtain an IP address via DHCP.

**Cause:** The LabSwitch virtual switch in Hyper-V is an Internal switch with no external DHCP server reachable during the early boot phase of the installer. The VM came up before it could get a DHCP lease from DC01.

**Fix:** Configured a static IP manually in the Ubuntu installer instead of relying on DHCP. This is best practice for servers regardless — static IPs ensure the server is always reachable at a known address.

| Field | Value |
|---|---|
| Address | 192.168.10.6/24 |
| Gateway | 192.168.10.254 (pfSense) |
| DNS | 192.168.10.1, 192.168.10.2 (DC01, DC02) |

---

## Issue 2 — Ubuntu archive mirror unreachable during install

**Phase:** 1 — VM Setup

**Issue:** During installation the Ubuntu archive mirror check failed with "Temporary failure resolving 'archive.ubuntu.com'".

**Cause:** At the point the installer checks the mirror, the network had just been configured with a static IP. The DNS servers (DC01/DC02) were not yet fully reachable, so the domain name could not be resolved.

**Fix:** Clicked "Continue" to proceed with installation from the ISO only, skipping the mirror check. After the system was fully booted and DNS was confirmed working, `apt update && apt upgrade` was run to pull all updates.

**Lesson:** Mirror check failures during installation are not critical — the system installs from the ISO and can be updated post-install.

---

## Issue 3 — osTicket download URL returned 404

**Phase:** 3 — osTicket Installation

**Issue:** The `latest` download redirect for osTicket returned a 404 error:
```
wget https://github.com/osTicket/osTicket/releases/latest/download/osTicket-v1.18.zip
ERROR 404: Not Found
```

**Cause:** The GitHub `latest` redirect pointed to `v1.18.3` but the actual release asset had a different filename format than expected.

**Fix:** Used a direct URL to the specific release version instead of the `latest` redirect:
```
wget https://github.com/osTicket/osTicket/releases/download/v1.18.2/osTicket-v1.18.2.zip
```

**Lesson:** When downloading from GitHub releases, always verify the exact filename in the releases page rather than relying on `latest` redirects.

---

## Issue 4 — php-imap not available on Ubuntu 26.04

**Phase:** 2 — LAMP Stack

**Issue:** `sudo apt install php-imap` returned:
```
Error: Package 'php-imap' has no installation candidate
```

**Cause:** Ubuntu 26.04 LTS changed how the IMAP extension is packaged. `php-imap` is no longer available as a standalone package in the default repositories.

**Fix:** Skipped `php-imap` during initial installation. The osTicket web installer flagged this as a missing recommended extension (not required). The missing extension only affects email-to-ticket functionality — core osTicket operation and AD integration are not impacted.

**Status:** Known limitation. Email-to-ticket via IMAP is not configured in this lab.

---

## Issue 5 — LDAP plugin incompatible with PHP 8.5

**Phase:** 4 — Active Directory Integration

**Issue:** Installing the osTicket LDAP Authentication plugin from the official osTicket plugins repository caused a 500 Internal Server Error. The Apache error log showed:

```
PHP Fatal error: Class LDAPAuthentication contains 5 abstract methods and must 
therefore be declared abstract or implement the remaining methods 
(AuthenticationBackend::login, AuthenticationBackend::getUser, 
AuthenticationBackend::getAllowedBackends, ...)
```

**Cause:** Ubuntu 26.04 LTS ships with PHP 8.5.4, which enforces stricter abstract class implementation rules than older PHP versions. The osTicket LDAP plugin was written for PHP 7.x/8.1 and has not been updated to support PHP 8.5. Multiple breaking changes were identified:

- `LDAPAuthentication` class missing required abstract method implementations
- `autodiscover()` method needed to be declared static
- `Net_LDAP2::PEAR()` dependency incompatible with PHP 8.5

**Attempted fixes:**
1. Added missing abstract method stubs to `LDAPAuthentication` class — resolved initial error but revealed further incompatibilities
2. Made `autodiscover()` static — resolved second error but revealed `Net_LDAP2` library issue
3. Attempted to install PHP 8.2 via `ondrej/php` PPA — PPA does not yet support Ubuntu 26.04

**Root cause:** The `Net_LDAP2` library used by the plugin relies on the PEAR framework which was deprecated and removed in PHP 8.0+. This is a fundamental incompatibility that cannot be patched without replacing the entire LDAP library.

**Resolution:** The LDAP plugin configuration was completed and saved (domain, DNS, bind DN, search base all configured correctly). The plugin cannot fully function due to the PHP 8.5/Net_LDAP2 incompatibility. This is a known upstream issue pending resolution by the osTicket team.

**What was documented:**
- LDAP plugin installed and enabled in osTicket
- Instance created: "TestNet Active Directory"
- Configuration saved:
  - Default Domain: `TestNet.Domain`
  - DNS Server: `192.168.10.1` (DC01)
  - Search User: `CN=osadmin,CN=Users,DC=TestNet,DC=Domain`
  - Search Base: `DC=TestNet,DC=Domain`

**Lesson:** When deploying on a brand new OS release, always check upstream project compatibility with the new runtime versions. Ubuntu 26.04 with PHP 8.5 is ahead of many open source projects. In a production environment, Ubuntu 24.04 LTS with PHP 8.2 would be the safer choice for maximum compatibility.

---

## Format for future entries

**Issue:** What went wrong  
**Cause:** Why it happened  
**Fix:** What resolved it  
**Lesson:** What to take away from it