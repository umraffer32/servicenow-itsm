# Linux client cannot reach hosts behind a Tailscale subnet router

> **ServiceNow article fields**
> - **Knowledge Base:** IT
> - **Category:** Network and Remote Access
> - **Article type:** Troubleshooting
> - **Short description:** Linux client cannot reach hosts behind a Tailscale subnet router
> - **Keywords:** Tailscale, subnet router, accept-routes, Linux, Ubuntu, routing, VPN, pfSense

---

## Issue

A Linux client connected to a Tailscale mesh Virtual Private Network (VPN) cannot reach devices on a subnet that a Tailscale subnet router advertises. The client shows as connected in `tailscale status`, but traffic to the remote subnet times out. Windows clients on the same network reach the subnet with no extra steps.

## Environment

- Tailscale mesh VPN
- A subnet router, for example pfSense, advertising a private subnet such as `10.200.50.0/24`
- Client devices running Linux (Ubuntu or Debian) and Windows
- A separate primary network, for example `192.168.4.0/22`, that needs to reach the advertised subnet

This setup is common when the primary router cannot add static routes. A Tailscale subnet router bridges the two networks instead, so a machine on the primary network can reach a service on the lab subnet without changing the main network.

## Cause

Tailscale does not accept advertised subnet routes by default on Linux. Windows, macOS, iOS, and Android accept advertised routes automatically, which is why those clients reach the subnet with no extra configuration. On Linux you have to opt in. This keeps Tailscale from changing the system routing table on its own, which matters on the headless servers that Linux clients often are. Until the Linux client is told to accept routes, it has no route to the advertised subnet and the traffic goes nowhere.

## Before you start

Confirm the subnet router side is correct first:

1. The subnet router advertises the subnet, for example `sudo tailscale up --advertise-routes=10.200.50.0/24`.
2. The advertised route is approved in the Tailscale admin console, under the subnet router device. Tailscale ignores an advertised route until an admin approves it.
3. Internet Protocol (IP) forwarding is enabled on the subnet router so it can pass traffic between networks. pfSense has this on by default because it is a router. A plain Linux subnet router needs it turned on with `net.ipv4.ip_forward=1`.

## Resolution

On the Linux client, enable route acceptance. The current Tailscale command changes only this one setting:

```bash
sudo tailscale set --accept-routes
```

The older form below also works and is what you will see in most guides, but `tailscale up` reapplies your full configuration, so include any other flags you normally run with or they reset to their defaults:

```bash
sudo tailscale up --accept-routes
```

The setting persists across reboots, so you run it once per client.

## Verify

1. `tailscale status` lists the subnet router and the routes it offers.
2. `ip route` shows the advertised subnet routed through the `tailscale0` interface.
3. Ping a host on the advertised subnet, or open a web service hosted there from a browser on your primary network.

## If traffic still does not pass, check the subnet router firewall

A subnet router such as pfSense denies traffic you have not explicitly allowed. Accepting the route gets the packets to the router, but the router still drops them when no rule permits the destination. Add a firewall rule on the subnet router that allows the source, the client or its Tailscale address, to reach the destination host or subnet, then test again.

## Still need help

- Tailscale subnet routing reference: https://tailscale.com/kb/1019/subnets
- If the route never appears in `tailscale status`, recheck that it is approved in the admin console and that IP forwarding is enabled on the subnet router.
