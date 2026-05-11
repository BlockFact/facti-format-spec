# BlockFact Media File Specification (.facti / .facta / .factv)

**Version:** 2.0  
**Last Updated:** May 11, 2026

---

## Overview

The BlockFact family of media formats (`.facti`, `.facta`, `.factv`) are binary container formats designed for provenance-verified media. Each file wraps a standard media payload (JPEG, PNG, WAV, MP4, etc.) with cryptographic provenance data including:

- JSON metadata (GPS, timestamp, content hash, blockchain registration)
- C2PA Content Credentials (JUMBF manifest)
- Zero-knowledge proof data (reserved)
- Steganographic watermark parameters (reserved)

The format is backward-compatible: v1 readers process the header + metadata + media payload and ignore extension blocks. v2 readers additionally parse typed extension blocks.

### IANA Registrations

| Format | MIME Type | Registration |
|--------|-----------|--------------|
| `.facti` | `image/vnd.blockfact.facti` | https://www.iana.org/assignments/media-types/image/vnd.blockfact.facti |
| `.facta` | `audio/vnd.blockfact.facta` | https://www.iana.org/assignments/media-types/audio/vnd.blockfact.facta |
| `.factv` | `video/vnd.blockfact.factv` | https://www.iana.org/assignments/media-types/video/vnd.blockfact.factv |

---

## Binary Format

### Magic Bytes

| Format | Magic Bytes (hex) |
|--------|-------------------|
| `.facti` | `FA 49 41 00` |
| `.facta` | `FA 41 41 00` |
| `.factv` | `FA 56 41 00` |

### File Layout

```
┌──────────────────────────────────────────────────────────────┐
│ [4 bytes]  Magic bytes                                        │
│ [4 bytes]  Metadata length N (big-endian uint32)              │
│ [N bytes]  JSON metadata (UTF-8)                              │
│ [M bytes]  Media payload (JPEG, PNG, WAV, MP4, etc.)          │
│ ─── v1 readers stop here ───                                  │
│ [2 bytes]  Extension block count (big-endian uint16)          │
│ For each extension block:                                     │
│   [4 bytes]  Block type (4 ASCII bytes)                       │
│   [4 bytes]  Block data length L (big-endian uint32)          │
│   [L bytes]  Block data                                       │
└──────────────────────────────────────────────────────────────┘
```

### Byte-Level Detail

```
Offset    Size      Field
────────  ────────  ──────────────────────────────────────
0x00      4         Magic bytes
0x04      4         Metadata length N (big-endian uint32)
0x08      N         JSON metadata (UTF-8 encoded)
0x08+N    M         Media payload
0x08+N+M  2         Extension block count (big-endian uint16)
                    Repeated for each block:
  +0      4           Block type identifier (ASCII)
  +4      4           Block data length L (big-endian uint32)
  +8      L           Block data
```

---

## JSON Metadata

The metadata is a UTF-8 encoded JSON object. Required fields for v2 files:

| Field | Type | Description |
|-------|------|-------------|
| `format_version` | integer | Format version. MUST be `2` for files with extension blocks. |
| `image_length` or `media_length` | integer | Byte length of the media payload. Required to locate extension blocks. |

Optional fields:

| Field | Type | Description |
|-------|------|-------------|
| `poseidon_hash` | string | Poseidon hash of the media content (BN254 field) |
| `tx_hash` | string | Blockchain transaction hash |
| `block_number` | integer | Block number of the registration transaction |
| `wallet` | string | Creator's wallet address |
| `latitude` | number | GPS latitude at capture time |
| `longitude` | number | GPS longitude at capture time |
| `timestamp` | string | ISO 8601 capture timestamp |
| `session_id` | string | Unique session identifier |
| `starknet_network` | string | Blockchain network (`mainnet` or `sepolia`) |
| `version` | string | Application version that created the file |
| `verification_method` | string | Verification method used (e.g., `poseidon_lsb_watermark`) |
| `mime` | string | MIME type of the embedded media payload |

