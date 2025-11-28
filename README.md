# PS256Signature2020 Suite

A Linked Data Signature suite for creating and verifying Data Integrity Proofs using RSASSA-PSS with SHA-256 (PS256 algorithm).

## Installation

Install from NPM:
```
npm i @eecc/ps256-signature-2020
```

## Usage

### Signing a Verifiable Credential

```javascript
import jsigs from 'jsonld-signatures';
import * as jose from 'jose';
import { PS256Signature2020 } from 'ps256-signature-2020';

const { purposes: { AssertionProofPurpose } } = jsigs;

// Create an unsigned credential
const unsignedCredential = {
  '@context': [
    'https://www.w3.org/2018/credentials/v1'
  ],
  id: 'https://example.edu/credentials/3732',
  type: ['VerifiableCredential'],
  issuer: 'https://example.edu/issuers/565049',
  issuanceDate: '2020-03-10T04:24:12.164Z',
  credentialSubject: {
    id: 'did:example:ebfeb1f712ebc6f1c276e12ec21'
  }
};

// Generate a PS256 key pair
const generatedKeyPair = await jose.generateKeyPair('PS256', { extractable: true });
const privateKeyJwk = await jose.exportJWK(generatedKeyPair.privateKey);
const publicKeyJwk = await jose.exportJWK(generatedKeyPair.publicKey);

// Create a key object with DID-based identifier
const key = {
  id: 'did:web:example.com#keys-1',
  type: 'JsonWebKey2020',
  controller: 'did:web:example.com',
  privateKey: privateKeyJwk
};

// Create the suite with the key
const suite = new PS256Signature2020({ key });

// Sign the credential
const signedCredential = await jsigs.sign(unsignedCredential, {
  suite,
  purpose: new AssertionProofPurpose(),
  documentLoader
});

console.log(JSON.stringify(signedCredential, null, 2));
```

**Result:**

```json
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1"
  ],
  "id": "https://example.edu/credentials/3732",
  "type": ["VerifiableCredential"],
  "issuer": "https://example.edu/issuers/565049",
  "issuanceDate": "2020-03-10T04:24:12.164Z",
  "credentialSubject": {
    "id": "did:example:ebfeb1f712ebc6f1c276e12ec21"
  },
  "proof": {
    "type": "JsonWebSignature2020",
    "created": "2024-11-06T10:23:45Z",
    "verificationMethod": "did:web:example.com#keys-1",
    "proofPurpose": "assertionMethod",
    "proofValue": "z3QJPuVjHMfKMVvnvP3YdKNZvxVj5xGc..."
  }
}
```

### Verifying a Signed Credential

```javascript
import jsigs from 'jsonld-signatures';
import { PS256Signature2020 } from 'ps256-signature-2020';

const { purposes: { AssertionProofPurpose } } = jsigs;

// The signed credential from above
const signedCredential = { /* ... */ };

// Create a verification suite
const suite = new PS256Signature2020();

// Verify the credential
const result = await jsigs.verify(signedCredential, {
  suite,
  purpose: new AssertionProofPurpose(),
  documentLoader
});

console.log('Verified:', result.verified); // true
```

## Compatibility

This library uses RSASSA-PSS signature algorithm with SHA-256 hash function. It is compatible with standard RSA key pairs used for PS256 signing.

## Licence
[BSD 3-Clause License](LICENSE) © 2025 European EPC Competence Center GmbH
