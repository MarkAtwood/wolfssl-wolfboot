# Contributing to wolfBoot

## Contributor Agreement

External contributors must sign a contributor agreement before pull requests can be merged.
When you open your first PR, a wolfSSL team member will ask you to email support@wolfssl.com
referencing the PR. The agreement is tracked via wolfSSL's Zendesk ticketing system. Once
signed, your PR will be approved for CI testing.

## Fork Workflow

Do not push branches to this repository. Fork to your personal GitHub account and open pull
requests from your fork.

## Source Code Rules

CI enforces all of these on every PR. Violations block merge.

### Formatting

- **No trailing whitespace.** Files must end with a newline.
- **No hard tabs** in C or header files. Makefiles are exempt.
- **ASCII only.** No non-ASCII bytes in source files. All code, comments, and string literals
  must be pure ASCII.
- **No CR characters** (`\r`). Use Unix line endings.

### C Style

- **C comments only.** Use `/* */`, not `//`, in all `.c` and `.h` files.
- Keep functions short and focused. wolfBoot targets constrained MCUs with limited stack.
- HAL implementations live in `hal/<target>.c` and `hal/<target>.ld`. Do not mix target-specific
  code into the core bootloader (`src/`).

### Spelling and Linting

- **cppcheck** runs on all C source files (`make cppcheck`). Fix any flagged issues before
  submitting.
- Spell-check comments and documentation. wolfSSL uses `codespell` in CI on related projects.

### AI Attribution

- **No AI attribution in commits.** CI rejects commits containing `Co-authored-by:` or
  `Signed-off-by:` trailers that reference:
  - `noreply@anthropic.com`
  - `noreply@openai.com`
  - `+Copilot@users.noreply.github.com`
  - Any `[bot]@users.noreply.github.com` address
- Commits authored by bot email addresses are also rejected.
- **Do not add these trailers.** Your PR will fail CI if they are present.

## PR Requirements

Every PR should include:

- **Description** of the scope of the fix or feature
- **Target information** — which hardware targets are affected or were tested
- **Test description** — how the change was tested (simulator, unit tests, or real hardware)
- For new HAL targets: include the example `.config` file in `config/examples/`

All CI checks must pass before merge.

## Testing Before Submitting

At minimum, run the simulator build and unit tests:

```bash
cp config/examples/sim.config .config
make keytools
make
make -C tools/unit-tests run
```

If your change touches signing, key generation, or image parsing, also run the keytools tests:

```bash
make -C tools/keytools test
```

For HAL changes, test on the actual target hardware if possible. CI runs simulator and
QEMU-based tests automatically, but hardware-specific regressions are the contributor's
responsibility.

## Adding a New Hardware Target

1. Create `hal/<target>.c` implementing the wolfBoot HAL API (see `include/hal.h`)
2. Create `hal/<target>.ld` with the linker script for the target's flash layout
3. Add a `config/examples/<target>.config` with sensible defaults
4. Document the target in `docs/Targets.md` or add a dedicated `docs/<target>.md`
5. If the target needs a new `ARCH`, update `arch.mk` and the relevant boot entry point
   in `src/boot_<arch>.c`

## Security Reports

Do not open GitHub issues for security vulnerabilities. Report them to support@wolfssl.com.
Include a description of the vulnerability, affected targets or configurations, and any
proof-of-concept if available.
