+++
title = "hello, world;"
description = "how the site is run: raspberry pi, phone battery, and too much rust"
date = 2026-10-31
tags = ["homelab", "rust", "nix"]
draft = true
preamble = """
#import "@personal/quantum:0.1.0": ket
"""
+++

> Every programmer's blog ought to start with a post about how the blog works.
> This is precisely that post! :)

## the box

Everything you're looking at is served by the black box sitting on my desk.
Open the case, and there's a Raspberry Pi 5 inside, which you can monitor in real time at
[`/host`][host].

The box runs NixOS, and getting the Pi 5 to boot it turned into a long-term love-hate relationship.
The disk is ZFS on top of LUKS, unlocked with my YubiKey over FIDO2,
or a passphrase in case anything happens to the key.
Which means someone walking off with the Pi gets a rather small, well-encrypted brick. ^^

The system itself could be cross-built,
but there's something special about the Pi compiling its own kernel... :p

On top of that sits a single-node Kubernetes cluster (k3s, to be precise).
Overkill? Absolutely. Am I going to stop? No.
The site, the battery endpoint and other services are all just pods managed by a Helm chart,
so deploying is one `helm upgrade` and nothing else.

## getting traffic in

The box sits behind my home router: I don't have a public IP,
and I wouldn't open ports even if I did. ^~^

So requests take a little detour:

```text
you -> cloudflare <= server -> box
```

Arrows show who initiates connections. The only special case here is the server (VPS)
connecting to Cloudflare to create the tunnel (via cloudflared).
The server routes traffic from Cloudflare to the box via the tailnet, my private Tailscale network.

- _Cloudflare_ handles TLS and caching, and hides where things actually are.
- _The server_ is a small VPS, the only machine connected to both the internet and the tailnet.
- _The box_, you've already met. ^^

"Why the VPS?" my dear reader may be wondering. Well, the Pi cannot connect directly to Cloudflare
due to the imposed restrictions: connections from Russia to Cloudflare are throttled,
and often dropped altogether.
Instead of routing the Pi through proxies, I opted for Tailscale and let the VPS talk to Cloudflare,
which turned out much more stable and faster overall.

So, in the end, the only way in is from the inside out. :D

## the site

## live things

## this blog

## why `;`

## quantum

$$
  i planck partial / (partial t) ket(psi) = hat(H) ket(psi)
$$

[host]: https://nekit.dev/host
