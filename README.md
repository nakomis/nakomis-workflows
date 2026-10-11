# nakomis-workflows — Reusable GitHub Actions workflows for nakomis projects

## Support

If you find this useful, please consider buying me a coffee:

[![Donate with PayPal](https://www.paypalobjects.com/en_GB/i/btn/btn_donate_SM.gif)](https://www.paypal.com/donate?hosted_button_id=Q3BESC73EWVNN&custom=nakomis-workflows)

## Table of Contents

<!-- toc -->

- [Workflows](#workflows)
  * [Publishing a remote](#publishing-a-remote)
  * [Deploying CDK](#deploying-cdk)
  * [Publishing firmware](#publishing-firmware)
- [Conventions](#conventions)
- [Licence](#licence)
- [Support](#support)

<!-- tocstop -->

## Workflows

All three are `workflow_call` workflows: call them from a job with `uses:` and `secrets: inherit` is not needed (they use OIDC roles).

| Workflow | Use for |
|---|---|
| `deploy-static.yml` | Build a Vite app and publish it under a prefix of the `app.nakomis.com` bucket: the shell host (`shell`) or a Module Federation remote (`remotes/<name>`). Hashed assets are immutable; entry points (`index.html`, `remoteEntry.js`) are `no-cache`. |
| `cdk-deploy.yml` | Run a pnpm CDK deploy script with an OIDC role, in a GitHub environment (so `production` keeps its approval gate). |
| `pio-firmware-upload.yml` | Build a PlatformIO project and publish `firmware.bin` to `nakomis-firmware-{env}` via the project's `scripts/build_and_upload.py`. |

### Publishing a remote

The remote's CI role needs the shell's `nakomis-shell-remote-deployer-{env}` managed policy (exported to SSM at
`/nakomis-shell/{env}/remote-deployer-policy-arn`).

```yaml
  deploy-web-sandbox:
    needs: [test]
    if: github.event_name == 'push'
    uses: nakomis/nakomis-workflows/.github/workflows/deploy-static.yml@main
    with:
      environment: sandbox
      aws-role-arn: arn:aws:iam::975050268859:role/nakomis-<repo>-github-ci-sandbox
      working-directory: web
      ssm-bucket-param: /nakomis-shell/sandbox/bucket-name
      ssm-distribution-param: /nakomis-shell/sandbox/distribution-id
      prefix: remotes/<name>
      invalidation-paths: /remotes/<name>/*
```

### Deploying CDK

```yaml
  deploy-infra-prod:
    needs: [deploy-infra-sandbox]
    uses: nakomis/nakomis-workflows/.github/workflows/cdk-deploy.yml@main
    with:
      environment: production
      aws-role-arn: arn:aws:iam::637423226886:role/nakomis-<repo>-github-ci-prod
      command: pnpm ci-deploy-prod
```

### Publishing firmware

```yaml
  firmware:
    uses: nakomis/nakomis-workflows/.github/workflows/pio-firmware-upload.yml@main
    with:
      environment: sandbox
      fleet-env: sandbox
      aws-role-arn: arn:aws:iam::975050268859:role/nakomis-<repo>-github-ci-sandbox
      project-dir: embedded/esp32/boiler
```

The CI role needs `s3:PutObject` on `arn:aws:s3:::nakomis-firmware-{env}/<Project>/*`.

## Conventions

- Action majors: checkout v7, setup-node v7, pnpm/action-setup v6 (before setup-node), configure-aws-credentials v6, setup-python v7, cache v6.
- Node 25, pnpm pinned via each package's `packageManager` field.
- CI role names: `nakomis-<repo>-github-ci-<sandbox|prod>`. Trust policies should accept both the
  `repo:nakomis/<repo>:*` and `repo:nakomis@1488244/<repo>@<repo-id>:*` subject forms.

## Licence

CC0 1.0 Universal — see [`LICENSE`](LICENSE).

## Support

If you find this useful, please consider buying me a coffee:

[![Donate with PayPal](https://www.paypalobjects.com/en_GB/i/btn/btn_donate_SM.gif)](https://www.paypal.com/donate?hosted_button_id=Q3BESC73EWVNN&custom=nakomis-workflows)
