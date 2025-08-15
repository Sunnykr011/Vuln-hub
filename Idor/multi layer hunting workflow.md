# 0) Core stance

* **Prime belief:** Har request ek **claim of authority** hai. Server ko prove karna chahiye ki tum is object ke haqdar ho; tumhe prove nahi karna.
* **Rule:** Agar system tumhe object choose karne deta hai, to tum check karoge ki kya system ownership verify karta hai. Bas.

---

# 1) Surface Recon → “Where can I point the gun?”

Quick pass, noise ignore karo:

* **Enumerate features:** profile, orders, invoices, messages, files, tickets, notifications, analytics exports.
* **Find object nouns:** `userId`, `orderId`, `invoiceId`, `fileId`, `ticketId`, `orgId`, `workspaceId`, `teamId`.
* **Collect entrypoints:** Web + API. Browser devtools → `Fetch/XHR`; Burp → target sitemap.
* **Heuristic:** Jo feature “share,” “download,” “export,” “view details,” “switch account” bolta hai, waha IDOR probability high.

**Self-check:** Kya mere paas kam se kam 10 distinct object types list me aa gaye?

---

# 2) Map the Object Space → “Chishiya grid”

* **For each object type**, note:

  * Param names: `id`, `user`, `entityId`, `resource`, `node`.
  * Formats: int sequence, UUID v4, ULID, slug.
  * **Owner field** ka guess: `ownerId`, `createdBy`, `accountId`, `orgId`.
* **Role/Tenant matrix:** `Anonymous, User, Premium, Admin`, plus `Org A vs Org B`.
* **Method matrix:** GET, POST, PUT/PATCH, DELETE. Kabhi auth GET pe strong, POST pe weak hota.

**Self-check:** Kya mere paas 1 grid bani jisme rows = objects, columns = method x role x tenant?

---

# 3) Baseline Tampering → “uid=50 school kid test, but smarter”

* Replace `id=yourId` with:

  * **+1 / -1** sequential.
  * **Far jump**: `+500`, `+5000`.
  * **Nonexistent**: absurd big number.
  * **Format-keep**: UUID same length but 2–3 chars change.
* **Bulk endpoints:** `ids=[yourId, victimId]` payload me mix karo. Kabhi 1 unauthorized slip ho jata.
* **Path params too:** `/users/50`, `/files/abc`, not just query.

**Self-check:** Kya maine list endpoints aur detail endpoints dono pe tamper kiya?

---

# 4) Non-CRUD Surfaces → “Where devs forget checks”

* **Downloads/Exports:** `/download`, `/export/csv`, `/report.pdf?id=...`
* **Previews/Thumbnails:** `/thumbnail?id=`, `/preview?fileId=`
* **Webhooks/Logs:** `/events`, `/audit`, `/activity?userId=...`
* **Notifications/Inbox:** `/messages?threadId=...`
* **Search:** `/search?q=&ownerId=` or hidden filters.
* **Analytics dashboards:** `/metrics?orgId=...`
* **Weak HEAD/OPTIONS:** Same path different method.

**Self-check:** Kya download ya export endpoints par exact same ID tampering lagayi?

---

# 5) Indirect Reference & Leaky Joins → “Camus pattern in chaos”

* **Two-step flows:**

  1. `/list` se kisi aur ka `fileName`/`slug` leak,
  2. `/download?fileName=leaked` se pull.
* **Join mismatches:** `/comments?postId=X` pe tum doosre user ke comments dekh sako, kyunki check sirf post pe tha, comment pe nahi.
* **Token wrappers:** Short-lived token banata hai but **object ownership validate nahi** karta.

**Self-check:** Kya maine 2-step chain test kiya: list → action?

---

# 6) State/Workflow Breaks → “Nietzschean will to power”

* **Draft vs Published:** Draft objects par checks kam.
* **Soft-delete/Archive:** Archived objects access control loose.
* **Impersonation modes:** “Switch user,” “login as,” “support view” leaks.
* **Migrations/Legacy endpoints:** `/v1` secure, `/legacy` aadha-dead.

**Self-check:** Kya maine non-happy-path states touch kiye (draft, archived, pending)?

---

# 7) Verb/Method Swaps → “Function-level IDOR”

* Same endpoint, different methods:

  * GET secure, PUT unsafe.
  * DELETE pe owner check missing.