### Example

```json
{
  "format_version": 2,
  "image_length": 3145728,
  "poseidon_hash": "16609302302658229514986934480202291780178107023053511680319977605648105652992",
  "tx_hash": "0x1abe0fcea5367d0d9bdbb6c1eda0c2ddb7e9fd750b480c59d8fd035e0b066b1",
  "wallet": "0x1180bb3342105759ac4788afa2aafa33392aecac513dfb31a9bf455983c664d",
  "latitude": 18.4588,
  "longitude": -77.9435,
  "timestamp": "2026-05-09T15:00:00Z",
  "session_id": "abc123",
  "starknet_network": "mainnet",
  "version": "1.6.0",
  "verification_method": "poseidon_lsb_watermark",
  "mime": "image/png"
}
```

---

## Extension Blocks

Extension blocks provide a typed, forward-compatible mechanism for embedding additional data after the media payload.

### Defined Block Types

| Type (ASCII) | Hex | Description |
|--------------|-----|-------------|
| `C2PA` | `43 32 50 41` | C2PA JUMBF manifest store |
| `ZKPF` | `5A 4B 50 46` | Zero-knowledge proof data (reserved) |
| `WMRK` | `57 4D 52 4B` | Watermark parameters (reserved) |

Readers MUST ignore block types they do not recognize.

### C2PA Block

The `C2PA` extension block contains a complete JUMBF manifest store as defined by the C2PA specification. No additional framing or transformation is applied — the block data is byte-for-byte identical to what would be embedded in a JPEG or PNG file.

---

## Reading a .facti File

```
1. Read 4 bytes → verify magic
2. Read 4 bytes → metadata length N (big-endian)
3. Read N bytes → parse JSON metadata
4. Read M bytes → media payload (M = metadata.image_length or metadata.media_length)
5. If format_version >= 2:
   a. Read 2 bytes → block count (big-endian)
   b. For each block:
      - Read 4 bytes → block type
      - Read 4 bytes → block data length L (big-endian)
      - Read L bytes → block data
```

### v1 Compatibility

v1 readers that do not check `format_version` will read the metadata and then treat all remaining bytes as the media payload. Since JPEG and PNG decoders stop at their respective end markers (FFD9 for JPEG, IEND for PNG), the trailing extension block bytes are ignored by image decoders.

---

## Writing a .facti File

```
1. Serialize JSON metadata (include format_version: 2 and image_length/media_length)
2. Write magic bytes (4 bytes)
3. Write metadata length as big-endian uint32 (4 bytes)
4. Write JSON metadata
5. Write media payload
6. Write extension block count as big-endian uint16 (2 bytes)
7. For each block:
   a. Write block type (4 ASCII bytes)
   b. Write block data length as big-endian uint32 (4 bytes)
   c. Write block data
```

---

## Security Considerations

- The JSON metadata is not encrypted in v2. Sensitive data should be stored on-chain rather than in the file.
- The C2PA manifest provides cryptographic binding between the file content and its provenance assertions.
- The Poseidon hash enables zero-knowledge verification of content integrity without revealing the content.
- The steganographic watermark survives format conversion and screenshots.

---

## Contact

- **Website**: https://blockfact.io
- **Email**: egbert@blockfact.io
- **GitHub**: https://github.com/BlockFact

---

## Changelog

### v2.0 (May 11, 2026)
- Complete rewrite to reflect production format
- Added extension block system (C2PA, ZKPF, WMRK)
- Updated magic bytes (`0x41` replaces `0x01` in third byte)
- Removed encryption/reserved fields from header (simplified to metadata length only)
- Added .facta and .factv format variants
- Documented C2PA JUMBF embedding
- Added v1 backward compatibility notes

### v1.4 (January 23, 2025)
- Initial public specification

---

**Copyright © 2025-2026 BlockFact Technologies Inc. All rights reserved.**
