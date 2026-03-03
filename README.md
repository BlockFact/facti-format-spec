# .facti Format Specification

Official specification for the BlockFact `.facti` file format - a cryptographically verifiable image container format.

## Overview

The `.facti` file format wraps standard image formats (JPEG, PNG, etc.) with blockchain-verified metadata to ensure authenticity and prevent tampering.

**Key Features:**
- 🔗 Blockchain verifiability via on-chain references
- 🔐 Encrypted metadata for privacy
- ✅ Checksum validation against tampering
- 📦 Lite & Full system support

## Quick Start

```bash
# View the specification
cat SPECIFICATION.md

# Or read online
https://github.com/BlockFact/facti-format-spec
```

## File Structure

```
┌─────────────────────────────────────┐
│ Magic Number (4 bytes)              │  0xFA 0x49 0x01 0x00
├─────────────────────────────────────┤
│ Version (4 bytes)                   │  Format version
├─────────────────────────────────────┤
│ Metadata Length (4 bytes)           │  Size of metadata block
├─────────────────────────────────────┤
│ Metadata Block (N bytes)            │  JSON metadata (encrypted)
├─────────────────────────────────────┤
│ Image Data (remaining bytes)        │  Original image (JPEG/PNG/etc)
└─────────────────────────────────────┘
```

## Use Cases

- **Insurance Claims**: Verify photo authenticity for claims processing
- **Real Estate**: Prove property photos are unedited and recent
- **Journalism**: Authenticate news photos and prevent deepfakes
- **Legal Evidence**: Create tamper-proof visual evidence
- **Digital Assets**: Verify NFT and digital art provenance

## SDKs

Create and verify `.facti` files using our official SDKs:

- **React Native Pro**: `npm install @blockfact/react-native-facti-pro`
- **iOS**: `pod 'BlockFactSDK'`
- **Android**: `implementation 'io.blockfact:sdk:2.1.0'`
- **React Web**: `npm install @blockfact/react-facti`

[View all SDKs →](https://blockfact.io/developers)

## Specification

See [SPECIFICATION.md](./SPECIFICATION.md) for the complete technical specification.

## Version History

- **v1.4** (Current) - Finalized format, clarified encryption
- **v1.3** (Jan 2025) - Separated IANA parameters from internal fields
- **v1.2** - Added encryption guidelines
- **v1.1** - Introduced lite vs. full parsing
- **v1.0** (Jan 16, 2025) - Initial release

## Resources

- **Website**: [blockfact.io](https://blockfact.io)
- **Demo**: [blockfact.io/demo](https://blockfact.io/demo)
- **Documentation**: [GitHub](https://github.com/BlockFact)
- **Support**: support@blockfact.io

## License

Copyright © 2025 BlockFact. All rights reserved.
