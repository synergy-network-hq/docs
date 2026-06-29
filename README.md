# Synergy Validator Workspace

Canonical filesystem template and validation tooling for Synergy Testnet validators.

This repository is the source of truth for how Linux validators must be laid out after standardization. Live validators use the `node` Linux user and these canonical paths:

- `/home/node/.synergy/testnet/nodes/validator-workspace`
- `/etc/synergy/validator`
- `/etc/synergy/validator/keys`
- `/var/lib/synergy/validator`
- `/var/log/synergy/validator`
- `/var/backups/synergy/validator`
- `/opt/synergy/bin`
- `/opt/synergy/scripts`
- `/etc/systemd/system/synergy-validator.service`

Files that would contain private keys, validator identity, node identity, WireGuard private keys, passwords, tokens, or RPC secrets are committed only as `.example` files with placeholder values.

## Contents

- `template/` mirrors the target validator filesystem.
- `template/MANIFEST.yaml` defines ownership, permissions, generated status, and parity expectations.
- `template/IDENTITY_FIELDS.yaml` defines validator-specific values that may differ.
- `template/DRIFT_CHECKS.yaml` defines byte-identical, masked-identical, expected-different, and forbidden paths.
- `template/opt/synergy/scripts/` contains install, migration, verification, rollback, and block-time diagnostic scripts.
- `docs/` explains operator process, migration, rollback, secret handling, and control-panel integration.
- `tests/` validates the template structure and checks for likely committed secrets.

## Validation

Run:

```bash
tests/test-required-files.sh
tests/test-template-no-secrets.sh
python3 tests/test-mask-identity.py
tests/test-render-config.sh
```

The scripts are dry-run friendly by default and refuse blind secret overwrites.
