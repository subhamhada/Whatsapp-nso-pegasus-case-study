# Case Study Centred on the 2019 WhatsApp–NSO Group / Pegasus Incident

**End-to-End Encryption: Security Mechanism, Authentication and Non-Repudiation in Modern Communication (Unit 4)**

**Subject:** Cyber Security | **Type:** Case Study Report | **CVE:** CVE-2019-3568 | **Batch:** B.Tech 2023–27, Semester VII | **License:** Academic Use

**Symbiosis Institute of Technology, Pune**
Constituent of Symbiosis International (Deemed University), Pune
Department of Electronics & Telecommunication Engineering

---

## Table of Contents

1. [Project Information](#1-project-information)
2. [Abstract](#2-abstract)
3. [Keywords](#3-keywords)
4. [Introduction](#4-introduction)
5. [WhatsApp E2EE Architecture and Cryptographic Mechanics](#5-whatsapp-e2ee-architecture-and-cryptographic-mechanics)
6. [Methodology and Evidence Triangulation](#6-methodology-and-evidence-triangulation)
7. [Case Study: The 2019 Zero-Click Exploit (CVE-2019-3568)](#7-case-study-the-2019-zero-click-exploit-cve-2019-3568)
8. [Preventive Measures and Defensive Engineering](#8-preventive-measures-and-defensive-engineering-hardening)
9. [Results, Discussion and Uncertainty Assessment](#9-results-discussion-and-uncertainty-assessment)
10. [Legal, Ethical and Accountability Context](#10-legal-ethical-and-accountability-context)
11. [Conclusion and Future Scope](#11-conclusion-and-future-scope)
12. [Repository Structure](#12-repository-structure)
13. [How to Use This Repository](#13-how-to-use-this-repository)
14. [Tools, Standards and Technologies Referenced](#14-tools-standards-and-technologies-referenced)
15. [References](#15-references)
16. [How to Cite](#16-how-to-cite)
17. [Acknowledgements](#17-acknowledgements)
18. [License and Disclaimer](#18-license-and-disclaimer)
19. [Contact](#19-contact)

---

## 1. Project Information

| Field | Details |
|-------|---------|
| **Title** | Case Study Centred on the 2019 WhatsApp–NSO Group / Pegasus Incident |
| **Subject** | Cyber Security |
| **Topic selected** | End-to-End Encryption: Security Mechanism, Authentication and Non-Repudiation in Modern Communication (Unit 4) |
| **Group No.** | 15 |
| **Faculty** | Dr. Mangal Singh |
| **Department** | Electronics & Telecommunication Engineering |
| **Program / Batch** | B.Tech, Batch 2023–27, Semester VII |
| **Institute** | Symbiosis Institute of Technology, Pune |

### Group Members

| Name | PRN |
|------|-----|
| Prafful Kulkarni | 23070123099 |
| Saanvi Mohapatra | 23070123110 |
| Sai Sankar Jena | 23070123112 |
| Subham | 23070123132 |

---

## 2. Abstract

End-to-end encryption is the most secure way of communicating over public cellular and IP networks. Data confidentiality is achieved by applying cryptographic transformations to the data only at the two end nodes of the communication, which prevents attacks by the communications infrastructure, relays, carriers and eavesdroppers. With over two billion registered users worldwide, WhatsApp's end-to-end encryption relies on Signal's Extended Triple Diffie-Hellman (X3DH) protocol for key exchange and the Double Ratchet protocol for message confidentiality and forward secrecy. However, these protections have been shown to be weak at the lower layers of the network stack.

This report examines WhatsApp's end-to-end encryption in detail, focusing mainly on memory-based attacks that can break the confidentiality of plaintext messages protected by Signal. It shows how WhatsApp's implementation was used to mount a remote code execution attack on a target device by causing a heap buffer overflow in `libwhatsapp.so` while it processed specially crafted Secure Real-time Transport Control Protocol (SRTCP) packets. With this flaw, an attacker could read the memory of the targeted WhatsApp process on iOS or Android and obtain plaintext messages not otherwise accessible.

The report also discusses entity authentication, AES-GCM message integrity, deniability of content and legal non-repudiation, and examines how WhatsApp's defense-in-depth design principles help mitigate attacks on the encryption infrastructure. These range from memory-safe Rust programs to process isolation within the operating system, fuzzing, and hardware isolation of cryptographic keys.

---

## 3. Keywords

`WhatsApp End-to-End Encryption` · `Signal Protocol` · `X3DH Key Exchange` · `Double Ratchet` · `CVE-2019-3568` · `Pegasus Exploit` · `VoIP Stack` · `SRTCP Heap Overflow` · `Deniability` · `Legal Non-repudiation` · `Memory Safety` · `Defense in Depth` · `Rust Programming`

---

## 4. Introduction

### 4.1 Evolution of Messaging Security and WhatsApp E2EE Adoption

Communication has moved from unencrypted channels (SMS, early HTTP-based chat applications) to end-to-end encrypted communication. In early mobile communication, messages traveled through carrier networks in clear text and were protected only between the user's device and the carrier's server. Standard TLS protects data on the wireless link but exposes plaintext, multimedia and database content on central servers to operational inspection. A centralized architecture creates risks of large-scale data leakage, insider threats and government surveillance.

WhatsApp began collaborating with Open Whisper Systems (Signal) in 2014 to adopt the Signal Protocol across its clients (Android, iOS, Windows Phone, desktop). In April 2016 WhatsApp announced global E2EE covering one-to-one and group messaging, voice and video calls, media, file transfer and status updates for more than 2 billion users. Encryption and decryption happen only on the user's device, and WhatsApp's infrastructure acts as a ciphertext-only proxy. Security rests on the secrecy of session keys exchanged only between the communicating devices.

### 4.2 Problem Statement: The Protocol–Endpoint Execution Dichotomy

Modern application-layer cryptographic protocols have formal proofs, but there is a fundamental gap between the guarantees of a protocol and the safety of its software implementation on client hardware. Symbolic verification tools such as ProVerif and CryptoVerif reason about resistance to the classical **Dolev-Yao** adversary, which controls an insecure network while assuming the endpoints are secure.

In practice, client messaging applications are very large (tens of millions of lines of code) and written in low-level, unsafe languages such as C and C++. Incoming packets pass through socket listeners, transport framing, media decoding and signalling parsers *before* reaching the cryptographic layer. A parsing bug, such as an unchecked buffer overflow, lets an attacker craft packets that violate memory safety and achieve arbitrary code execution before any cryptographic processing. With code execution, the attacker can read the application's memory, leak plaintext protected by E2EE, and exfiltrate short-lived encryption keys and local database content, bypassing the cryptographic primitives entirely.

### 4.3 Scope, Methodological Boundaries and Evidence Sources

This is a multi-disciplinary security analysis of WhatsApp's E2EE implementation. It includes:

- Endpoint vulnerability vectors
- A mathematical walk-through of the Diffie-Hellman key agreement and Double Ratchet algorithms
- Analysis of WhatsApp's 60-digit safety number fingerprint scheme
- Examination of post-quantum (PQXDH / ML-KEM-768) extensions
- A low-level C/C++ analysis of the CVE-2019-3568 VoIP heap buffer overflow
- An assessment of the legal precedent set by *WhatsApp LLC v. NSO Group Technologies Ltd.*

The report is bounded by qualitative evidence synthesis. It is a secondary-source engineering case study built on the National Vulnerability Database (NVD), cryptographic specification reviews, Citizen Lab and Check Point Research analyses, and sworn federal court filings from the U.S. District Court for the Northern District of California. **No live zero-day exploit binaries were created and no commercial spyware samples were decompiled.** Where server telemetry is proprietary or confidential, the analysis notes this.

### 4.4 Cryptographic Deniability vs. Legal Non-Repudiation

| | Cryptographic deniability | Legal non-repudiation |
|---|---|---|
| **Nature** | Technical property | Legal concept |
| **Mechanism** | Message Authentication Codes (MACs) from a shared symmetric key | Asymmetric digital signatures (RSA / ECDSA) bound to HSMs, trusted PKI certificates and timestamping authorities |
| **Effect** | Either party could have generated a valid MAC tag, so a party can plausibly claim the other forged a transcript (off-the-record deniability) | The signer cannot deny the authenticity of a signed statement or transaction |
| **Used by** | WhatsApp and Signal (by design) | Systems that need provable authorship |

Privacy-focused messaging systems deliberately refuse to offer legal non-repudiation to avoid leaking message transcripts. However, in a court of law authorship can still be decided through contextual corroborating evidence such as a seized phone, metadata, carrier cell-tower logs and forensic images. Cryptographic deniability did not shield a person from legal liability when their phone was physically seized.

---

## 5. WhatsApp E2EE Architecture and Cryptographic Mechanics

### 5.1 Untrusted Relay Server Model and Data Flow

WhatsApp's network uses an **untrusted relay server** design. The central server handles user directory registration, public prekey bundles, phone number validation, offline message queues, push notification relays and packet routing. It never has access to plaintext or session keys.

**Alice to Bob message path:**

1. **Prekey retrieval:** If there is no active session, Alice fetches Bob's public prekey bundle from the server directory.
2. **Session initialization (X3DH):** Alice verifies the signed prekey signature, creates an ephemeral key pair and computes the X3DH agreement to obtain a master shared secret (S).
3. **Payload encryption:** The Double Ratchet generates a per-message key (MKi). The message is encrypted with AES-256-GCM, producing ciphertext plus a 128-bit authentication tag.
4. **Framing:** The encrypted envelope (ciphertext, tag, IV, ratchet information) is encapsulated in a Noise Protocol framework socket frame and sent over TCP or UDP to WhatsApp's servers.
5. **Server relay:** Servers read only unencrypted routing headers (recipient account ID). If Bob is offline, the envelope is queued; otherwise it is relayed over his push connection. The server cannot decrypt the payload.
6. **Client decryption and verification:** Bob updates his DH/KDF ratchet state from the header, verifies the AEAD tag, decrypts and displays the plaintext.

### 5.2 Extended Triple Diffie-Hellman (X3DH) Key Agreement

X3DH allows asynchronous session setup, so a message can be sent while Bob is offline. Each client holds key pairs derived from Curve25519 (or Ed25519 for signatures):

| Key | Description |
|-----|-------------|
| **Identity Key (IK)** | Long-term key pair specific to the user account |
| **Signed Prekey (SPK)** | Medium-term key pair, replaced periodically and signed by the Identity Key to prevent server-side forgery |
| **One-Time Prekeys (OPK)** | Single-use prekeys uploaded to the server and consumed at session start |
| **Ephemeral Key (EK)** | One-time key pair generated by the session initiator (Alice) |

Alice downloads Bob's bundle (IK_B, SPK_B, Sig_B, OPK_B), verifies Sig_B using Bob's public identity key, generates her ephemeral key EK_A, and computes:

```
DH1 = ECDH(IK_A, SPK_B)    [Mutual identity authentication]
DH2 = ECDH(EK_A, IK_B)     [Forward secrecy against Alice's IK leakage]
DH3 = ECDH(EK_A, SPK_B)    [Forward secrecy against ephemeral leakage]
DH4 = ECDH(EK_A, OPK_B)    [One-time prekey forward secrecy]

Master Shared Secret (S) = HKDF-SHA256(DH1 || DH2 || DH3 || DH4)
```

### 5.3 The Double Ratchet Algorithm

After X3DH establishes the initial keys, the Double Ratchet provides confidentiality of future messages, per-message forward secrecy and break-in recovery through two intertwined key-derivation loops:

- **Symmetric-key (KDF) ratchet:** Runs for every message. A KDF chain key (CK) exists for each sending and receiving direction. CKi is passed to HKDF-SHA256 to derive a one-time message key (MKi) and the next chain key (CKi+1). MKi is zeroized after encrypting one message, which gives per-message forward secrecy.
- **Diffie-Hellman ratchet:** Interactive, runs on each reversal of conversation direction. The sender's current ratchet public key is sent in the message header. When Bob receives a new ratchet public key from Alice, he performs a DH ratchet step, computes a new Root Key (RK), derives new sending and receiving chains, and generates a new local ratchet key pair.

If an adversary extracts one endpoint's chain key from memory, the DH ratchet ensures future communication becomes safe again once a new DH ratchet step occurs (**post-compromise security / break-in recovery**).

### 5.4 Cryptographic Suite and Security Objectives

| Primitive | Standard / Specification | Function in WhatsApp | Security objective and boundary |
|-----------|--------------------------|----------------------|---------------------------------|
| **Curve25519** | RFC 7748 / Bernstein | ECDH key agreement in X3DH and DH ratchet | 128-bit security; constant-time implementation resists timing side channels; vulnerable to quantum computers |
| **Ed25519** | RFC 8032 / FIPS 186-5 | Signatures for identity key authentication and prekey binding | Entity authentication; prevents server-side prekey substitution |
| **AES-256-GCM** | FIPS 197 / NIST SP 800-38D | Message payload encryption and AEAD tag | Payload confidentiality and tamper detection; bypassed if endpoint RAM is read |
| **HKDF-SHA256** | RFC 5869 / FIPS 180-4 | Extract-and-expand key derivation for master secrets, root keys, chain keys | Provably secure pseudorandom key expansion; cleanly separates key contexts |
| **HMAC-SHA256** | FIPS 198-1 / RFC 2104 | Per-message MAC and symmetric KDF chain ratcheting | Symmetric message authentication and off-the-record deniability |
| **ML-KEM-768 (PQXDH)** | NIST FIPS 203 / Kyber-768 | Post-quantum lattice key encapsulation combined with Curve25519 ECDH | Protects traffic against future retrospective decryption by quantum computers |

### 5.5 Security Fingerprints and 60-Digit Safety Numbers

To defend against man-in-the-middle key substitution by a compromised relay server, WhatsApp offers out-of-band identity verification through a **Security Code / Safety Number**. The 60-digit numeric fingerprint is derived from the lexicographically sorted public identity keys (IK_A and IK_B) of both parties. The concatenated keys are hashed with SHA-512, and the 512-bit hash is parsed into 5-digit blocks with a protocol version prefix.

Users verify by scanning a 2D QR code in person or by comparing the 60-digit string over an already trusted channel. If the server attempts key substitution, the strings will not match. Empirical usability studies cited in the report found that more than 90% of non-technical users ignore automatic key-change warnings, which is a significant human-factor risk.

### 5.6 Post-Quantum Extended Key Agreement (PQXDH and ML-KEM-768)

Because traffic can be recorded today and decrypted later by quantum computers ("harvest now, decrypt later"), the Signal specification now uses PQXDH. It is a hybrid of classical Curve25519 ECDH and a lattice-based key encapsulation mechanism, ML-KEM-768 (NIST FIPS 203, Kyber-768 based). A session is established with a classical ECDH operation plus an ML-KEM encapsulation:

```
Hybrid Shared Secret = HKDF-SHA256(DH1 || DH2 || DH3 || DH4 || ML-KEM_SS)
```

---

## 6. Methodology and Evidence Triangulation

### 6.1 Secondary-Source Qualitative Case Study Design

The report uses a qualitative, evidence-based cybersecurity case-study method adapted for zero-day exploitation and commercial surveillance. Direct dynamic analysis of offensive tools such as Pegasus is technically, operationally and legally difficult: the software is restricted by licensing, obfuscated, self-removing on detection, and protected by state-level defense secrecy laws.

### 6.2 Four-Tier Evidence Triangulation Framework

To support every technical claim and avoid confirmation bias, claims are cross-referenced against four tiers of sources:

| Tier | Purpose | Sources |
|------|---------|---------|
| 1 | Vulnerability records | NVD (CVE-2019-3568), WhatsApp Security Advisory, MITRE CWE-120 and CWE-787 |
| 2 | Protocol requirements | Signal and WhatsApp public specifications, NIST FIPS 203 and 204 |
| 3 | Attack-chain analysis | Check Point Research and Citizen Lab independent audits of Pegasus |
| 4 | Legal record | Court documents in *WhatsApp LLC v. NSO Group Technologies Ltd.* (Ninth Circuit; N.D. Cal.) |

---

## 7. Case Study: The 2019 Zero-Click Exploit (CVE-2019-3568)

### 7.1 Incident Context, Discovery Timeline and Global Impact

In May 2019, WhatsApp published a security advisory for **CVE-2019-3568**, an unauthenticated remote code execution (heap overflow) vulnerability affecting WhatsApp for Android, WhatsApp for iOS, WhatsApp Business and WhatsApp Enterprise clients. A commercial exploit sold by NSO Group was used against more than 1,400 targeted phones worldwide to install Pegasus spyware. Targets included human rights lawyers, investigative journalists, dissidents and government officials.

**Table 1: CVE-2019-3568 incident timeline**

| Date / Period | Event | Technical significance | Response / Outcome |
|---------------|-------|------------------------|--------------------|
| 2016–2018 | Citizen Lab investigations on NSO Group's Pegasus spyware | Identified commercial spyware targeting dissidents and journalists across 45 countries | Global forensic monitoring of NSO command-and-control infrastructure began |
| May 2019 | Active exploitation of CVE-2019-3568 in WhatsApp's VoIP stack | Unauthenticated zero-click SRTCP packet injection causing a heap buffer overflow in `libwhatsapp.so` | WhatsApp patched the vulnerability, updated server infrastructure and deployed client updates |
| May 13, 2019 | WhatsApp public security advisory and user notification | Formal disclosure of CVE-2019-3568; notifications sent to about 1,400 affected users | Global app updates; forced attacker infrastructure to decommission the active exploit vector |
| October 2019 | WhatsApp LLC and Meta file a lawsuit against NSO Group | Civil complaint in U.S. District Court (N.D. Cal.) alleging CFAA (§ 1030) and CDAFA (§ 502) violations | Established precedent of a major platform suing a commercial surveillance vendor on behalf of its users |
| November 2021 | Ninth Circuit ruling on foreign sovereign immunity | Court rejected NSO's claim of derivative foreign sovereign immunity under the FSIA | WhatsApp's CFAA civil lawsuit allowed to proceed toward discovery in U.S. federal court |
| 2023–2025 | U.S. Supreme Court cert denial; district court sanctions and liability rulings | Supreme Court denied cert (January 2023); district court issued evidentiary sanctions and granted summary judgment | NSO Group held liable for CFAA, CDAFA and breach of contract violations |

### 7.2 WhatsApp VoIP Stack Architecture and Real-Time Media Signalling

WhatsApp's VoIP uses two standard streaming protocols. The voice channel is streamed in packets over the **Real-Time Transport Protocol (RTP)**, and the **Secure Real-Time Transport Control Protocol (SRTCP)** controls sessions, estimates jitter and carries signalling. Real-time media stacks have very low latency requirements: they must receive UDP packet streams, start jitter buffers and complete the signalling handshake *before* accepting or rejecting an incoming call. This means untrusted packets are parsed before the user can respond.

### 7.3 Low-Level Binary Analysis: Heap Buffer Overflow in `libwhatsapp.so`

CVE-2019-3568 maps to **CWE-120** (Buffer Copy Without Checking Size of Input) and **CWE-787** (Out-of-bounds Write). The flaw lay in the native C/C++ shared library `libwhatsapp.so`, which parses SRTCP signalling frames.

When the client receives a VoIP call, an SRTCP signalling stream arrives on the same UDP socket. The vulnerable parsing routine read a length field from the packet header and passed it straight to a memory copy operation targeting a fixed-size heap buffer, **with no bounds check**. A malformed packet with an oversized length field made the copy overrun the buffer, corrupting adjacent heap memory (object metadata, vtable pointers and function pointers).

Because the parser ran automatically on socket receipt, the exploit was **zero-click**: the target did not need to answer the call, click a link or open a file. Attacker shellcode then suppressed call ringers, deleted call logs and hid UI notifications to conceal the attack.

### 7.4 Pegasus Delivery, Privilege Escalation and Volatile RAM Exfiltration

After gaining code execution in the WhatsApp process, first-stage shellcode acted as a stager that contacted command-and-control (C2) servers hosting second-stage spyware modules. Pegasus then used local OS privilege escalation (unpatched kernel or iOS WebKit/kernel vulnerabilities) to escape the application sandbox. With kernel- or root-level privileges, it established a long-term footprint on the device.

The malware intercepted WhatsApp messages by reading them directly from process RAM, before encryption or after decryption, by hooking functions in system libraries. It could also exfiltrate contact lists, SMS messages, location data, photos, and real-time microphone and camera data.

### 7.5 Boundary Invalidation Argument: Why Protocol Security Fails on Broken Endpoints

The incident shows that channel security becomes irrelevant when the client's binary execution protection is broken. Under the Signal Protocol, a network eavesdropper sees only ciphertext. The attacker instead hijacked the process at the point of presentation and obtained plaintext from display-layer memory in RAM, without ever traversing the protocol's cryptographic abstractions.

---

## 8. Preventive Measures and Defensive Engineering Hardening

### 8.1 Memory-Safe Systems Programming (Rust Migration)

The technical root cause of CVE-2019-3568 was the use of legacy, memory-unsafe C/C++ parsing libraries to process network data without validation. The best defense is to write network parsers, media decoders and protocol engines in a memory-safe language such as **Rust**, which provides compile-time memory safety, ownership and borrowing guarantees. This eliminates spatial memory errors (buffer overflows, out-of-bounds writes) and temporal errors (use-after-free, double free) without garbage collection.

### 8.2 Input Validation, Bounds Verification and Coverage-Guided Fuzzing

Where legacy C/C++ must remain, input sanitization is mandatory. Every length, offset and count field in an incoming packet header must be checked against tight limits before it reaches allocation or copy routines, for example:

```c
if (srtcp_len > MAX_ALLOWED) return PARSE_ERR;
```

Media parsers should also be continuously fuzzed with tools such as **libFuzzer** or **AFL++** to find edge-case parsing bugs before release.

### 8.3 OS Sandboxing, Least Privilege and System Call Filtering

OS-level process isolation reduces the blast radius of a compromise. High-risk network parsers should run in isolated sandbox processes restricted by system call filters (Linux `seccomp-bpf`, iOS App Sandboxing), with no access to the file system, microphone/camera or database sockets. Persistent identity private keys should be kept in hardware security enclaves (Android StrongBox, Apple Secure Enclave), and session keys locked in memory with `mlock` so they are never paged to disk.

### 8.4 Defensive Engineering Control Matrix

**Table 3: Defensive engineering control matrix for secure messaging platforms**

| System risk vector | Control priority | Recommended defensive mechanism | Verification and testing framework |
|--------------------|------------------|---------------------------------|------------------------------------|
| Unauthenticated packet parsing / zero-click RCE | Critical / Highest | Migrate network parsing engines to Rust/Go; enforce explicit bounds checking | Continuous coverage-guided fuzzing (libFuzzer / AFL++) and AddressSanitizer (ASan) |
| Process memory escalation and sandbox escape | Very High | Restrict media parsers inside isolated sandboxes using seccomp-bpf filters | Sandbox boundary penetration testing and privilege reviews |
| Volatile key scraping from RAM | High | Lock session keys via `mlock`; zeroize key memory immediately after use | Automated volatile RAM heap inspection after decryption |
| Post-quantum traffic harvesting | Medium-High | Deploy PQXDH hybrid handshakes combining Curve25519 with ML-KEM-768 | Automated symbolic protocol verification in ProVerif |

---

## 9. Results, Discussion and Uncertainty Assessment

### 9.1 Critical Analytical Findings

Cross-analysis of protocol specifications, vulnerability disclosures and forensic reports leads to three major conclusions:

1. **E2EE transport integrity remained intact.** WhatsApp's use of the Signal Protocol was not compromised in this attack. Payloads in transit stayed protected from relay eavesdropping by the Double Ratchet and X3DH algorithms.
2. **The execution layer was the point of failure.** Native C/C++ libraries that parse SRTCP without validation were exploited to subvert protocol defenses before the protocol was involved. Memory corruption in the application execution layer invalidated confidentiality.
3. **Encryption does not equal non-repudiation.** Encrypted channels provide entity authentication and cryptographic deniability, but not legal non-repudiation. Even if messages are assumed to come from real humans, endpoint compromise remains a problem.

### 9.2 Telemetry Gaps, Evidence Quality and Exploitation Uncertainties

- Court documents show WhatsApp notified about 1,400 user accounts, but there are **no public records of the percentage of successful infections**.
- The attack chain has several distinct stages (targeting, initial RCE, sandbox escape, persistent spyware installation). Some targeted devices received initial exploits that were defeated by ASLR randomization on the heap overflow and never received the Pegasus payload.
- Public forensic reports cover only the devices that were examined, which introduces **selection bias**.

---

## 10. Legal, Ethical and Accountability Context

### 10.1 WhatsApp LLC v. NSO Group Litigation and CFAA Rulings

In late 2019, WhatsApp LLC and its parent Meta Platforms, Inc. sued NSO Group Technologies Ltd. in the U.S. District Court for the Northern District of California (*WhatsApp LLC v. NSO Group Technologies Ltd.*, No. 4:19-cv-07123). The complaint alleged violations of the **Computer Fraud and Abuse Act (18 U.S.C. § 1030)** and the **California Comprehensive Computer Data Access and Fraud Act (Cal. Penal Code § 502)**: the defendants used fake SRTCP packets sent through WhatsApp servers to run arbitrary code on target phones.

NSO moved to dismiss on grounds of foreign sovereign immunity, arguing that because its customers were governments, the court lacked jurisdiction. The **Ninth Circuit denied the motion in November 2021**, on the basis that private parties who sell surveillance tools to sovereign states do not have universal immunity. In **January 2023**, the U.S. Supreme Court declined to hear the case, allowing WhatsApp's CFAA claims to proceed.

### 10.2 Coordinated Vulnerability Disclosure and Global Policy Impact

The incident significantly influenced vulnerability disclosure policy and commercial surveillance regulation worldwide. It contributed to regulatory responses including the U.S. Department of Commerce adding **NSO Group to the Entity List** for malicious cyber activities. It also showed that software vendors need clearly defined Coordinated Vulnerability Disclosure (CVD) programs, detailed advisories, automatic client updates, and active notification of users who were targeted.

---

## 11. Conclusion and Future Scope

### 11.1 Synthesis of Strategic Lessons

The WhatsApp E2EE and CVE-2019-3568 analysis is a reminder that secure messaging is the product of a holistic system engineering approach. End-to-end encryption is the de-facto standard for digital privacy, protecting message content from relay servers, ISPs and eavesdroppers. But protocol security extends only as far as the binary interface of the client application. Memory-corruption defects in parsers that handle untrusted network input can make state-of-the-art cryptography practically useless against out-of-band RAM exfiltration.

### 11.2 Actionable Engineering Mandates for Secure Messaging Platforms

To defend against zero-click endpoint attacks, engineering teams should follow seven architectural mandates:

1. **Memory-safe languages:** rewrite untrusted network parsing, media decoding and VoIP subroutines in a memory-safe language (Rust).
2. **Strict input sanitization:** explicitly check all network length headers before any memory allocation or copy.
3. **Coverage-guided fuzzing:** integrate libFuzzer / AFL++ in the CI/CD pipeline to catch parsing bugs before deployment.
4. **Isolate high-risk subroutines:** run VoIP media parsing only in sandbox processes with system call filters (seccomp-bpf).
5. **Key protection:** use volatile memory locking and immediate zeroization for in-memory keys, and store long-term keys in hardware enclaves.
6. **Switch to PQXDH:** use ML-KEM-768 hybrid post-quantum handshakes to protect traffic against future quantum attacks.
7. **Emergency patch deployment:** automate and silently deploy client updates to minimize exposure after disclosure.

### 11.3 Future Research Scope

- Validate out-of-band key verification techniques using formally verified tooling (e.g., CompCert and F*)
- Study memory-safe hardware architectures such as ARM CHERI (Capability Hardware Enhanced RISC Instructions) that can protect client devices from zero-day attacks

---

## 12. Repository Structure

```
.
├── README.md                          # Project documentation (this file)
├── Case_Study_Report_Group15.pdf      # Full case study report
└── Similarity_Report.pdf              # Similarity / plagiarism report
```

> Rename the entries above to match the files you actually upload.

---

## 13. How to Use This Repository

1. Open `Case_Study_Report_Group15.pdf` to read the complete report.
2. Use this README as a quick guide to the contents, findings and references.
3. Refer to Section 15 for the sources behind each claim.

No software installation or code execution is required: this is a documentation-only repository.

---

## 14. Tools, Standards and Technologies Referenced

| Category | Items |
|----------|-------|
| Protocols | Signal Protocol, X3DH, Double Ratchet, PQXDH, Noise Protocol Framework, RTP, SRTCP, TLS |
| Cryptography | Curve25519, Ed25519, AES-256-GCM, HKDF-SHA256, HMAC-SHA256, SHA-512, ML-KEM-768 |
| Standards | NIST FIPS 197, 180-4, 186-5, 203, 204, 140-3; NIST SP 800-38D; RFC 5869, 7748, 8032, 8439, 8446 |
| Vulnerability classes | CWE-120, CWE-787, CWE-125 |
| Security tooling | ProVerif, CryptoVerif, libFuzzer, AFL++, AddressSanitizer, seccomp-bpf |
| Platform security | Android StrongBox, Apple Secure Enclave, iOS App Sandbox, ARM CHERI |
| Languages | C/C++ (vulnerable components), Rust (recommended mitigation) |

---

## 15. References

[1] National Institute of Standards and Technology (NIST), "Recommendation for Key-Derivation Methods in Key-Establishment Schemes," *NIST Special Publication 800-56C Rev. 2*, 2020.

[2] NIST, "Advanced Encryption Standard (AES)," *FIPS PUB 197*, updated 2023.

[3] WhatsApp LLC, "WhatsApp End-to-End Encryption Technical Whitepaper," WhatsApp Security Documentation, 2021.

[4] NIST, "Recommendation for Block Cipher Modes of Operation: Galois/Counter Mode (GCM) and GMAC," *NIST Special Publication 800-38D*, 2007.

[5] NIST, "Secure Hash Standard (SHS)," *FIPS PUB 180-4*, 2015.

[6] NIST, "Digital Signature Standard (DSS)," *FIPS PUB 186-5*, 2023.

[7] NIST, "Security Requirements for Cryptographic Modules," *FIPS PUB 140-3*, 2019.

[8] NIST, "Guidelines for the Selection, Configuration, and Use of Transport Layer Security (TLS) Implementations," *NIST Special Publication 800-52 Rev. 2*, 2019.

[9] NIST, "Module-Lattice-Based Key-Encapsulation Mechanism Standard," *FIPS Publication 203*, National Institute of Standards and Technology, Gaithersburg, MD, 2024.

[10] NIST, "Module-Lattice-Based Digital Signature Standard," *FIPS Publication 204*, National Institute of Standards and Technology, Gaithersburg, MD, 2024.

[11] T. Perrin, "The Noise Protocol Framework," *RFC 9420*, 2023.

[12] C. Cremers, M. Horvat, J. Hoyland, and S. Scott, "A Comprehensive Symbolic Analysis of the Signal Protocol," in *Proceedings of the IEEE European Symposium on Security and Privacy (EuroS&P)*, Paris, France, 2017.

[13] M. Bellare and P. Rogaway, "Entity Authentication and Key Distribution," in *Advances in Cryptology – CRYPTO '93*, Springer, 1993.

[14] D. J. Bernstein, "Curve25519: New Diffie-Hellman Speed Records," in *Public Key Cryptography – PKC 2006*, Lecture Notes in Computer Science, vol. 3958, Springer, Berlin, 2006.

[15] H. Krawczyk and P. Eronen, "HMAC-based Extract-and-Expand Key Derivation Function (HKDF)," *RFC 5869*, Internet Engineering Task Force, 2010.

[16] Y. Nir and A. Langley, "ChaCha20 and Poly1305 for IETF Protocols," *RFC 8439*, Internet Engineering Task Force, 2018.

[17] E. Rescorla, "The Transport Layer Security (TLS) Protocol Version 1.3," *RFC 8446*, Internet Engineering Task Force, 2018.

[18] NIST, "Guide to Computer Security Log Management," *NIST Special Publication 800-92*, 2006.

[19] M. Marlinspike and T. Perrin, "The Double Ratchet Algorithm," Signal Specification, Revision 1, 2016.

[20] M. Marlinspike and T. Perrin, "The X3DH Key Agreement Protocol," Signal Specification, Revision 1, 2016.

[21] Signal Technology Foundation, "Signal Protocol Specifications and Cryptographic Documentation," Signal.org, 2023.

[22] K. Bhargavan, C. Jacomme, F. Kiefer, and R. Schmidt, "Formal Verification of the PQXDH Post-Quantum Key Agreement for End-to-End Messaging," in *Proceedings of the USENIX Security Symposium*, 2024.

[23] National Vulnerability Database, "CVE-2019-3568 Detail," National Institute of Standards and Technology (NIST), May 2019.

[24] WhatsApp LLC, "Security Advisory: CVE-2019-3568 - Buffer Overflow in VoIP Stack," Facebook Security Bulletins, May 13, 2019.

[25] Check Point Research, "The NSO WhatsApp Vulnerability—This Is How It Happened," Technical Analysis Report, May 14, 2019.

[26] The Citizen Lab, "The Million Dollar Dissident: NSO Group's Pegasus Spyware," Munk School of Global Affairs and Public Policy, University of Toronto, 2016–2019.

[27] The Citizen Lab, "Hide and Seek: NSO Group's Pegasus Spyware Actively Operating in 45 Countries," Research Report No. 113, 2018.

[28] U.S. District Court for the Northern District of California, *WhatsApp LLC et al. v. NSO Group Technologies Ltd. et al.*, Case No. 4:19-cv-07123-PJH, Civil Filings and Orders, 2019–2024.

[29] U.S. Court of Appeals for the Ninth Circuit, *WhatsApp LLC v. NSO Group Technologies Ltd.*, 17 F.4th 930 (9th Cir. 2021), cert. denied, 143 S. Ct. 562 (2023).

[30] MITRE Corporation, "CWE-120: Buffer Copy without Checking Size of Input (Classic Buffer Overflow)," Common Weakness Enumeration, 2019.

[31] MITRE Corporation, "CWE-787: Out-of-bounds Write," Common Weakness Enumeration, 2019.

[32] MITRE Corporation, "CWE-125: Out-of-bounds Read," Common Weakness Enumeration, 2019.

[33] OWASP Foundation, "Input Validation Cheat Sheet," OWASP Cheat Sheet Series, 2023.

[34] OWASP Foundation, "Mobile Application Security Verification Standard (MASVS)," Version 2.0, 2023.

[35] Google Developers, "Android Security Internal Architecture and Application Sandbox," Android Open Source Project, 2023.

[36] Apple Inc., "Apple Platform Security Architecture Documentation," Apple Security Publications, 2023.

[37] Cybersecurity and Infrastructure Security Agency (CISA), "Shifting the Balance of Cybersecurity Risk: Principles and Approaches for Secure by Design Software," CISA/NSA/FBI Joint Guidance, 2023.

[38] CISA, "Known Exploited Vulnerabilities Catalog (KEV)," Cybersecurity and Infrastructure Security Agency, 2023.

[39] International Organization for Standardization, "Information Security Management Systems - Requirements," *ISO/IEC 27001:2022*, 2022.

[40] ISO/IEC, "Information Security Controls," *ISO/IEC 27002:2022*, 2022.

---

## 16. How to Cite

```bibtex
@techreport{group15_whatsapp_nso_2026,
  title       = {Case Study Centred on the 2019 WhatsApp--NSO Group/Pegasus Incident},
  author      = {Kulkarni, Prafful and Mohapatra, Saanvi and Jena, Sai Sankar and Subham},
  institution = {Symbiosis Institute of Technology, Pune},
  year        = {2026},
  note        = {Cyber Security case study report, Group 15, B.Tech Batch 2023--27, Semester VII}
}
```

---

## 17. Acknowledgements

We thank **Dr. Mangal Singh** for guidance on this case study, and the Department of Electronics & Telecommunication Engineering at Symbiosis Institute of Technology, Pune. We also acknowledge the public work of Citizen Lab, Check Point Research, the Signal Foundation, NIST and MITRE, on which this study depends.

---

## 18. License and Disclaimer

This repository contains academic coursework and is shared for **educational purposes only**. All third-party material (specifications, reports, court documents) remains the property of its respective authors and publishers, and is cited in Section 15.

The analysis relies solely on public sources. **No exploit code was developed, and no spyware samples were analysed.** Nothing in this repository should be used to attack, surveil or compromise any system or person.

---

## 19. Contact

| Name | PRN |
|------|-----|
| Prafful Kulkarni | 23070123099 |
| Saanvi Mohapatra | 23070123110 |
| Sai Sankar Jena | 23070123112 |
| Subham | 23070123132 |

Department of Electronics & Telecommunication Engineering, Symbiosis Institute of Technology, Pune

*Add your GitHub usernames or college emails here if you want to be reachable.*
