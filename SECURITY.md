# Security Policy

## Reporting a vulnerability

Please do not open a public issue for security problems.

- Preferred: open a [private security advisory](https://github.com/popnow-ai/popnow-api-mcp/security/advisories/new) on GitHub.
- Or email **support@popnow.ai** with "Security" in the subject.

Include the affected component (the hosted server at `https://api.popnow.ai/api/v1/mcp` or the `@popnow/mcp`
package), steps to reproduce and the impact you observed. We will acknowledge your report and keep you updated
while we fix it.

## Leaked API keys

If you exposed a `sk-popnow-` key, revoke it right away in the [API console](https://popnow.ai/api/console/keys)
and create a new one. Giving each agent its own key with a low daily cap limits what a leaked key can spend.
