# LIMEN approval control

This repository is a public, secret-free source of independently reviewed
GitHub Environment approval records. It never receives LIMEN source, private
repository names or IDs, production account data, credentials, private commit
SHAs, run IDs, epochs, artifacts, or human-readable incident details.

The only public request fields are a protocol version, a coarse operation, a
random 256-bit nonce, and a SHA-256 digest of a private canonical challenge.
The protected job performs syntax validation and no production action.
`approval-rehearsal` is a separate coarse operation whose private stage is
`verify-control` and whose compensation mode is `no-mutation`; its approval can
never satisfy a production operation/stage pair.

## Human roles

- `Goocz` dispatches a run using the four values emitted by the private
  challenge builder.
- `sawa-cell` first inspects the exact private target and its prerequisite
  evidence, independently checks every bound Bybit account, and then approves
  the waiting Environment deployment with comment
  `APPROVE_BOUND_CHALLENGE_V1`.
- If `sawa-cell` dispatches the run, self-review prevention blocks approval and
  the private verifier rejects the requester identity.

Follow [BOOTSTRAP.md](BOOTSTRAP.md) exactly: take and verify the owner-only
pre-grant snapshot, add `Goocz` with non-admin Write permission, reverify the
unchanged controls, complete every non-mutating rehearsal, and only then freeze
the control. Do not add Actions, scripts, secrets, variables, packages, Pages,
releases, webhooks, OIDC, deployment credentials, or external integrations.

This record is an independent attestation only for an unchanged compliant
private consumer. Any write/admin principal in the private repository can
bypass that consumer or use its repository-level production credential.

## Protected-main rehearsal

Bootstrap includes a README-only protected-main movement rehearsal before any production credential is enabled.
