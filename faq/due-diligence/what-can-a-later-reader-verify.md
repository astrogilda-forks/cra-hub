---
Status: ⚠️ Draft
---

# What can a later reader verify in a record?

Two kinds of reader come to a compliance record after it was made. A market surveillance authority can ask a steward for its cybersecurity policy on reasoned request ([[Article 24]]), and for a high-risk AI system a competent authority can ask for "access to the automatically generated logs" under [Article 21(2) of the AI Act](https://eur-lex.europa.eu/eli/reg/2024/1689/oj). A manufacturer exercising due diligence on a component ([[due-diligence/what-is-due-diligence]]) reads the documentation the project publishes. Neither was present when the record was written, and both need to know what they can check without trusting its author.

A document returned by email or a questionnaire answer, taken alone, establishes only that someone wrote it. It carries no digest, so the reader cannot tell whether it matches what the author holds, and often no release identifier, so it cannot be tied to a particular build.

A record can be checked by someone who did not write it when it has four properties:

- **Integrity**: the content has a cryptographic digest, so a change to any byte is detectable.
- **Origin**: a signature over those exact bytes verifies against a key the reader obtained separately, for example one the project publishes alongside its releases.
- **Subject**: the record names what it covers by digest, such as a release artifact, not only by name and version.
- **Time**: the record states when it was made, ideally with a timestamp from a third party.

Open formats already provide these. An [in-toto statement](https://github.com/in-toto/attestation) binds claims to the digests of the artifacts they describe. A [DSSE envelope](https://github.com/secure-systems-lab/dsse) signs the statement together with its payload type, so there is no doubt about which bytes were signed. [RFC 8785](https://www.rfc-editor.org/rfc/rfc8785) canonical JSON makes the same data serialise to the same bytes, so author and reader compute the same digest. Many projects already publish SBOMs and build provenance in these forms ([[stewards/sbom]]).

What a reader cannot verify from the record alone is whether its claims are true. A valid signature shows that someone controlling a particular signing key signed those exact bytes and that the signed content has not changed. It identifies a person or organisation only when the reader has separate, current evidence binding that key to them. The reader still decides whether to trust the signer, the signer's authority, the process that produced the record and the claims themselves.

The subject property is the one a reader can check first, before any signature. Given the document and the artifact it describes, one command recomputes the artifact's digest and compares it with the digest the document names for its own subject (the component a CycloneDX SBOM describes, a TEA artifact's checksums, or an in-toto statement's subjects):

```
uvx agent-evidence-admission==0.1.0 digest-check sbom.json artifact.jar
```

It passes only on a match. A digest listed for a dependency does not count, and a document that names no digest for its subject fails, because nothing could be compared.

To watch these checks run, a public test corpus of signed records, including deliberately broken ones a verifier must reject, replays with one command:

```
uvx --python 3.13 agent-evidence-vectors==0.17.5
```

On 9 October 2026 version 0.17.5 reported "281 vectors, 281 pass, 0 fail". The corpus and its documentation are in [its repository](https://github.com/probityai/agent-evidence-vectors).
