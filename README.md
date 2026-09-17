# run-agent

## Headed mode

`HEADED=true` runs the agent under `xvfb-run`. The action does not install
Xvfb — the runtime image ships it (`runtime/Dockerfile.base`), and job steps
on those runners have no root, so an `apt-get` fallback could not work there
anyway. If `xvfb-run` is missing the agent simply runs headless.
