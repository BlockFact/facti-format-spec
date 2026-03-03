# BlockFact Image File Specification (.facti)

**Version:** 1.4  
**Last Updated:** January 23, 2025

---

## Overview

The `.facti` file format is a proprietary image container developed by BlockFact. It wraps standard image formats (e.g., JPEG, PNG) and includes encrypted metadata for verification and integrity.

The format ensures secure and verifiable image distribution, leveraging blockchain technology to authenticate file integrity.

### Key Benefits

- **Blockchain Verifiability**: Ties an image to a unique on-chain reference (blockchain-id)
- **Encrypted Metadata**: Prevents unauthorized access to sensitive data
- **Checksum**: Guards against image manipulation or corruption
- **Lite & Full System Support**: Systems that only need to display the image can bypass metadata decryption (lite mode), while full systems decrypt and validate the metadata for authenticity verification

---

## File Structure

A `.facti` file consists of the following components:

### 1. Magic Number
Identifies `.facti` files uniquely.

**Value:** `0xFA 0x49 0x01 0x00`

### 2. Required Parameters

- **version** (string, required): Specifies the `.facti` format version (e.g., `1.0`)

### 3. Optional Parameters

- **encryption** (string, optional): Specifies the encryption type for metadata (e.g., `AES256`)
- **compression** (boolean, optional): Indicates whether the image content is compressed (`true`/`false`)

### 4. Encrypted Metadata Block

Contains required fields (`content-type`, `blockchain-id`, `checksum`, `encryption-key`) plus any additional fields.

If encrypted, only clients with the appropriate decryption key can parse it. Metadata is typically stored on-chain, but an optional local copy may exist within the `.facti` file for offline verification.

### 5. Image Content

The raw binary of the wrapped image format (JPEG, PNG, etc.). Systems that only need the image can skip or ignore the metadata after reading its size.

---

## Metadata Fields

The `.facti` format may include an optional embedded metadata section that contains structured information. These fields are not media type parameters but are part of the `.facti` file structure itself.

### Optional Internal Fields

- **author** (string, optional): The name or identifier of the image creator
- **creation_date** (ISO 8601 string, optional): The timestamp when the image was created
- **location** (object, optional): Geotagging data (latitude/longitude) if enabled
- **tags** (array, optional): List of keywords or labels related to the image
- **description** (string, optional): A textual description of the image

**Note:** While metadata can be stored inside the `.facti` file, the blockchain-stored metadata is the authoritative source for verification. Systems should treat the local metadata copy as a convenience feature.

---

## Encoding & Encryption

The `.facti` file is binary-encoded, meaning readers must handle raw byte data to parse it correctly.

### Encrypted Metadata

The entire metadata block (N bytes) may be encrypted so unauthorized parties cannot read the fields.

A parser that supports decryption will:

1. Read the metadata block from the file
2. Retrieve the correct key using `encryption-key`
3. Decrypt to obtain the plaintext JSON/object
4. Parse the fields (`blockchain-id`, `checksum`, etc.)

### Symmetric Encryption (AES)

AES (e.g., AES-256-GCM) is commonly used:

1. Generate or retrieve an AES key
2. Encrypt the metadata JSON using that key
3. Store the ciphertext (and IV, if needed) in the `.facti` file
4. The client uses the same key (obtained from a secure KMS) to decrypt

### Asymmetric Encryption (RSA/ECC)

Alternatively, encrypt with a public key so only the holder of the private key can decrypt:

- The `.facti` file stores the ciphertext and a reference to the public/private key pair

---

## Security & Verification

- Metadata is encrypted for privacy and integrity
- The blockchain-based system verifies file authenticity
- The checksum ensures the wrapped image content has not been tampered with

---

## Interoperability

### Lite Systems
Can read and extract the wrapped image without requiring decryption of metadata.

### Full Systems
Can decode both the image and metadata to verify file authenticity and retrieve associated metadata.

### Blockchain Access
Some verification features require a blockchain connection; offline-only systems may not fully validate `.facti` files.

---

## Intended Use Cases

The `.facti` format is ideal for applications requiring secure image distribution with verification capabilities, including:

- Digital asset management
- Blockchain-integrated content platforms
- Secure document and media exchanges
- Insurance claim verification
- Real estate photo authentication
- Journalism and news verification
- Legal evidence documentation

---

## Binary Format Details

```
Offset | Size (bytes) | Field
-------|--------------|------------------
0x00   | 4            | Magic Number (0xFA 0x49 0x01 0x00)
0x04   | 4            | Version (uint32, big-endian)
0x08   | 32           | Reserved (32 bytes, zeros)
0x28   | 4            | Metadata Length (uint32, big-endian)
0x2C   | N            | Metadata Block (JSON, possibly encrypted)
0x2C+N | Remaining    | Image Data (JPEG/PNG/etc.)
```

### Parsing Example

```javascript
// Read magic number
const magic = buffer.slice(0, 4)
if (magic[0] !== 0xFA || magic[1] !== 0x49 || 
    magic[2] !== 0x01 || magic[3] !== 0x00) {
  throw new Error('Invalid .facti file')
}

// Read version
const view = new DataView(buffer)
const version = view.getUint32(4, false) // big-endian

// Read metadata length
const metadataLength = view.getUint32(36, false)

// Extract metadata
const metadataBytes = buffer.slice(40, 40 + metadataLength)
const metadata = JSON.parse(new TextDecoder().decode(metadataBytes))

// Extract image
const imageData = buffer.slice(40 + metadataLength)
```

---

## Contact

For more information or integration support, contact:

- **Website**: [blockfact.io](https://blockfact.io)
- **Email**: support@blockfact.io
- **GitHub**: [github.com/BlockFact](https://github.com/BlockFact)

---

## Changelog

### v1.4 (Current Revision - January 23, 2025)
- Updated version number to 1.4 to reflect finalized changes
- Confirmed removal of metadata from media type parameters
- Clarified IANA parameters vs. internal format fields
- Adjusted encryption and interoperability sections for clarity

### v1.3 (January 23, 2025)
- Removed metadata from media type parameters (moved to internal file structure)
- Clarified distinction between IANA parameters and internal format fields
- Updated encryption guidelines for metadata security
- Improved interoperability section for Lite vs. Full systems

### v1.2
- Added detailed encryption guidelines for metadata (symmetric/asymmetric)
- Clarified storing IV or encryption parameters
- Emphasized privacy and expanded references to key management systems

### v1.1
- Documented usage examples
- Introduced lite vs. full parsing approaches

### v1.0 (Initial Release - January 16, 2025)
- Introduced 4-byte magic number `0xFA 0x49 0x01 0x00`
- Defined core metadata fields (`content-type`, `version`, `blockchain-id`, `checksum`, `encryption-key`)

---

**Copyright © 2025 BlockFact. All rights reserved.**
