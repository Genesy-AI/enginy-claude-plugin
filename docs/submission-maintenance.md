# Submission maintenance

The Claude plugin lives in this repository. Its manifest is `.claude-plugin/plugin.json`, its hosted connector is `.mcp.json`, and its 39 skills live in `skills/`.

## Validate the package

Run from the repository root with a current Claude Code release:

```bash
claude --version
claude plugin validate .
```

Claude Code 2.1.281 and later accepts the directory listing fields without warnings. Older validators may report `icon`, `documentationUrl`, `supportUrl`, `privacyPolicyUrl`, and `termsOfServiceUrl` as unknown fields. Keep these fields: the directory uses them for the listing.

The validator may suggest a marketplace install command. This repository contains a plugin without a marketplace catalog, so the README documents loading it with `--plugin-dir`. A successful package check does not establish OAuth access, tool behavior, or directory approval.

The [manifest reference](https://code.claude.com/docs/en/plugins-reference#directory-listing-fields) documents the listing fields. The [command reference](https://code.claude.com/docs/en/plugins/cli-reference) documents validation and session loading.

## Check the listing

Open the existing submission in Claude's **Submissions** dashboard. Compare the latest version and commit with the source before making changes.

1. Read **Review**, including expanded validation and directory-policy findings. Record actionable feedback separately from findings marked "No action needed."
2. Inspect **Listing** for the name, description, icon, publisher, documentation, support, privacy, and terms links.
3. Open the public links and check their content. The support page should show a usable contact method, and policy pages should identify Enginy.
4. Read **Versions** to confirm which commit passed checks, which version has a publish request, and whether anything is live.

Keep review observations, screenshots, and reviewer-access details outside the public package. Record the observation time, version, commit, review status, icon status, and unresolved feedback for the next check.

## Listing artwork

`assets/enginy-logo.png` is the existing 512 × 512 Enginy mark. The manifest references it, and the README uses the same asset.

The listing dashboard also accepts an uploaded icon. Its preview and review status are separate from the manifest path. A letter placeholder beside the package path does not establish that the uploaded icon is missing.

Before replacing an icon, inspect the uploaded preview and any reviewer feedback. Use the approved Enginy branding and check small-size legibility. Verify the saved preview and review status after an upload.

Use real product captures for any requested screenshots or demo. Check that captures omit credentials and customer data, and that any recording link is accessible to reviewers.

## Apply review feedback

Make each correction in the file that owns it. Preserve the plugin identity, connector endpoint, and existing confirmation steps for spending credits, sending messages, and activating campaigns.

Validate the final package and record which checks ran. For a behavior change, exercise the affected workflow in an appropriate test workspace and describe any untested steps.

Publishing source changes and changing the tracked branch can affect the version under review. Read the dashboard's current explanation before taking either action. Obtain authorization before pushing source changes or cancelling a pending review.

After an authorized update, confirm the imported commit and saved listing in the dashboard. A successful scan, a publish request, and a live listing are distinct states; report the state that the dashboard shows.

For daily checks, notify the maintainer when feedback, approval, rejection, a completed correction, or a required action changes. Keep unchanged pending reviews quiet.
