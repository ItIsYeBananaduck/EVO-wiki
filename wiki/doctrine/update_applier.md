---
title: update_applier
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/update_applier.md"]
updated: 2026-07-24
---

# generate keys
node -e "const fs=require('fs');const {generateKeyPairSync}=require('crypto');const {privateKey, publicKey}=generateKeyPairSync('ed25519',{privateKeyEncoding:{type:'pkcs8',format:'pem'},publicKeyEncoding:{type:'spki',format:'pem'}});fs.mkdirSync('keys',{recursive:true});fs.writeFileSync('keys/ed_priv.pem', privateKey);fs.writeFileSync('keys/ed_pub.pem', publicKey);console.log('WROTE keys/ed_priv.pem and keys/ed_pub.pem');"

# run tests with signing enabled but no DP noise
$env:PATCH_SIGNING_PRIVATE_KEY = Get-Content -Raw .\keys\ed_priv.pem
$env:PATCH_SIGNING_PUBLIC_KEY = Get-Content -Raw .\keys\ed_pub.pem
$env:CLIP_NORM = '1e12'
$env:DP_SIGMA = '0'
$env:MIN_CONTRIBUTORS = '1'
npx vitest run api/tests/integration/test-update-fetch-apply.test.js
```

Privacy guidance & production tips

- Embed the server public key in the shipped app (or fetch it from a pinned endpoint) so devices can verify signatures without contacting a third party.
- Use `CLIP_NORM` + `DP_SIGMA` to get meaningful privacy guarantees. If you enable DP, update device-side acceptance tests to allow numeric tolerance.
- Ensure `staging_data` is stored on encrypted storage and that logs do not leak modelId or other identifying fields.
- Consider rotating signing keys periodically and publishing `meta.keyId` so clients can detect key rotation.

Further improvements

- Add a public-key endpoint `/operator/keys/current` that returns the current public key PEM and `keyId` so devices can fetch and cache the correct verification key.
- Add a key-rotation mechanism and include `keyId` in patch meta to allow clients to select the right public key.
- For strongest privacy, explore secure aggregation protocols (more complex client changes required).

Questions or next steps

- I can add a small script `scripts/generate-keys.js` and a test wrapper to run the integration suite with signing enabled.
- I can also add a minimal `/operator/keys` endpoint and a small client helper that fetches and caches the current keyId/public key.

Pick what you want next and I will implement it.

## Related

^[{src_rel}]
