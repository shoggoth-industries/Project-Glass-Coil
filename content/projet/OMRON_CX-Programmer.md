---
title: "CX-Programmer — Simple Password Recovery"
date: 2024-04-22
draft: false
description: "A reverse engineering walkthrough showing how CX-Programmer stores Function Block source passwords in plaintext memory, and how to recover them using x32dbg, IDA Pro, and HxD."
tags: ["reverse engineering", "ICS", "SCADA", "PLC", "debugging", "MFC", "x32dbg", "IDA Pro"]
categories: ["Security Research"]
showToc: true
TocOpen: false
cover:
  image: "/omron/password-dialog.png"
  alt: "CX-Programmer password dialog"
  caption: "The Function Block password dialog in CX-Programmer"
---

# CX-Programmer — Simple Password Recovery

> **TL;DR** — CX-Programmer (Omron's PLC programming IDE) stores Function Block source passwords in **plaintext** in process memory. By placing a single breakpoint on `DoModal()` in x32dbg, the password becomes immediately visible. A memory dump + `grep` is all that's needed for offline recovery.

---

## Tools Used

- **x32dbg** — 32-bit Windows debugger
- **HxD** — Hex editor
- **IDA Pro** — Interactive disassembler

---

## Step 1 — Initial Binary Analysis with Detect It Easy

Before diving into dynamic analysis, we run a quick static inspection of the binary to identify the DLLs it imports.

![Detect It Easy analysis of CX-Programmer](/omron/detect-it-easy.png)
*Detect It Easy showing CX-Programmer is a 32-bit PE compiled with MSVC and MFC 4.2*

Key findings:

- CX-Programmer uses **MFC libraries** (`MFC42.DLL`) for its UI.
- The binary is compiled as **32-bit (x86)**.
- The compiler is **Microsoft Visual C++ 12.00** and the linker is **Microsoft Linker 6.00**.

---

## Step 2 — Identifying the Password Dialog

The feature we are interested in is the password protection on Function Block source files. Let's understand how the password input is captured by CX-Programmer.

![CX-Programmer password input dialog](/omron/password-dialog.png)
*The "Disable Function Block Protection" modal dialog prompting for a password*

This is a **modal dialog** — it blocks the rest of the application and forces the user to provide input. In C++ with MFC (`mfc42.dll`), such dialogs are typically created using the `DoModal()` method of the `CDialog` class, which Microsoft documents as a common base class.

We can therefore place an initial **breakpoint** on this method in x32dbg to identify which function will process our input. We are almost certainly dealing with a **class derived from `CDialog`** designed for this specific dialog.

Before moving to x32dbg, it is worth noting that clicking the **"Disable"** button calls `CDialog::OnOK()`, which handles the input processing logic (and therefore the password verification).

---

## Step 3 — Mapping the Execution Flow

Based on the MFC architecture, the expected execution chain — starting from the derived class — looks like this:

![Execution flow diagram from button click to password verification](/omron/execution-flow.png)
*Call flow: user clicks "Disable" → `CDialogDeactivate::gestion_bouton` → `CDialog::DoModal` → `CDialogDeactivate::OnOk()` → password check result*

The flow can be summarised as:

1. User clicks the **"Disable"** button.
2. `CDialogDeactivate::gestion_bouton` creates the modal window.
3. `CDialog::DoModal(this_CDialogDeactivate)` calls the parent's `DoModal`.
4. Once the user clicks OK, `DoModal` dispatches to `CDialogDeactivate::OnOk()` via the vtable.
5. `OnOk()` branches to either a **correct** or **incorrect** password outcome.

---

## Step 4 — Finding `DoModal` via IDA Pro FLIRT Signatures

To place a breakpoint on `DoModal` in x32dbg, we need to locate it in the import table of `mfc42.dll`. The problem: **CX-Programmer imports this library by ordinal, not by name**. x32dbg only shows numeric indices instead of function names.

**IDA Pro** solves this elegantly using its **FLIRT** (*Fast Library Identification and Recognition Technology*) signatures. FLIRT maps the ordinal numbers back to their original function names, letting us identify the exact ordinal for `DoModal`:

```
; Import by ordinal 2514
__declspec(dllimport) public: virtual int __thiscall CDialog::DoModal(void)
extrn imp_?DoModal@CDialog@@UAEHXZ:dword
```

`DoModal` corresponds to **ordinal #2514** in `mfc42.dll`.

---

## Step 5 — Setting the Breakpoint in x32dbg

We can now search for this ordinal in x32dbg's import table and place a breakpoint on it.

Back in CX-Programmer, we click the **"Disable"** button (the password had been previously set to `"mayeul"` for this demonstration). The breakpoint on `DoModal` fires exactly when the input dialog is about to appear.

![x32dbg disassembly showing password "mayeul" visible in memory](/omron/debugger-memory.png)
*x32dbg paused at DoModal — the password `"mayeul"` is immediately visible in the comments column as a plaintext string*

> 🔑 **Key finding:** The password stored in the file appears **in plaintext** in process memory at this point. No decryption, no hashing — just a raw string.

---

## Step 6 — Hardware Breakpoint on the Password Buffer

To go further and understand the verification mechanism, we place a **hardware breakpoint on memory access** to the buffer holding the password. This lets us pinpoint exactly which function reads it.

![x32dbg hardware breakpoint panel showing active access breakpoint](/omron/hardware-breakpoint.png)
*Hardware breakpoint configured on the password buffer address — triggered on byte access*

After entering a test password in the dialog, the hardware breakpoint fires.

---

## Step 7 — Analysing the Password Comparison

The verification turns out to be a straightforward **`mbscmp_l`** (multibyte string comparison) between the two strings:

![x32dbg disassembly showing mbscmp_l comparing "helloworld" and "mayeul"](/omron/mbscmp-comparison.png)
*`_mbscmp_l` called with `esi:"helloworld"` (entered password) and `edi:"mayeul"` (stored password) — a simple string comparison with no hashing*

Walking up the call stack reveals the method that has overridden `OnOk`:

![Call stack in x32dbg showing OnOk override with password strings visible](/omron/call-stack-onok.png)
*The overriding `OnOk` method: `[esi+64]:"mayeul"` (stored password) and `[esi+60]:"helloworld"` (input) are passed directly to `_mbscmp`*

---

## Step 8 — Offline Recovery via Memory Dump

Armed with this knowledge, we can recover the password without even needing to trigger the dialog. The workflow is:

1. Load the PLC project file in CX-Programmer.
2. **Dump the process memory** (e.g., using x32dbg's memory dump feature or tools like `procdump`).
3. Open the dump in **HxD** (or run `grep`) and search for the password string.

![HxD hex editor showing "mayeul" password in plaintext in the memory dump](/omron/hxd-memory-dump.png)
*HxD view of the memory dump: `PasswordString="maye` followed by `ul"` is clearly visible in the decoded text column — the password stored in plaintext*

A simple text search is sufficient. No brute-force, no cryptanalysis.

---

## Going Further

### 1. Reversing the PLC Backup File Format

A more elegant (and CX-Programmer-independent) approach would be to **reverse-engineer the PLC backup file format** (`.cxp` or similar) directly. If the password is stored as-is in the file — which the memory dump strongly suggests — a Python parser or hex template in HxD would let you extract credentials without ever launching CX-Programmer.

### 2. Differential Execution Trace with DynamoRIO

For a more **systematic** approach to locating the verification function without manual breakpoint hunting, generate execution traces using **DynamoRIO**:

1. Record a trace with the **correct** password.
2. Record a trace with an **incorrect** password.
3. Perform a **differential analysis** to isolate the instructions that diverge between the two runs.

This eliminates guesswork and scales well to more complex binaries where the password check is buried deep in the call chain.

---

## Summary

| Step | Action | Tool |
|------|--------|-------|
| Static analysis | Identify imported DLLs, architecture | Detect It Easy |
| Ordinal resolution | Map `mfc42.dll` ordinals to function names | IDA Pro (FLIRT) |
| Dynamic analysis | Breakpoint on `DoModal`, observe memory | x32dbg |
| Buffer tracing | Hardware breakpoint on password buffer | x32dbg |
| Verification analysis | Confirm `mbscmp_l` — no hashing | x32dbg |
| Offline recovery | Memory dump + hex search | HxD / grep |

**Root cause:** CX-Programmer stores Function Block source passwords as **plaintext strings** in memory and in the project file, comparing them with a simple `mbscmp_l`. There is no hashing, no encryption, and no obfuscation.

---

*This research was conducted for educational and security awareness purposes on a local installation of CX-Programmer.*