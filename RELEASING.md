# Releasing

Releases are automated with [release-please](https://github.com/googleapis/release-please) and npm
[trusted publishing](https://docs.npmjs.com/trusted-publishers). No npm token is stored anywhere.

## How a release happens

1. Pull requests are squash-merged into `main` with [Conventional Commits](https://www.conventionalcommits.org/)
   titles.
2. The `Release` workflow keeps a release pull request open that bumps `package.json`,
   `src/version.ts` and `.release-please-manifest.json`, and updates `CHANGELOG.md`. Versions stay on
   1.0.x: `release-please-config.json` sets `"versioning": "always-bump-patch"`, so features and fixes
   alike move 1.0.4 to 1.0.5.
3. A maintainer reviews and merges the release pull request.
4. The workflow tags `vX.Y.Z`, creates the GitHub release, runs every check, publishes to npm with a
   provenance attestation and attaches a CycloneDX SBOM to the release.

## One-time setup

Do these once, in order, before the first release.

### GitHub (StowCards organisation)

1. Create the public repository `StowCards/stow-node` and push this directory as its root. Make the
   first commit a `chore:` commit (for example `chore: import the Stow Cards Node.js SDK 1.0.0`).
   Version 1.0.0 is already recorded in `package.json` and `.release-please-manifest.json`; a `feat:`
   first commit would make release-please propose 1.0.1 before 1.0.0 is published.
2. Create the team `sdk-maintainers` (referenced by `.github/CODEOWNERS`) and give it write access.
3. In **Settings → Code security**: enable private vulnerability reporting, Dependabot alerts,
   Dependabot security updates and secret scanning with push protection.
4. Protect `main` with a ruleset: require a pull request with one approving review from Code Owners,
   require the `CI`, `No breaking change`, `CodeQL` and `Dependency review` checks, block force pushes, and require linear
   history.
5. In **Settings → Actions → General**, allow GitHub Actions to create pull requests (release-please
   needs this).
6. Create a GitHub App for releases, so the release pull request runs the required checks. GitHub starts
   no `pull_request` workflows for a pull request opened with the workflow's own `GITHUB_TOKEN`, so
   without the app the release pull request waits for checks that never start.
   - In the organisation's **Settings → Developer settings → GitHub Apps**, create a private app (for
     example `stow-node-release`) with webhooks off and these repository permissions: **Contents** read
     and write, **Pull requests** read and write.
   - Install it on `StowCards/stow-node` only.
   - Generate a private key. In the repository's **Settings → Secrets and variables → Actions**, add
     the app's ID as the variable `RELEASE_APP_ID` and the private key (the whole `.pem` file) as the
     secret `RELEASE_APP_PRIVATE_KEY`.

   The `Release` workflow uses the app's token whenever `RELEASE_APP_ID` is set. Until it is, it falls
   back to `GITHUB_TOKEN`: a maintainer then closes and reopens each release pull request (or pushes a
   commit to its branch) to start the checks.

7. Create an environment named `npm` (**Settings → Environments**), limited to the `main` branch, with
   the maintainers as required reviewers if you want a manual gate before each publish.

### npm

1. Create the npm organisation `stowcards` and require two-factor authentication for its members.
2. Publish the first version, `1.0.0`. npm configures trusted publishing per package, so if npm will not
   let you add a trusted publisher before the package exists, publish once by hand from a clean clone:

   ```bash
   npm ci
   npm run ci
   npm publish --access public    # prompts for your 2FA code
   ```

   Then record the baseline so release-please starts from it:

   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   gh release create v1.0.0 --title v1.0.0 --notes-file CHANGELOG.md
   ```

3. On npmjs.com, open the package's **Settings → Trusted publishing** and add a GitHub Actions publisher:
   organisation `StowCards`, repository `stow-node`, workflow `release.yml`, environment `npm`.
4. Still in the package settings, set **Publishing access** to "Require two-factor authentication and
   disallow tokens", so only the workflow can publish.

From then on, every release follows the automated flow above.

## Checking a release

```bash
npm view @stowcards/sdk version
npm audit signatures   # in a project that depends on it: verifies the provenance attestation
```

## Hotfixes and pre-releases

- To force a version, add a `Release-As: 1.0.9` footer to a commit on `main`. Keep it within 1.0.x.
- A broken release is fixed forward with a new patch release. Use `npm deprecate @stowcards/sdk@1.0.4 "reason"`
  rather than `npm unpublish`.
