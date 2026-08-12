# vega-reproducer-staging

vega-reproducer-staging is the staging counterpart of
[vega-reproducer](https://github.com/Ad-Astra-Computing/vega-reproducer). It
independently rebuilds attested Nix outputs and attests the result to the Vega
staging control plane, which is where the reproduction loop is exercised before
a change reaches production. The workflow is kept identical to the production
one apart from the control-plane URL, so testing here tests what production
runs.

## How it works

The Vega staging control plane dispatches a reproduction job to this repository for an
output that has reached agreement among distinct, independent owners. The job
rebuilds the derivation from its recorded provenance (a flake reference, an
attribute, and a locked revision) on a fresh GitHub-hosted runner, then attests
the rebuilt output to Vega over a GitHub Actions OIDC token. Vega compares the
reproduced fingerprint against the agreed one: a match satisfies the
independent-rebuild requirement, and a mismatch retracts any existing signature.

The build is treated as untrusted. It runs in the Nix sandbox, and the workflow
drops the OIDC token-minting capability before building, so a malicious flake
cannot re-mint the runner identity.

## Triggering

The workflow runs only through `workflow_dispatch`, taking the provenance as
inputs (`flake_ref`, `attr`, `rev`). It is normally triggered by the Vega control
plane and can also be run manually from the Actions tab.

## Security

Actions are pinned to commit SHAs, the default workflow token is read-only, and
the checkout does not persist credentials. See [SECURITY.md](SECURITY.md) for how
to report a vulnerability, and https://docs.vega-cache.dev/reproducibility for the
trust model.

## License

BSD-3-Clause. See [LICENSE](LICENSE).
