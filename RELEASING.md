# Releasing

This repository holds no executable code, so a release is only a version bump. The server at `https://alira-ai.ai/mcp` is deployed separately.

## Steps

1. Bump `version` in `.claude-plugin/plugin.json`, `plugin.json`, `gemini-extension.json`, and `server.json` so all four agree. Below, `X.Y.Z` is that version. Check them with:

   ```
   grep -n '"version"' .claude-plugin/plugin.json plugin.json gemini-extension.json server.json
   ```

   Leave `version` out of `.claude-plugin/marketplace.json`. `plugin.json` takes precedence there, and `claude plugin validate` warns if both are set.

2. Validate:

   ```
   claude plugin validate --strict .
   ```

3. Merge to `main`. Anthropic's directory picks up new versions from the branch it follows, so there is nothing to resubmit.

4. Publish the release, which triggers `.github/workflows/publish-mcp-registry.yml`:

   ```
   gh release create vX.Y.Z --title vX.Y.Z --notes "..."
   ```

   The first publish runs the workflow by hand instead: `gh workflow run publish-mcp-registry.yml`.

5. Confirm the listing:

   ```
   curl -s "https://registry.modelcontextprotocol.io/v0/servers?search=io.github.SuresoftTechnologies/alira-ai-agent"
   ```

## Things that are hard to change

- `name` in `plugin.json`: a renamed plugin is a different plugin to everyone who installed it.
- The server name in `server.json`: each remote URL can belong to only one server name in the registry.
- The directory listing's URL slug is permanent once published.

## Repository settings

GitHub topics feed channels that collect by topic. The call below replaces the whole list, so include every topic each time:

```
gh api -X PUT repos/SuresoftTechnologies/alira-ai-agent/topics \
  -f 'names[]=claude-code-plugin' -f 'names[]=claude-code-plugins' -f 'names[]=mcp' \
  -f 'names[]=mcp-server' -f 'names[]=gemini-cli-extension' -f 'names[]=sdd' \
  -f 'names[]=software-design-document' -f 'names[]=documentation' -f 'names[]=c'
```
