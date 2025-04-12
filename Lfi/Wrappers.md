Bhai ab aa gaya tu **LFI Wrappers** wale real hacker jungle mein — yahaan pe normal log confuse ho jaate hain, lekin tu sab modes se samjhega toh kabhi bhoolega nahi.

---

## **“Wrappers” kya hote hain?**

### **Sunny Mode – “Straight & Simple”**

> **Wrappers = Special protocols jisse PHP files ko alag tarike se process karta hai.**  
> Jaise:
> - `php://`  
> - `data://`  
> - `expect://`

**Soch: Tu normal file read nahi kar raha, tu ek “trick” use kar raha to *wrap* the process.**  
Jaise: Base64 me file chhupa ke mangna.

---

### **Chishiya Mode – “Logic behind Wrappers”**

> Wrappers = PHP ke *virtual protocols*  
> Inka kaam: File ke saath kuch extra kaam karna.

**Examples:**
- `php://filter/convert.base64-encode/resource=config.php`  
  → Yeh kya karta hai? `config.php` file ko **base64** encode karke deta hai — Taki tu PHP code dekh sake bina execute kare.

- `data://text/plain;base64,PD9waHAgc3lzdGVtKCdscycpOz8+`  
  → Tu base64 encoded PHP code ko input me hi bhej deta hai → Execution.

---

### **Glitch Mode – “Protocol Ka Glitch Abuse”**

> Wrappers = **Normal file access ka alternate universe.**  
> `php://input` → Read POST body  
> `php://memory` → RAM use  
> `php://stdin` → Input stream

**Hacking style:**  
> “Server kuch aur expect kar raha, tu wrapper daal ke uska brain confuse karta hai.”

---

### **Arisu Mode – “Why Do Wrappers Exist?”**

> Devs ne wrappers banaye the **convenience ke liye.**  
> Taki logs ya base64 handle ho sake.  
> Lekin jab tu unhe attacker ki tarah soch ke use karta hai, **yei dev ka trust teri victory ban jaata hai.**

---

### **Gojo Mode – “Wrappers = Hidden RCE Gateways”**

> Most dangerous wrappers:
> - `php://input` → Log poisoning ke saath use karo  
> - `expect://` → Direct system command run karta hai  
> - `data://` → Remote shell injection via base64

**Chain Reaction:**  
LFI → Wrapper → File Inclusion → Payload Execution → RCE

---

### **Kira Mode – “Wrappers = Hidden Blades”**

> Wrapper ka real power tab dikhta hai jab sabko lagta hai LFI waste hai.  
> Tu `php://filter` se config file ka secret le aata hai,  
> `data://` se shell inject karta hai.  
> Most people give up at 403 — Kira starts hunting.

---

### Quick Cheatsheet:

| Wrapper         | Purpose                         |
|----------------|----------------------------------|
| `php://filter` | Encode/decode file output        |
| `php://input`  | Read raw POST data               |
| `data://`      | Inject base64-encoded PHP shell  |
| `expect://`    | Run shell commands (rare cases)  |
| `zip://`       | Access zip file content remotely |

---

### Tera memory-lock mantra:

> "**Wrappers = Jaadu ke tunnel hai, jahan se tu server ko confuse karke apna kaam nikaal leta hai.**"  
> Simple file read → **Wrapper daala** → Advanced read, payload, ya bypass ho gaya.

---

Agla kaunsa samjhaun?  
`php://filter` deep dive?  
`../` traversal ka flow?  
`require vs include`?  
Bol aur next gate open karte hain.
