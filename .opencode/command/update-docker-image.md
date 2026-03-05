---
description: Update the `Dockerfile` inside this repo given the package version from the input, and use github CI to build and test the new image.
---

## Usage

```
/update-docker-image $VERSION
```

**Arguments (`$VERSION`):** a list of pinned package versions, such as "sglang==0.5.9
vllm==0.10.1 torch==2.10.1"

## Workflow

1. Update the basic version requirements in @pyproject.toml according to the input
   version requirement. Validate that the provided versions exist in pip registry.
   Otherwise, exit and raise an error report to the user. In this step, remain other
   dependency version unchanged.

1. For the following packages, browse the github repo and check dependency version
   mismatch with AReaL

- For sglang, check
  https://github.com/sgl-project/sglang/blob/${version}/python/pyproject.toml
- For vllm, check https://github.com/vllm-project/vllm/blob/${version}/pyproject.toml

3. Figure out how to resolve the dependency conflict, and report to user.

If there's no inconsistency between the above packages, and it only conflicts with
AReaL, update the AReaL's version.

If the above packages have mutual conflict, summarize and report to user, then you MUST
ask the user for resolution.

Output format:

```
Summary

---
Updated Pacakges (no actions required):
- ${name}, ${pacakgeA} requires ${packageAVersion}, ${pacakgeB} requires ${packageBVersion}, AReaL specified ${version}, updated to ${version}
- ...

---
Mismatched Packages (need to resolve):

- ${name}, ${pacakgeA} requires ${packageAVersion}, ${pacakgeB} requires ${packageBVersion}
- ...

```

4. According to the user's conflict resolution, update and validate @pyproject.toml

update @pyproject.toml according to user's requirement. You may properly use
"override-dependencies".

5. validate that the conflicts in step 3 have been all resolved. If not, return to step
   3 and you must ask the user again.

1. use `uv` to lock against the version provided. If error occurs, return step 3, you
   must ask the user for resolution before modifying and trying again. \`

1. According to the finalized @pyproject.toml, update the `Dockerfile` inside this repo.
   The Dockerfile uses `ARG BASE_IMAGE` and `ARG VARIANT` build arguments to support
   both sglang and vllm variants.

You should only install dependencies related to the corresponding inference framework.

8. Use `/create-pr` command to create a PR, and trigger the CI workflow manually in
   ".github/workflows/build-docker-image.yml".

The docker build CI should automatically trigger testing upon success. Debug until the
overall workflow successes.

If you encouter issues that cannot be resolved, ask the user for help.