* **Edge verbs:** `PATCH`: partial updates often skip full auth.

**Self-check:** Kya maine har method pe same object test kiya?

---

# 8) Mass Assignment Blend

* Hidden fields inject karo: `ownerId`, `isAdmin`, `orgId`.
* Agar update request me ye accept ho gaya, to tum **ownership hijack** karke IDOR ko elevate kar sakte ho.

**Self-check:** Kya update forms me unexpected fields bheje?

---

# 9) Multi-tenant Walls → “Kira’s moral exploit”

* `orgId`/`workspaceId` swap: Org A user se Org B data.
* Many apps auth check me sirf userId verify karte hain, **org boundary** bhool jaate.

**Self-check:** Kya maine cross-org tampering ki?

---

# 10) Bulk Import/Sync/Integrations

* CSV import, third-party sync endpoints.
* Batched actions: “process 100 items” me ek unauthorized item slip.
* Background jobs: queued tasks trust kar lete hain caller ko.

**Self-check:** Kya maine batch payloads me mixed-ownership inject kiya?

---

# 11) Evidence Patterns → “Absurd hints”

Signals that auth is flaky:

* “404 Not Found” jab “403 Forbidden” expected tha.
* Kabhi-kabhi same tamper pe kabhi data aata, kabhi deny hota.
* Rate limits error instead of auth errors.
* Error objects me sensitive metadata: `ownerId: 1234`.

**Self-check:** Kya maine error text ko data source treat kiya?

---

# 12) Automation-lite, without killing brains

* **Burp Repeater macros:** Same request variants queue me.
* **Param Miner:** Hidden params discover.
* **Intruder with brain:** Low volume, object sets curated from recon.
* **JS scraping:** App JS se param names, endpoints pull.

**Self-check:** Kya mere variations smart hain, ya main bas brute nudging kar raha hu?

---

# 13) Reporting mindset → “Best-version Kael”

* **Minimum PoC:** show access to someone else’s non-public field, ya proof via hash of content.
* **Impact frame:** privacy, financial, lateral movement, compliance.
* **Fix suggetsion:** “Enforce per-request ownership check: currentUser must match resource.ownerId; verify org boundary; unify policy via middleware.”

**Self-check:** Kya PoC reproducible + ethical boundary clear hai?

---

# 14) Future-proof: when AI cleans the basics

Even if `uid=51` dries up:

* **Workflows will always drift.** New features, rushed checks.
* **Authorization centralization lag.** Middleware miss on niche endpoints.
* **Cross-object joins** and **batch jobs** remain fragile.
* **Preview/Export/Backup** surfaces stay forgotten.

**Daily drill:** Har new feature me ye 6 angles auto-run:

1. Object map + owner field
2. Role/tenant cross-check
3. Method variant
4. List→action chain
5. Bulk/batch misuse
6. Draft/archive state poke

---

## Tiny playbook you can run in any app in 20–30 minutes

1. Devtools XHR open, browse every feature once, dump endpoints to notes.
2. Pick 3 object types with the most activity.
3. For each, do: detail GET tamper, list mix, download/export try, and one update with hidden field.
4. Swap org/team if app supports it.
5. Try one weird method (PATCH/DELETE).
6. Record only anomalies; ignore legit 403.
7. Revisit anomalies with role switch.

---

## Kael’s “best version” cues

* **Calm scan > frantic brute.** Tum pattern hunter ho, hammer nahi.
* **Map first, stab later.** 10 minute grid saves 2 hours noise.
* **Chain-building mindset.** Ek mini-leak ko 2-step me exploitable banao.
* **Ethics + elegance.** Clean PoC, minimal data touch, maximum clarity.

---

## Philosophical overlays, quick anchors

* **Chishiya:** “Game board first.” Make the grid, then move.
* **Nietzsche:** “Weak checks deserve conquest.” If a rule isn’t defended, it isn’t real.
* **Camus:** “Expect absurdity.” Don’t trust consistency; verify it.
* **Kira:** “Access defines power.” Focus where unauthorized access grants leverage, not trivia.

---

## Mini practice seeds (use anywhere)

* Try `ids` array mixing on any batch endpoint you find.
* Change `workspaceId` on any search or analytics call.
* Replace a filename from a listing into a download endpoint.
* Send `ownerId` in an update payload and see if the object switches homes.
* Call the same path with DELETE or PATCH once.
