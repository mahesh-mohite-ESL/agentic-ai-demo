---
name: invoice-ai-pipeline
description: Set up an invoice PDF extraction microservice (Python, FastAPI, Claude API) and the GitHub Actions pipelines for Claude pull request review and bug auto-fix in the current folder, by copying a tested reference project. Takes one argument - service, pipelines or all.
argument-hint: service | pipelines | all
disable-model-invocation: true
---

# invoice-ai-pipeline

Sets up, in the current folder, an invoice PDF extraction microservice (Python, FastAPI, Claude API) and the GitHub Actions pipelines for Claude pull request review and bug auto-fix. Everything is copied from a tested reference project.

**Argument:** exactly one of `service`, `pipelines` or `all`. If the argument is missing or anything else, stop and tell the user the three valid values. Do not guess.

## Reference project

- Repository: `https://github.com/mahesh-mohite-ESL/invoice-extractor-ai-ci-pipeline-demo`
- Commit: `9c1a418`

## Rules (apply to every stage)

1. **Copy the files exactly.** Never rewrite, fix, reformat or improve them. The service contains five deliberate bugs that the auto-fix pipeline repairs later. "Fixing" a bug defeats the purpose of the project. If something looks wrong, leave it and mention it in the summary.
2. **Never overwrite an existing file that differs without asking.** Compare each destination with the source first. Identical files are skipped silently. If a destination exists and differs, list those files and ask the user (overwrite, keep existing, or abort) before touching them. Files that do not exist yet are created without asking.
3. **Never run `git init`, `git commit` or `git push`.** Do not run any other command that changes git state either.
4. **Never ask for an API key or token.** Do not read, create or prompt for secrets. Tell the user in the summary where keys are configured (see `docs/SETUP-GUIDE.md` once copied).
5. **At the end of each stage, summarise** what was created (and what was skipped), the test result, and name the next step.

## Fetching the reference

Use a temporary folder **outside the project** (for example `$env:TEMP` on Windows or `mktemp -d`; the session scratchpad is also fine). Name it uniquely.

1. `git clone https://github.com/mahesh-mohite-ESL/invoice-extractor-ai-ci-pipeline-demo <tmp>/ref`
2. `git -C <tmp>/ref checkout 9c1a418`
3. Copy entries from `<tmp>/ref` into the current folder, preserving relative paths and file bytes (use `cp -r` / `Copy-Item -Recurse`; do not run text transformations, so line endings are preserved).
4. When the run ends, **delete the temporary folder**, even if a step failed. Confirm it is gone. Cloning is read-only for git; it is the only git operation allowed.

If the clone or checkout fails (network, access, missing commit), stop, report the error, clean up, and do not improvise a substitute.

If both `service` and `pipelines` are needed (`all`), clone once and reuse the clone for both stages.

## Stage: service

Copy these entries:

- `app/`
- `samples/`
- `tests/`
- `tools/`
- `pyproject.toml`
- `requirements.txt`
- `requirements-dev.txt`
- `Dockerfile`
- `.dockerignore`
- `.gitignore`
- `.gitattributes`
- `README.md`
- `docs/PLANTED-BUGS.md` (create `docs/` if needed)

Then:

1. Create a virtual environment: `python -m venv .venv` (use `py -3 -m venv .venv` on Windows if `python` is not found). If `.venv` already exists, reuse it and say so.
2. Install into it: `.venv/Scripts/python -m pip install -r requirements-dev.txt` (Windows) or `.venv/bin/python -m pip install -r requirements-dev.txt` (POSIX).
3. Run pytest with the venv's Python: `.venv/Scripts/python -m pytest`.
4. **Expected result: `28 passed`.** Report the actual result. If it differs, show the failing output and do not edit any file to make it pass.

If Python is not installed, copy the files, skip steps 1 to 3, and say so clearly.

Summary then names the next step: run `/invoice-ai-pipeline pipelines`.

## Stage: pipelines

**Precondition:** `app/main.py` must exist in the current folder. If it does not, stop and tell the user to run the `service` stage first. Copy nothing.

Copy these entries:

- `.github/`
- `scripts/`
- `.claude/` (merge into the existing `.claude/` folder; this skill's own `.claude/skills/invoice-ai-pipeline/` must be left untouched, and if the reference has a differing file at the same path, apply Rule 2)
- `package.json`
- `CLAUDE.md`
- `docs/flawed-change.patch`
- `docs/SETUP-GUIDE.md`

Then, if Node.js is installed (`node --version` succeeds):

1. Run `npm run test:autofix`.
2. **Expected result: `12 passing`.** Report the actual result. If it differs, show the output and do not edit any file.

If Node.js is not installed, skip the test and say so.

Summary then names the next step: follow `docs/SETUP-GUIDE.md` to push the project to GitHub and configure the repository secrets and settings yourself (this skill does not run git or handle keys).

## Stage: all

Run the `service` stage, then the `pipelines` stage. Give a summary at the end of each stage. If `service` fails to copy, do not start `pipelines`. If `service` tests do not show `28 passed`, still continue to `pipelines` only if `app/main.py` exists, and flag the discrepancy in the final summary.
