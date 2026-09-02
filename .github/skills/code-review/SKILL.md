---
name: code-review-official-docs
description: Use during GitHub.com Copilot Code Review when a finding depends on Microsoft or GitHub product specifications, APIs, SDKs, configuration, permissions, limits, or support state, and must be verified against Microsoft Learn or GitHub Docs.
---

# code-review-official-docs: Code Review with Official Documentation

Use this workflow during GitHub.com Copilot Code Review. It applies when the correctness of a finding depends on external official specifications from Microsoft or GitHub, such as an API, SDK, Azure resource, .NET/C# behavior, Bicep resource, GitHub Action, repository setting, permission, ruleset, or documented platform behavior.

## Classify the basis of each finding

Before creating a finding, classify why the finding is correct:

### Repository-grounded finding

The finding can be fully justified from the repository's own code, design documents, tests, configuration, or the diff itself.

- Official documentation search is optional.
- An official documentation URL is not required.

Examples: obvious logic errors, null/undefined handling bugs, unreachable code, resource leaks, race conditions, mismatch with an interface or contract defined in this repository, changes that break existing tests, self-contradiction inside a design document, mismatch between an implementation and a design document in the same repository, or a security issue that is evident from the code itself.

Do not search Microsoft Learn or GitHub Docs only to satisfy such findings, and do not cite loosely related documentation for form.

### Documentation-grounded finding

External Microsoft or GitHub specifications or product behavior are required to determine whether the finding is correct.

- Verifying the official documentation with the appropriate MCP tool is required.
- Create a definitive compliance or non-compliance finding only when the official documentation was actually retrieved during this review.
- Record the canonical official URL that justifies the finding inside the same review comment.

Examples: a change violates an Azure resource specification or limit, a Bicep resource property is used incorrectly, an implementation disagrees with the documented behavior of a .NET API, a workflow disagrees with the documented behavior of a GitHub Action, an assumption about GitHub permissions or rulesets is wrong, or a statement about preview/GA, licensing, prerequisites, or limits is wrong.

Decide whether official documentation is required based on whether external official specifications are needed to prove the finding, not merely on whether the change touches Azure, Microsoft, or GitHub.

## Verifying Microsoft documentation

For Azure, .NET, C#, Microsoft APIs, Bicep, Azure services, and other Microsoft technologies, use the Microsoft Learn MCP Server and treat Microsoft Learn as the authoritative source. Reference implementation: https://github.com/microsoftdocs/mcp

Use these tools explicitly:

- `microsoft_docs_search`
- `microsoft_docs_fetch`
- `microsoft_code_sample_search`

Expected verification flow:

1. Call `microsoft_docs_search` to find the relevant official Microsoft documentation.
2. When a relevant page is found and the exact specification or behavior must be confirmed, call `microsoft_docs_fetch` with the retrieved Microsoft Learn URL.
3. When the validity of the implementation depends on official Microsoft code samples or API usage, use `microsoft_code_sample_search`.
4. When the document body must be confirmed, do not rely on a search-result summary alone.

## Verifying GitHub documentation

For GitHub Actions, repository settings, permissions, rulesets, pull requests, GitHub Enterprise, GitHub Copilot, GitHub Advanced Security, and other GitHub products, call the remote GitHub MCP Server tool explicitly and treat `docs.github.com` as the authoritative source:

- `github_support_docs_search`

`github_support_docs_search` is a remote-only toolset. Reference: https://github.com/github/github-mcp-server#additional-toolsets-in-remote-github-mcp-server

Do not assume that every remote-only toolset is available just because a built-in GitHub MCP Server exists. If `github_support_docs_search` is not available in the GitHub.com Copilot Code Review execution environment, do not substitute the model's memory or general knowledge; treat that documentation check as `unverified`.

## Evidence requirements

For a documentation-grounded finding, all of the following are required:

- The relevant official documentation was actually retrieved during this review.
- The canonical official documentation URL that justifies the finding is written in the same review comment as that finding.
- The finding explains the specific official specification that contradicts, constrains, or affects the proposed change.
- Do not cite documentation only because it seems related.
- Do not claim an MCP server or MCP tool was used unless the invocation actually succeeded.
- Do not use the model's memory, blogs, community posts, third-party documentation, or search-result snippets as a substitute for available official documentation.

If an authoritative source could not be retrieved, do not report the issue as definitively non-compliant with official documentation.

## Findings

For every finding, provide:

- `Severity`: `blocking`, `high`, `medium`, or `low`
- File and line location
- Observed behavior and impact
- Official source URL for documentation-grounded findings, written in the same review comment
- Recommended fix or verification

Limit findings to actionable correctness, security, compatibility, configuration, and documentation-integrity issues. Do not turn style preferences into findings.

## Unverified checks

When an official documentation check is required but the required MCP tool is unavailable, the MCP tool invocation fails, authentication or connection fails, or no authoritative source is found, mark the check as `unverified`.

State:

- what could not be verified,
- why it could not be verified,
- which documentation or tool should be used next.

Never fabricate a citation or URL, and never fall back to the model's general knowledge to judge compliance or non-compliance with official documentation.

## Output shape

1. Findings ordered by severity.
2. A short summary of checked technologies and official sources.
3. An `Unverified` section for incomplete checks.
4. A note that comments are advisory unless a separate repository ruleset or required check makes them a merge gate.