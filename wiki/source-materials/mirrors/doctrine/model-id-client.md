---
title: model-id-client
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/model-id-client.md"]
updated: 2026-07-24
---

# Model ID client: download Bloom, test candidate, register

This snippet shows how a device can download the signed Bloom filter, verify signature
with the server public key, test a candidate model id locally, and post the hashed
model id to `/staging/model-ids/register` when unique.

Notes:

- The server returns a gzipped Bloom bitset base64-encoded in the `bloom` field and
  optional `signature_ed25519`, `keyId`, and `publicKey` in the meta.
- The client should verify the signature before trusting the bloom.

Example (Node.js):

```js
import crypto from "crypto";
import zlib from "zlib";
import https from "https";

async function getJson(url) {
  return new Promise((resolve, reject) => {
    https
      .get(url, (res) => {
        let b = "";
        res.on("data", (c) => (b += c.toString()));
        res.on("end", () => resolve(JSON.parse(b)));
      })
      .on("error", reject);
  });
}

// simple bloom test matching server's implementation
function testBloom(bloomBuf, hashHex, m = 16384, k = 4) {
  const mBits = m;
  for (let i = 0; i < k; i++) {
    const seed = crypto
      .createHash("sha256")
      .update(hashHex + "|" + i)
      .digest();
    const idx = seed.readUInt32BE(0) % mBits;
    const byte = Math.floor(idx / 8);
    const bit = idx % 8;
    if ((bloomBuf[byte] & (1 << bit)) === 0) return false;
  }
  return true;
}

async function run() {
  const meta = await getJson("http://127.0.0.1:3000/staging/model-ids/filter");
  // meta.bloom is base64 gz
  const gz = Buffer.from(meta.bloom, "base64");
  const bloomBuf = zlib.gunzipSync(gz);

  // verify signature if provided
  if (meta.signature_ed25519 && meta.publicKey) {
    const sig = Buffer.from(meta.signature_ed25519, "base64");
    const ok = crypto.verify(null, gz, { key: meta.publicKey }, sig);
    if (!ok) throw new Error("invalid signature on bloom");
  }
  // If the server provides `keyId`, you can (and should) verify it matches the
  // SHA256 of the returned public key PEM to guard against key substitution:
  // const keyId = crypto.createHash('sha256').update(meta.publicKey).digest('hex');
  // if (meta.keyId && meta.keyId !== keyId) throw new Error('keyId mismatch');

  // pick candidate model id and test
  const modelId = crypto.randomBytes(32).toString("hex");
  const h = crypto.createHash("sha256").update(modelId).digest("hex");
  const present = testBloom(bloomBuf, h, 16384, 4);
  if (present) {
    console.log("candidate appears present; pick another");
    return;
  }

  // register by POSTing modelHash to server (example uses fetch)
  await fetch("http://127.0.0.1:3000/staging/model-ids/register", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "x-client-platform": "mobile",
    },
    body: JSON.stringify({ modelHash: h }),
  }).then((r) => console.log("register status", r.status));
}

run().catch(console.error);
```

Adjust m/k to match the server's bloom parameters if you change them.

## Related
