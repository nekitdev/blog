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
The disk is ZFS on top of LUKS, unlocked with my YubiKey,
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

Everything you see here is written in Rust, from the server to the WebAssembly running
in your browser right now. Well, with two exceptions: a tiny bit of CSS for styling
and a few lines of JS --- I'll explain that later. ;)

The site is built with Dioxus in fullstack mode. The server renders each page to plain HTML,
so it's displayed right away, while your browser loads the WebAssembly in the background.
Once loaded, the WASM takes over the page that's already there, which is called _hydration_.

If your connection is slow enough, you can catch the spinner in the footer before
the page is hydrated. After hydration, the spinner turns into `?`,
which opens the keyboard shortcuts --- more on them later.
On a faster connection or once the WASM is cached, the spinner's gone before you notice;
you can bring it back by disabling the cache and throttling the connection.

The design is largely borrowed from my favorite place on every computer: the terminal!
Everything is monospace, mostly lowercase, and nothing moves unless you ask it to.

At the bottom of every page there's a status bar, `tmux`-style: the current server time,
the `?` I mentioned before, a little `<3`, my phone's battery level (please don't worry if it's red)
and, last but not least, the theme switcher.

Click `system` in the status bar to cycle through `system`, `dark` and `light`,
or just press `t` on your keyboard if you have one.
Clicking the question mark or pressing `?` toggles the list of keyboard shortcuts ---
the site is meant to be usable without touching the mouse, which gives it some `vim` vibes.

There's exactly one bit of JS I wrote by hand: the few lines that apply the theme
before the page renders, so dark-mode people, myself included, don't get flashbanged.
Everything else, CSS aside, is Rust, as promised! ^^

The [`/blog`][blog] page is basically `ls -l` output: dates on the left,
everything else on the right. And the [`/host`][host] page shows what the box from earlier
is up to right now, with `htop`-style bars for CPU, memory and swap. At the time of writing,
`pi` has been up for almost two weeks with a load average of something like `0.03 0.14 0.15`
while running a k3s server and several pods. It's a rather relaxed computer when it isn't,
you know, compiling the kernel. :p

## load average?

Pressing `^C` won't save you now: we're taking a math detour!

Those three numbers you've seen are the load average over the last 1, 5 and 15 minutes.
You probably already knew that, but do you know how they're calculated?

Every five seconds, the kernel counts how many tasks are running or waiting to run
(or stuck waiting on disk), and folds that count into each one of the averages: $$
L_t = lambda L_(t - 1) + (1 - lambda) n_t
$$

Here $t$ counts the 5-second updates (_ticks_), $n_t$ is the number of tasks at tick $t$,
and $L_(t - 1)$ is the previous value.

As for the weight $lambda$, it follows exponential decay: $$
lambda = e^(- Delta T slash tau)
$$
with $Delta T = 5$ seconds per tick and $tau = 60 m$ seconds, where $m in {1, 5, 15}$
is the window in minutes.

In numbers, here's how much of each new count contributes to the average, $alpha = 1 - lambda$:

| $m$  | $alpha$   |
| ---- | --------- |
| $1$  | $0.07996$ |
| $5$  | $0.01653$ |
| $15$ | $0.00554$ |

So the first number jumps around while the last one barely moves.

Honestly, this detour was partly an excuse to show off the server-side math rendering I added,
but it's over now, and `^C` works again! ;)

## live things

## this blog

## why `;`

## what's next?

$$
  ket(psi) = alpha ket(0) + beta ket(1)
$$

[host]: https://nekit.dev/host
[blog]: https://nekit.dev/blog
