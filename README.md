# run-agent

## Headed mode

`HEADED=true` runs the agent under `xvfb-run`. The action does not install
Xvfb — the runtime image ships it (`runtime/Dockerfile.base`), and job steps
on those runners have no root, so an `apt-get` fallback could not work there
anyway. If `xvfb-run` is missing the agent simply runs headless.

## Extra browser switches

`WOPEE_BROWSER_ARGS` adds Chromium switches, whitespace-separated, e.g.
`--auth-server-allowlist=* --auth-negotiate-delegate-allowlist=*` for a site
behind Negotiate (Kerberos/NTLM) SSO. Set it as a project or suite variable in
Commander (it arrives in `client_payload.env`) or in the project repo's
`.github/workflows/.env` (loaded after the payload, so it wins). Interactive
runs pass it to the Chrome this action starts; headless runs leave it to
pw-agent, which gives it to its own browser. Unset, nothing changes.
