# Protected bootstrap

Bootstrap is performed by `sawa-cell` while `sawa-cell` is the only account
with write or administrative access. There is no protected-history claim for
the initial commit itself. The dispatcher grant is deliberately made only
after the owner-only protection snapshot has passed; the live rehearsal then
proves the grant without authorizing any production mutation.

1. Create a new **public personal repository owned by `sawa-cell`** with
   default branch `main`. Do not initialize Actions secrets, variables, apps,
   webhooks, Pages, environments, deploy keys, releases, packages, or OIDC.
2. Add exactly this scaffold as the initial commit. Resolve the public commit
   tree recursively by SHA and require exactly these five **blob** paths, with
   no sixth blob (directory tree entries do not count):
   `.github/CODEOWNERS`, `.github/workflows/approve-bound-challenge.yml`,
   `BOOTSTRAP.md`, `README.md`, and `control-policy.json`. Fetch all five blobs,
   byte-compare them to this private scaffold, and record each SHA-256. Confirm
   that the workflow contains no `uses:` step, no checkout, and only
   `contents: read`.
3. Create Environment `limen-production-approval` with exactly one required
   reviewer: user `sawa-cell` (`315051190`). Enable prevent-self-review,
   disable administrator bypass, select custom deployment branches/tags, and
   add exactly one Branch rule named `main`.
4. Protect `main`: require pull requests and one code-owner approval, dismiss
   stale approvals, require approval of the latest push by someone other than
   its pusher, apply rules to administrators, and disable force pushes and
   deletion. Do not configure a bypass actor.
5. While `sawa-cell` is still the sole write/admin principal, record repository
   ID, workflow ID, Environment ID, exact `main` SHA, collaborator list, and the
   full REST/UI protection snapshot. Independently verify the owner, reviewer,
   and requester numeric IDs and every setting above.
6. Populate those IDs and the exact `main` SHA in the private
   `.limen/production-approval-policy.json`, set `enabled` to `true`, and run the
   owner-only pre-grant verification:

   ```text
   node scripts/production-approval.mjs verify-bootstrap \
     --policy .limen/production-approval-policy.json \
     --output bootstrap-pre-grant.json \
     --admin-token-env GH_TOKEN
   ```

   The temporary token needs read access to public repository administration
   metadata only. Never store it in this repository or its Actions settings.
7. Invite `Goocz` (`226841374`) with the built-in **Write** role, never a custom
   role and never Maintain/Admin. Wait for `Goocz` to accept. As `sawa-cell`,
   verify `GET /repos/sawa-cell/limen-approval-control/collaborators/Goocz/permission`
   returns that exact user ID, `permission: write`, and `role_name: write`.
   Also require the collaborator-list permission map to report `push: true`,
   `maintain: false`, and `admin: false`. Record both responses. No other
   non-owner collaborator is allowed.
8. Re-run `verify-bootstrap` into `bootstrap-post-grant.json`. Byte-compare the
   protected repository/workflow/environment/ref/branch-protection portions of
   the pre-grant and post-grant snapshots and recheck the exact collaborator
   permission. Any change other than the recorded `Goocz` Write grant is a
   denial.
9. Immediately before each rehearsal, `Goocz` independently obtains the
   current sanitized D1 account-set count and digest, enters both exact values,
   and types `CURRENT_SANITIZED_D1_ACCOUNT_SET_INDEPENDENTLY_OBTAINED`. This is
   explicitly human-attested input, not provider-verified evidence. The count
   must be at least one and the digest must be 64 lowercase hex characters.
   The rehearsal workflow binds those bytes in a current-run prerequisite but
   never reads an Actions secret or production environment and never calls
   Sites, Cloudflare, Bybit, or another production provider.
10. Before approving any positive or deliberately malformed public run,
    `sawa-cell` independently reconstructs the account set exactly as
    `lib/recovery-readiness.ts` does: order the live non-revoked, verified rows
    by `environment, exchange, account_uid_hash, id`; map them without extra
    fields to `{environment,exchange,accountUidHash}`; preserve that order in
    `accounts`; then SHA-256 the UTF-8 bytes of
    `JSON.stringify({schema:'limen.production-account-set.v1',accounts})`.
    The independently derived count/digest must equal the private challenge,
    `sawa-cell` must have access to every bound account, and zero positions and
    orders must be reconfirmed immediately before the Environment review.
11. Complete the rehearsal matrix before any production workflow is allowed.
    For every case, dispatch the public run only after GitHub's next timestamp
    second following the private challenge artifact `created_at`, and remain
    inside the bound 300-second freshness window:

    - `positive`: one `Goocz` dispatch and one `sawa-cell` approval using the
      exact review comment; expect a complete challenge/selection/revalidation/
      audit proof.
    - `no-approval`: leave the protected job waiting during the resolver; do
      not approve, reject, or cancel it. Expect resolver exit 4 and no selection.
    - `wrong-reviewer`: let `Goocz` attempt self-review, then leave the protected
      job waiting during the resolver; do not let `sawa-cell` approve, reject,
      or cancel it. Expect self-review prevention and resolver exit 4.
    - `wrong-comment`: `sawa-cell` approves with the fixed non-empty comment
      `WRONG_BOUND_CHALLENGE_COMMENT`; expect resolver exit 3 and no selection.
    - `duplicate`: dispatch the identical four public fields twice; expect
      resolver exit 3 and no selection.
    - `main-movement`: while public `main` is SHA A, have `Goocz` author a
      pre-staged pull request that changes only the already-declared `README.md`
      documentation path and have `sawa-cell` give the required code-owner
      approval. Dispatch the A-bound public run but
      hold its Environment approval. Within the private resolver window, merge
      the protected A-to-B pull request, then let `sawa-cell` approve the still
      waiting A run with the exact comment. Expect the exact live-main error,
      resolver exit 3, and no selection. Before repinning B, resolve the exact
      B tree and prove it still has only the same five paths, only `README.md`
      changed from A, the approval workflow/CODEOWNERS/control policy/BOOTSTRAP
      bytes and recorded hashes are identical, and every protection setting is
      semantically unchanged at B.

12. After the controlled main-movement case, record the new public `main` SHA,
    update only the private policy pin, re-run both bootstrap/collaborator
    checks, and complete one final `positive` rehearsal against the new SHA.
    Freeze the public commit, collaborator set, and all protection settings.

Any failed, unreadable, ambiguous, paginated, rate-limited, renamed, moved,
partially pinned, or reconfigured state is denial. Loss of the control
repository leaves LIMEN fenced; it does not authorize a break-glass mutation.
