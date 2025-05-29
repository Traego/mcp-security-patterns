# MCP Security Patterns

A collection of security patterns to strengthen Model Context Protocol (MCP) implementations.

## Table of Contents

1. [First Touch Scoping](#first-touch-scoping)

---

## Pattern: First Touch Scoping

**Intent**  
Prevent MCP calls from touching unintended resources by automatically scoping the session to the first resource interacted with.

**Context**  
MCP Servers often handle requests across multiple resources (e.g., GitHub repositories). Without proper scoping, a malicious or buggy request can access or modify data in resources the user did not intend to share.

**Problem**  
In the absence of a scoping mechanism, once a session is open, malicious actors may trick clients into targeting different resources mid-session, leading to unauthorized cross-resource access or data leakage. This vulnerability was demonstrated in the GitHub MCP Server as described by Invariant Labs, where subsequent calls could switch context to repositories not originally requested ([source](https://invariantlabs.ai/blog/mcp-github-vulnerability)).

**Solution**  
On the first resource access request within an MCP session, record the resource identifier and automatically enforce that all subsequent requests operate solely on that resource. Any request targeting a different resource must be explicitly scoped or will be rejected.

**Example**  
1. **Client**: "Please look at the issues in my repo **XYZ** and summarize the issues."
   - MCP Server records `repo = "XYZ"` and scopes the session accordingly.
2. **Client**: "get_code **ABC**"
   - MCP Server rejects the request with an out-of-scope error, since the session is scoped to `XYZ`.

**Implementation Details**  
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
