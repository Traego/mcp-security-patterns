# MCP Security Patterns

A collection of security patterns to strengthen Model Context Protocol (MCP) implementations.

## Table of Contents

1. [First Touch Scoping](#pattern-first-touch-scoping)

---

## Pattern: First Touch Scoping

**Intent**  
Prevent cross resource prompt poisoning attacks by automatically scoping down the session first resource (or resource group) interacted with.

**Context**  
The Github MCP server was found [to be vulnerable](https://invariantlabs.ai/blog/mcp-github-vulnerability) to a prompt poisoning, cross resource access vulnerability, but this is not a unique issue to Github. Any MCP server where the user access is provisioned for multiple resources, but they may not want client or MCP access across these resources. Other examples would be customer service agents which have acceess to multiple customer data leaking customer data to malicious actors unintentionally. Without proper scoping, a malicious or buggy request can access or modify data in resources the user did not intend to share.

**Problem**  
In the absence of a scoping mechanism, once a session is open, malicious actors may trick clients into targeting different resources mid-session, leading to unauthorized cross-resource access or data leakage. This vulnerability was demonstrated in the GitHub MCP Server as described by Invariant Labs, where subsequent calls could switch context to repositories not originally requested.

**Solution**  
On the first resource access request within an MCP session, record the resource identifier and automatically enforce that all subsequent requests operate solely on that resource. Any request targeting a different resource must be explicitly scoped or will be rejected. Expanding the scope requires either a new session or an explicit out of band request.

**Example**  
1. **Client**: "Please look at the issues in my repo **XYZ** and summarize the issues."
   - MCP Server records `repo = "XYZ"` and scopes the session accordingly. Change notifications are sent scoping down resources and tools to just those relevant to repo **XYZ**.
2. **Client**: "get_code **ABC**"
   - MCP Server rejects the request with an out-of-scope error, since the session is scoped to `XYZ`.

**Sample Implementation Details**  
- **Session State**: Store `first_touch_resource` in the session context on the first qualifying request.
- **Request Interceptor**: Before processing each call, verify that any resource identifier matches `first_touch_resource`. If not, return a scoped violation error.
- **Re-scoping**: Provide an explicit API or command (e.g., `reset_scope` or new session initiation) for clients to scope to a different resource when needed.

**Benefits**  
- Strongly enforces least-privilege by default.  
- Mitigates class of cross-resource data leakage vulnerabilities.  
- Transparent to clients after initial scoping.

**Drawbacks**  
- Requires users to explicitly reset or start new sessions to interact with different resources.  
- May disrupt workflows that legitimately span multiple resources without session resets.

**References**  
- Invariant Labs. "MCP GitHub Vulnerability: Cross-Resource Context Hijack". Invariant Labs Blog. Retrieved from https://invariantlabs.ai/blog/mcp-github-vulnerability
