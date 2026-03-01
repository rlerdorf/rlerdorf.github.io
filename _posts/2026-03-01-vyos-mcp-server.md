---
date: '2026-03-01'
layout: post
title: Making VyOS Accessible with MCP
---

I have been running a [VyOS](https://vyos.io)-based router at home for years.
VyOS is incredibly powerful: VLANs, WireGuard tunnels, DNS
forwarding, firewall rules, DHCP, BGP if you are into that, and pretty much
anything else you can think of. But it has always been squarely aimed at
networking enthusiasts. The configuration interface is a CLI modeled after
Juniper's JunOS with a candidate config, commits, rollbacks. If you are used to
the web UI on a typical consumer router, VyOS is a different world entirely.

I have been thinking about how AI could change that, and specifically how the
[Model Context Protocol](https://modelcontextprotocol.io/) (MCP) could turn a
VyOS router into something that anyone can manage through natural language. Instead
of memorizing that the path to set a static route is:

```
set protocols static route 10.0.0.0/24 next-hop 192.168.1.1
```

you just say, "Route the 10.0.0.0/24 network through 192.168.1.1." Or better
yet, things like:

- "How many phones are connected to my wifi right now?"
- "Put all my Alexa devices on a separate VLAN"
- "Configure WireGuard so I can access my network remotely"
- "What is using all my bandwidth?"

Suddenly VyOS goes from a niche device for geeks like me to something that could
genuinely replace consumer routers for normal people. You get all the power of a
proper routing platform without that scary and intimidating CLI. The AI handles the
translation.

## Learning MCP by Building

I learn best by building things, and I wanted to understand the nuts and bolts
of MCP: the protocol itself, how streaming HTTP transport works, the JSON-RPC
framing, session management. Reading specs is one thing, implementing them is
another. So I wrote two MCP servers for my VyOS router, one in PHP and one in
Go, each taking a fundamentally different approach. The Go version I wrote
almost entirely with [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview),
which was a good test of how well AI-assisted development works when you
understand the problem but not the language idioms. I could describe what the
VyOS CLI tools expect and let Claude handle the Go specifics.

## The PHP Version: Stdio + REST API

The [PHP version](https://github.com/rlerdorf/vyos-mcp-php) was my first pass.
It uses the [PHP MCP SDK](https://github.com/AntCAS/php-sdk) with stdio
transport, meaning Claude Code or any MCP client spawns it as a subprocess on
my workstation. It talks to VyOS via the router's built-in REST API over HTTPS:

```
MCP Client → spawns → server.php (stdio) → HTTPS → VyOS REST API
```

The nice thing about this approach is simplicity. Nothing runs on the router
beyond what VyOS already provides. You configure VyOS to expose its API, set an
API key, and the PHP server wraps each API endpoint as an MCP tool. The entry
point is about as minimal as it gets:

```php
$server = Server::builder()
    ->setServerInfo('VyOS Router', '1.0.0')
    ->addRequestHandler(new NegotiatingInitializeHandler($serverInfo))
    ->setContainer($container)
    ->setDiscovery(__DIR__, ['src'])
    ->build();

$result = $server->run(new StdioTransport());
```

The tools themselves use PHP attributes for schema definitions which the SDK
picks up via auto-discovery:

```php
#[McpTool(name: 'vyos_show_config', description: 'Retrieve VyOS configuration at a path')]
public function showConfig(
    #[Schema(type: 'array', items: ['type' => 'string'], description: 'Configuration path components')]
    array $path = [],
    #[Schema(type: 'string', enum: ['json', 'raw'], description: 'Output format')]
    string $format = 'json',
): mixed {
    return $this->wrap(fn() => $this->client->showConfig($path, $format));
}
```

One thing I had to work around was MCP protocol version negotiation. The PHP
SDK's built-in `InitializeHandler` ignored the client's requested protocol
version entirely, so I wrote a `NegotiatingInitializeHandler` that actually
checks what the client asks for and responds accordingly.

The downside of the REST API approach: you need to configure HTTPS and an API
key on the router, and the responses can be chatty.

## The Go Version: On-Router with Streamable HTTP

The [Go version](https://github.com/rlerdorf/vyos-mcp-go) takes a different
approach. Instead of calling the REST API over the network, it runs directly on
the router as a systemd service and executes VyOS CLI tools like
`/bin/cli-shell-api`, `/opt/vyatta/sbin/my_set`, and
`/opt/vyatta/sbin/my_commit` directly. No API layer, no HTTPS configuration, no
API key.

```
MCP Client → SSH tunnel → vyos-mcp-go (on router) → VyOS CLI tools
```

The entire server compiles to a single 8MB static binary with zero runtime
dependencies. Cross-compile it on your workstation, scp it to the router, done:

```makefile
build:
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 $(GO) build -ldflags="-s -w" -o $(BINARY) .

deploy: build
	scp $(BINARY) mcp-server.service $(ROUTER):/config/user-data/
```

It lives in `/config/user-data/` which persists across VyOS image upgrades, so
you don't lose it when you update your router.

This version uses the [Go MCP SDK](https://github.com/modelcontextprotocol/go-sdk)'s
streamable HTTP transport rather than stdio. The server listens on
`127.0.0.1` (localhost only, no exposure to the network). You reach it
through an SSH tunnel:

```bash
ssh -L 8384:localhost:8384 router
```

Then configure your MCP client. For Claude Code it is:

```json
{
  "mcpServers": {
    "vyos": {
      "type": "http",
      "url": "http://localhost:8384/mcp"
    }
  }
}
```

The main.go is pretty compact. Set up the VyOS config session, register the
tools, serve streamable HTTP on `/mcp`:

```go
client, err := NewVyosClient()
if err != nil {
    log.Fatal(err)
}
defer client.Close()

mcpServer := mcp.NewServer(&mcp.Implementation{
    Name:    "VyOS Router",
    Version: "2.0.0",
}, nil)
registerTools(mcpServer, client)

handler := mcp.NewStreamableHTTPHandler(
    func(*http.Request) *mcp.Server { return mcpServer },
    nil,
)
```

The interesting part is the VyOS client. VyOS has a concept of config sessions
where you set up a session, make changes in a candidate config, then commit
atomically. The Go client initializes a session at startup by calling
`cli-shell-api getSessionEnv` and injecting the resulting environment variables
into all subsequent command executions. Config-modifying operations are
serialized with a mutex so concurrent MCP calls don't stomp on each other.

## The Tools

Both versions expose the same set of tools. The configuration tools follow the
VyOS workflow (show, set, delete, commit, save):

| Tool | What it does |
|------|-------------|
| `vyos_show_config` | Retrieve config at a path (JSON or raw) |
| `vyos_set_config` | Set a configuration value |
| `vyos_batch_config` | Set/delete multiple values atomically |
| `vyos_delete_config` | Delete a config node |
| `vyos_config_exists` | Check if a config path exists |
| `vyos_return_values` | Get values at a path |
| `vyos_commit` | Commit pending changes (with optional auto-rollback timer) |
| `vyos_confirm` | Confirm a pending commit to cancel auto-rollback |
| `vyos_save_config` | Save running config to startup |

Then there are operational and diagnostic tools:

| Tool | What it does |
|------|-------------|
| `vyos_show` | Run any operational show command |
| `vyos_reset` | Run a reset command |
| `vyos_generate` | Run a generate command |
| `vyos_system_info` | System version and build info |
| `vyos_interface_stats` | Interface statistics |
| `vyos_routing_table` | IP routing table |
| `vyos_dhcp_leases` | DHCP server leases |
| `vyos_health_check` | Combined health check (CPU, memory, disk, uptime) |
| `vyos_ping` | Ping a host from the router |
| `vyos_traceroute` | Traceroute to a host |

The `vyos_commit` tool has a `confirmTimeout` parameter that tells VyOS to
auto-rollback after N minutes if you don't call `vyos_confirm`. Handy safety net
when making firewall changes that might lock you out.

## Security

MCP itself has no authentication mechanism, which is why the Go server only
binds to localhost. The SSH tunnel provides authentication (SSH keys), encryption,
and access control. The systemd unit also runs with `NoNewPrivileges=yes` and
`PrivateTmp=yes`.

For the PHP version, security comes from the VyOS API key and HTTPS. Since
the MCP client spawns it as a local subprocess, the API key stays in environment
variables and never hits the wire in plaintext.

## What I Learned About MCP

Building both servers gave me a good feel for the protocol. A few observations:

**Streamable HTTP is the way to go.** The older SSE transport required two
connections (one for server-to-client events, one for client-to-server requests).
Streamable HTTP consolidates everything onto a single endpoint. The Go SDK makes
this trivial: call `NewStreamableHTTPHandler` and you are done.

**Stdio is simpler for development** but awkward for always-on services. Having
the MCP client spawn and manage the process lifecycle works fine, but you lose the
ability to keep a persistent daemon running. For a router that should always be
available, the HTTP transport makes more sense.

**The protocol is essentially JSON-RPC 2.0 with conventions.** Session
management via the `Mcp-Session-Id` header, a well-defined initialization
handshake, and a tool schema format based on JSON Schema. Nothing revolutionary,
but a solid, practical design.

**Tool design matters.** The difference between an AI being able to successfully
configure your router and it fumbling around is entirely in how you define the
tool schemas and descriptions. Clear parameter descriptions, sensible defaults,
and structured output go a long way.

## The Bigger Picture

This is what excites me about MCP beyond just my own router tinkering. VyOS is
an enterprise-grade routing platform. It can do things that no consumer router
can: traffic shaping, policy-based routing, multi-WAN failover, full BGP
support. But the barrier has always been the learning curve.

With a good MCP server in front of it, that barrier largely disappears. Someone
who has never touched a CLI can say, "I want my kid's devices to have no
internet access after 9pm" and the AI can translate that into the correct
firewall rules, time-based policies, and device groups. It can show you what it
is about to do, explain why, and commit it with an auto-rollback timer in case
something goes wrong.

Consumer routers are simple because they have to be. Their interfaces can only
expose a fraction of what the underlying hardware can do. When the interface is
natural language backed by a capable AI, the complexity of the underlying system
stops being a limitation and starts being an advantage.

Both projects are on GitHub:
[vyos-mcp-go](https://github.com/rlerdorf/vyos-mcp-go) and
[vyos-mcp-php](https://github.com/rlerdorf/vyos-mcp-php).
