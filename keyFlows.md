# 🔐 End-to-End Encryption & Backup Security
*WhatsApp/Signal Security Model Technical Documentation*

---

## 🏗 System Architecture Overview

### Core Security Protocols
| Protocol | Purpose | Security Properties |
|----------|---------|-------------------|
| **X3DH** | Initial Session Setup | Authenticated Key Exchange |
| **Double Ratchet** | Ongoing Messaging | Forward Secrecy + Break-in Recovery |
| **AES-256-CBC** | Message Encryption | Confidentiality |
| **HMAC-SHA256** | Message Authentication | Integrity & Authenticity |
| **Curve25519** | Key Pairs | Elliptic Curve Cryptography |

---

## 🔑 Cryptographic Key Inventory

### Long-Term Identity Keys
| Key Type | Lifetime | Storage | Purpose |
|----------|----------|---------|---------|
| **Identity Key Pair** | Permanent | Device Secure Storage | User Identity |
| **Signed Pre-Key** | Weeks/Months | Server + Device | Session Bootstrap |
| **One-Time Pre-Keys** | Single Use | Server Pool | First-Time Sessions |

### Session Keys (Ephemeral)
| Key Type | Lifetime | Derivation | Purpose |
|----------|----------|------------|---------|
| **Ephemeral Key Pair** | Per Session | Random Generate | DH Ratchet Steps |
| **Root Key** | Session Duration | X3DH Output | Chain Key Derivation |
| **Chain Keys** | Message Sequence | Root Key → KDF-R | Message Key Generation |
| **Message Keys** | Single Message | Chain Key → KDF-M | Per-Message Encryption |

### Backup Security Keys
| Key Type | Source | Storage | Purpose |
|----------|--------|---------|---------|
| **Backup Encryption Key** | Password KDF or Random | User Memory | Cloud Backup Protection |
| **64-digit Recovery Key** | Random 256-bit | User Secure Storage | Backup Recovery |

---

## ⚙️ Key Generation & Management Timeline

### Phase 1: Application Installation
```plaintext
1. Generate Identity Key Pair (Curve25519)
   ├── Private Key: Device Secure Storage
   └── Public Key: Upload to Server

2. Generate Signed Pre-Key
   ├── Signed with Identity Key
   └── Upload public part to server

3. Generate One-Time Pre-Key Batch
   ├── 100+ keys generated
   └── Public keys uploaded to server pool
```

### Phase 2: Session Establishment (X3DH)
```plaintext
Sender Actions:
1. Fetch recipient's bundle from server:
   │   - Identity Public Key
   │   - Signed Pre-Key  
   │   - One-Time Pre-Key (if available)
   │
2. Generate ephemeral key pair
   │
3. Perform X3DH key agreement:
   │   DH1 = DH(IKA, SPKb)
   │   DH2 = DH(EKA, IKB)  
   │   DH3 = DH(EKA, SPKb)
   │   DH4 = DH(EKA, OPKb)
   │
4. Compute master secret:
   └── K = KDF(DH1 || DH2 || DH3 || DH4)

5. Derive initial keys:
   ├── Root Key = KDF(K, "Root")
   └── Chain Keys = KDF(Root Key, "Chain")
```

### Phase 3: Ongoing Messaging (Double Ratchet)
```plaintext
For each sent message:
1. Derive Message Key:
   │   Message Key = KDF(Chain Key, "Message")
   │
2. Encrypt & Authenticate:
   │   Ciphertext = AES-256(Message Key, Plaintext)
   │   Auth Tag = HMAC-SHA256(Message Key, Ciphertext)
   │
3. Ratchet Chain:
   └── New Chain Key = KDF(Chain Key, "Ratchet")

Periodic DH Ratchet:
1. Generate new Ephemeral Key Pair
2. Perform DH exchange:
   │   New Root Key = KDF(DH(Root Key, EK_peer))
   └── Reset sending/receiving chains
```

### Phase 4: Backup Encryption Setup
```plaintext
Option A: Password-Based
1. User sets backup password
2. Derive Backup Key:
   │   Backup Key = PBKDF2(password, salt, 100,000)
   └── Salt stored with backup metadata

Option B: Recovery Key  
1. System generates random 256-bit key
2. Encode as 64-digit hexadecimal
3. Display to user for secure storage
   └── "1234-5678-9ABC-DEF0-..." format

Backup Process:
1. Encrypt chat database:
   │   Backup = AES-256(Backup Key, Plaintext Backup)
   └── Upload encrypted backup to cloud
```

### Phase 5: Recovery & Restoration
```plaintext
Recovery Flow:
1. User reinstalls application
2. Verify phone number via SMS OTP
3. Detect existing backup in cloud
4. Prompt for recovery credential:
   │   ├── Backup password, OR
   │   └── 64-digit recovery key
5. Derive/validate Backup Key
6. Download and decrypt backup
7. Restore chat history
```

---

## 🛡 Security Properties Matrix

| Property | Mechanism | Implementation |
|----------|-----------|----------------|
| **Forward Secrecy** | Per-message keys + deletion | Message keys destroyed after use |
| **Break-in Recovery** | DH Ratchet steps | New ephemeral keys refresh chains |
| **End-to-End Encryption** | Key agreement excludes server | Only communicating parties derive keys |
| **Authentication** | Identity key signatures | Verified signed pre-keys |
| **Backup Security** | User-only knowledge | Password/KDF or recovery key required |

---

## 🔄 Key Lifecycle Management

### Rotation Schedule
```plaintext
Identity Keys:        Never (until reinstall)
Signed Pre-Keys:      2-4 week rotation
One-Time Pre-Keys:    Consumed on use, replenished
Ephemeral Keys:       Per DH ratchet (hours/days)
Chain Keys:           Every message (ratchet advance)
Message Keys:         Single use, immediate destruction
Backup Key:           User-controlled change
```

### Destruction Policy
```plaintext
Immediate Destruction:
✓ Message keys after encryption/decryption
✓ Ephemeral private keys after DH computation
✓ One-time pre-keys after consumption

Conditional Destruction:
✓ Chain keys after ratchet step
✓ Root key on session termination
✓ All keys on app uninstall
```

---

## 🎯 Threat Model & Protections

### Protected Against:
- **Message Interception** → E2E Encryption
- **Historical Compromise** → Forward Secrecy  
- **Future Compromise** → Break-in Recovery
- **Server Compromise** → Zero-knowledge backups
- **Identity Spoofing** → Key authentication

### User Responsibilities:
- **Secure storage** of recovery codes
- **Memorization** of backup passwords
- **Verification** of safety numbers (contact identity)

---

## 📋 Summary

This security model provides comprehensive protection through:

1. **Layered Cryptography** - Multiple key types for different purposes
2. **Progressive Security** - Initial setup to ongoing messaging protection
3. **User-Controlled Backup Access** - Cloud storage with zero-knowledge encryption
4. **Automatic Key Management** - Transparent to users while maintaining security
5. **Recovery Options** - Balanced security and usability for backup access

The system ensures that even with full server compromise, message content remains protected and backup data remains inaccessible without explicit user credentials.
