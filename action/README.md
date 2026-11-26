# Sign contract deploys in Stratum from CI

The action builds an unsigned Soroban transaction for your multisig account, stores it on Refractor and
links to it in Stratum, where the signers review and sign it. The workflow only has the account's public
key, so no signing key is kept in GitHub secrets. The logic lives in [`src/lib/ci`](../src/lib/ci), which
other tools can call as well. Inputs and outputs are in [`action.yml`](action.yml).

Use a release tag, for example `tupui/stellar-stratum/action@action-v1`. A release builds the bundle and
publishes it under `action-vX.Y.Z` and the moving `action-vX` tag, so the bundle is not in the repository.
Locally, `npm run build:action` builds it into `action/dist`.

The link goes in the job summary, and in a pull request or commit comment if you set `comment`.

## Deploy

A deploy is two transactions, because the contract can only be created once its code is on the ledger.
Run the workflow with `upload`, sign and send that in Stratum, then run it again with `deploy`.
See [`examples/deploy.yml`](examples/deploy.yml). Constructor arguments go in `args`, a JSON object of
values by name, written as in the app's contract form.

## Upgrade

Upload the new code the same way, then run the contract's upgrade function with `operation: call`. The
example assumes an `upgrade` function that takes the new WASM hash. See
[`examples/upgrade.yml`](examples/upgrade.yml).

## Limits

- Refractor accepts request bodies up to about 100 KB, which is an upload of a WASM of about 73 KB. For a larger transaction the action leaves `refractor-id` empty and sets `artifact-path` to a file with the unsigned XDR. Upload it with `actions/upload-artifact`, and the signers download it, open Stratum on the same network, choose Import in the transaction builder and paste the XDR.
- The transaction uses the account's next sequence number. If another transaction from the account is sent first, run the workflow again.
- The authorization must be the source account's own, which the transaction signature covers. Stratum refuses a call that another address has to authorize.
