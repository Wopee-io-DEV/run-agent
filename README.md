# run-agent

## Headed mode

`HEADED=true` runs the agent under `xvfb-run`. The action does not install
Xvfb — the runtime image ships it (`runtime/Dockerfile.base`), and job steps
on those runners have no root, so an `apt-get` fallback could not work there
anyway. If `xvfb-run` is missing the agent simply runs headless.

## Extra browser switches

`WOPEE_BROWSER_ARGS` adds Chromium switches, whitespace-separated, e.g.
`--auth-server-allowlist=*.corp.example.com` for a site behind Negotiate
(Kerberos/NTLM) SSO. Set it as a project or suite variable in
Commander (it arrives in `client_payload.env`) or in the project repo's
`.github/workflows/.env` (loaded after the payload, so it wins). Interactive
runs pass it to the Chrome this action starts; headless runs leave it to
pw-agent, which gives it to its own browser. Unset, nothing changes.

## Kerberos (Windows SSO)

An app whose identity provider answers only `401 WWW-Authenticate: Negotiate`
needs a Kerberos ticket before the browser starts — on Linux Chromium cannot
turn a name and password into one (Windows does that for the browser, which
is why the same Playwright setup works on a domain-joined desktop). Set, as
Commander project or suite variables (a suite overrides the project):

| Variable | |
|---|---|
| `KERBEROS_PRINCIPAL` | default account, `user@REALM` (the AD DNS domain, upper case), or a bare `user` when the fleet sets `KERBEROS_REALM`. |
| `KERBEROS_PASSWORD` | its password. Masked in the log, blanked after sign-in. |
| `KERBEROS_ACCOUNTS` | optional, per test: `US001/TC002=alice; TC005=none; US003=bob; *=bob`. The most specific selector for this run's user story / test case wins; the alias's account is `KERBEROS_PRINCIPAL_<ALIAS>` + `KERBEROS_PASSWORD_<ALIAS>`; `none` = this test without SSO; no match = the default account. |
| `KERBEROS_SSO` | `off` = no sign-in (e.g. a suite of a project whose other suites use SSO). |
| `KERBEROS_AUTH_SERVERS` | optional `--auth-server-allowlist`, default `*.<realm's parent domain>` (`*.corp.example` for `AD.CORP.EXAMPLE`). |
| `KERBEROS_REALM`, `KERBEROS_KDC` | optional; usually set per runner fleet in infra |

One run is one test case, so each test signs in with its own account. None of
the `KERBEROS_*` variables is offered to the agent as a `${KEY}` placeholder.

The step "Kerberos sign-in" runs the runtime image's `wopee-kerberos login`:
one `kinit` (no retry, so parallel runs cannot lock the account), then
`KRB5_CONFIG`, `KRB5CCNAME` and the allowlist switch for every later step.
The runner host needs TCP 88 to the realm's KDCs. A failed sign-in fails the
job with the reason (KDC unreachable, wrong password, expired or locked
account). Unset, the step is skipped.

Values of variables whose names look secret (`PASS`, `PWD`, `SECRET`, `TOKEN`,
`CREDENTIAL`, `PRIVATE`, `KEYTAB`, `…_KEY`) are masked before they enter the
job environment, from `client_payload.env` and from `.github/workflows/.env`.
