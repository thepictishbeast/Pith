# Pith

**A fork of [CORE](https://github.com/coreemu/core) that draws the
machine you actually have.**

*Pith* is the core of a stem, and the essential part of a thing. This is
CORE with its inside taken out and put to a different use.

## What changed, and why a fork at all

CORE is an emulator for imaginary networks: you draw nodes, it builds
them out of Linux network namespaces, and real programs run inside them.
Every node on its canvas is something somebody invented.

Pith is the canvas for [Throughline](https://github.com/thepictishbeast/Throughline),
which reads a **real** machine — every connection, attributed to the
process that opened it — and can change where that machine's traffic
goes. So the nodes here stand for programs that are running right now,
the links are claims about where their traffic leaves, and the topology
can be applied back to the host it was read from.

Nothing else does that. CORE builds labs; Throughline reads and
configures one real machine. Pith is where the two meet.

### Why fork instead of using CORE's GUI as a library

Because it is not one. `CanvasNode.__init__` takes a
`core.api.grpc.wrappers.Node` — the thing on the screen *is* a gRPC
object — and the right-click dialogs are EMANE, WLAN and mobility
config. Started with no daemon it renders an empty grid, a menu bar
containing only *Help*, and a `Setup Error`. There is no "borrow the
canvas, leave the rest".

### The state of the upstream, stated plainly

CORE's last commit on its default branch is **2025-05-19**, with zero
commits in the 90 days before this fork was taken (measured against the
GitHub API, 2026-09-12). Forking a dormant project means owning it. That
is a cost, not a detail, and it is accepted with open eyes: among
permissively-licensed network emulators, CORE is the only one whose
execution model — Linux network namespaces — is the one this work needs.

Upstream stays on the `upstream` remote. Fixes that belong to everyone
should go back as pull requests, not sit here.

## Changes so far

### Link labels off by default (`daemon/core/gui/graph/manager.py`)

CORE labels both ends of every link with the addresses it invented for
the lab. When a node stands for a real program those addresses are
fiction, and they bury the one label that is true.

**This takes two edits, not one.** The constructor sets the defaults,
and the reset that runs when a session is joined sets them **back to
`True`**. Changing only the constructor applies cleanly, reports
nothing, and does nothing.

### `throughline/bridge.py`

Turns `tl-observe` output — a real machine's connections as JSON — into
a CORE session: one node per program talking off the box, each linked to
the machine its traffic leaves through, with per-kind icons.

It stores Throughline's evidence (pid, executable, peers, ports, and the
visibility assessment) in `Session.metadata`, which is a persisted
`map<string, string>`, so no change to CORE's protobuf is needed. The
way to get it there is `start_session(session, definition=True)` —
`definition=True` stores nodes, links and metadata while instantiating
**nothing**. That flag is the entire difference between a drawing and an
emulation.

It refuses to draw at all if the reading cannot be trusted. An empty
list of connections from a machine that is withholding its socket tables
renders identically to an idle machine, and that is the one failure this
whole project exists to refuse.

## Building it

`scripts/core-lab/` in the Throughline repository builds this and runs
it in a container with **no network of its own** and two capabilities,
never `--privileged`. Do not install CORE's daemon on a host you care
about: its own install documentation answers "docker installed?" with
`iptables --policy FORWARD ACCEPT`.

Six undocumented breakages have to be worked around to build upstream at
all — a GUI entry point that is not where it looks, gRPC stubs that are
generated rather than shipped, a `protobuf` version the current tooling
outruns, a `configure` that hard-codes `./venv/bin/python`, a missing
`pyproj`, and a config file only the packaged install puts in place. All
six are tabulated in that directory's README. **Every one of them is a
candidate pull request back to CORE**, since they cost any newcomer the
same afternoon.
