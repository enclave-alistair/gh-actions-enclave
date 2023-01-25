# Using Enclave to connect from inside a Github Action

This repo has got an example workflow that connects to a on-prem resource, securely, 
from inside a GitHub Actions Runner.

Some attributes of this setup:

- At no point are any public endpoints exposed anywhere.
- There is no additional network configuration on either end.
- The connectivity is point-to-point, not relayed or proxied.
- The transport is protocol-agnostic (i.e. you can route anything over it).

The basic workflow structure looks like this:

```yaml
build:

name: Build Something

runs-on: ubuntu-latest

steps:

- name: Setup Enclave
  uses: enclave-alistair/enclave-setup-action@v1
  with:
  # Enrolment key is a secret (brings you into your enclave account)
  # Use an Ephemeral enrolment key, to remove the system from your account once the action is finished.
  enrolment-key: ${{ secrets.ENCLAVE_ENROLMENT_KEY }}

- name: Wait for Connection to your on-prem server 
  # Name here comes from the config in the portal.
  run: enclave waitfor on-prem-server.enclave

- name: Query on-prem server
  run: curl on-prem-server.enclave

```

You can check out the Actions results for a run [here](https://github.com/enclave-alistair/gh-actions-enclave/runs/2841198674).

The enclave setup in this test, done from the [Enclave Portal](https://portal.enclave.io):

- An on-prem web-server running in my network, running enclave, and tagged with `gh-test-server`.  DNS configured to give it the name `on-prem-server.enclave`.
- An ephemeral enrolment key (`GH Actions Test Runner`), that auto-tags anything enrolled with that key with `gh-runner`.
- A policy `CI -> OnPrem`, that says anything tagged with `gh-runner` can connect to `gh-test-server`.

That config is all that's needed; once the runner starts, it auto-enrols, policy defines it should connect to the on prem server, and it does so.

We use an ephemeral enrolment key, so when the runner stops, the system is removed from your organisation.
