<!-- The spec text here mirrors index.html. Change both together. -->

<p align="center">
  <img src="logo.svg" alt="PaceVer" width="350" />
</p>

# Pace Versioning (PaceVer)

> Version your app by the **pace** each release travels at.

`MARKETING.NATIVE.OTA`

PaceVer is a versioning scheme for user-facing apps that reach people through
**two channels at two speeds**: a slow, gated **native** build that goes through an
app store, and a fast, independent **over-the-air (OTA)** update that lands on an
already-installed build. The version number tells you which channel a release came
down: its _pace_, not its size.

It is built for the world of React Native, Expo, and any framework where some
releases require a new binary and others can be pushed straight to devices.

---

## Summary

A version number takes the form `MARKETING.NATIVE.OTA`:

```
  1   .   4   .   12
  │       │       │
  │       │       └─ OTA        bumped on every over-the-air update   (fast)
  │       └───────── NATIVE     bumped on every new store binary      (slow)
  └───────────────── MARKETING  yours to define: era, year, redesign, or never
```

- **MARKETING**: the free number. Bump it for a redesign, a new year, a rebrand, a
  rewrite, a milestone, or never. It carries no compatibility promise. If you care
  about signalling the _magnitude_ of a change, this is the only place it lives.
  `0` is reserved for the pre-release era; ship your first stable release as `1.0.0`
  and go from there.
- **NATIVE**: bump it when a release needs a **new binary installed from a store**
  (anything OTA can't deliver: native code, new dependencies, SDK/runtime upgrades,
  permission changes). Slow: gated behind store review. Resets OTA to 0.
- **OTA**: bump it for every release pushed **over the air** to an existing build
  (JavaScript, styling, content, assets within the current runtime). Fast: lands in
  minutes, no store review.

A new native build always resets OTA to `0`. By default a MARKETING bump resets NATIVE
and OTA too, so a redesign goes to `2.0.0`; a team may instead let the numbers keep
climbing to `2.6.2` (see the FAQ). Pick one convention and stay consistent.

---

## Introduction

Mobile apps don't ship the way libraries do. A library has one release channel and
one question that matters: _did the contract break?_ That's what Semantic Versioning
answers, and it answers it well.

An app shipped with React Native, Expo, or a similar framework has **two** release
channels moving at **two speeds**. Some changes (a new native module, an SDK bump, a
new permission) can only reach users as a fresh binary through a distribution channel
(App Store, Play Store, TestFlight, enterprise/MDM): slow, gated behind redistribution,
impossible to roll back instantly. Other changes
(a copy fix, a restyled screen, a JS-only feature) can be pushed over the air and
land on devices in minutes, independently, with no store review.

SemVer can't see this. Its `MAJOR.MINOR.PATCH` describes _what changed_; it says
nothing about _how the change reaches the user_, which on mobile is the fact that
governs review latency, rollout speed, and rollback. PaceVer makes that fact the
version.

PaceVer deliberately says nothing about the size of a change. A one-word fix and a
screen rewrite both bump OTA by one, because they ship the same way, at the same
pace. If your team wants to signal that a release is a big deal, that signal goes in
MARKETING, which is yours to spend however you like.

---

## PaceVer Specification

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT", and "MAY" are to be
interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

1. A project using PaceVer MUST distinguish two kinds of releases: **native
   releases**, which require a new binary to be installed through a distribution
   channel (App Store, Play Store, TestFlight, enterprise/MDM, or equivalent), and
   **OTA releases**, which are delivered to an already-installed binary without
   redistribution.
1. A normal version number MUST take the form `X.Y.Z`, where `X` (MARKETING), `Y`
   (NATIVE), and `Z` (OTA) are non-negative integers and MUST NOT contain leading
   zeroes. Each successive version MUST increase in precedence, as defined in rule
   10. Individual elements need not increase one by one: incrementing an element resets
   the elements to its right (rule 6), so the version as a whole always orders forward
   even though a reset element drops to `0`.
1. **MARKETING (`X`)** is reserved at `0` to mean the app is pre-release, not yet
   considered stable or shipped to the public. Once MARKETING reaches `1` or higher,
   the _reason_ and _magnitude_ of an increment carry no required meaning and no
   compatibility promise: a team MAY increment it for any reason (a new year, a
   redesign, a rebrand, a rewrite, a milestone, the magnitude of a change) or never
   increment it again. Its one fixed constraint is direction: MARKETING MUST be
   monotonically non-decreasing over the life of the project.
1. **NATIVE (`Y`)** MUST be incremented when a release requires a new binary to be
   installed, that is, any change that cannot be delivered over the air. This
   includes, but is not limited to: native code or native dependency changes,
   runtime or SDK upgrades, permission or entitlement changes, and changes to
   compiled application configuration.
1. **OTA (`Z`)** MUST be incremented for each release delivered over the air to an
   existing native build, that is, any change deliverable without a new binary, such
   as JavaScript, styling, content, and assets running within the current native
   runtime.
1. When NATIVE is incremented, OTA MUST be reset to `0`, because a new native build
   begins a fresh OTA lineage. When MARKETING is incremented, NATIVE and OTA SHOULD
   be reset to `0`; this is the recommended default. A project MAY instead let NATIVE
   and OTA keep climbing across a MARKETING bump (see the FAQ), but it MUST pick one
   convention and apply it consistently.
1. A freshly installed native build, before any OTA release has been applied, is at
   OTA `0`. The JavaScript and assets bundled inside a native build are its
   **baseline**, not an OTA release, and MUST NOT be counted as OTA `1`.
1. The version a build reports SHOULD be composed at runtime: MARKETING and NATIVE
   are fixed by the installed binary; OTA reflects the over-the-air release currently
   applied, or `0` if none has been applied.
1. An OTA release MUST be compatible with the native build it targets, and MUST NOT
   be delivered to a native build it was not produced for. (In Expo/EAS terms, OTA
   releases are scoped to a matching runtime version.)
1. **Precedence** is determined by comparing MARKETING, then NATIVE, then OTA,
   numerically (`1.2.0 < 1.2.7 < 1.3.0 < 2.0.0`).
1. **One version line, all platforms in lockstep.** MARKETING, NATIVE, and OTA are
   single project-wide numbers, **not** tracked per platform. When NATIVE is
   incremented, a new binary MUST be released to every native distribution channel
   the app ships through (App Store, Play Store, and so on), even for a platform
   with no functional change, so that all platforms share the same NATIVE build.
   This keeps every platform on a common runtime, so a single OTA release applies to
   all of them under one OTA number. Platforms therefore never diverge: there is
   always exactly one current version for the whole app.
1. **Scope.** PaceVer is for user-facing application clients that ship through a
   native channel and an OTA channel. It is **NOT** for libraries, packages, or
   APIs that expose a programmatic contract to other software. Version those with
   SemVer.
1. Identifiers MAY be appended after OTA: a hyphen introduces pre-release identifiers
   (e.g. `1.4.0-rc.1`) and a plus sign introduces build metadata (e.g. `1.4.0+ios`).
   PaceVer assigns these suffixes no channel or pace meaning; their interpretation is
   left to the team. Build metadata MUST be ignored when determining precedence.
1. **Initial version and going stable.** A project SHOULD begin at `0.1.0`, where
   MARKETING `0` marks the pre-release era. NATIVE and OTA increment as usual
   through the `0.x` era. When the team considers the app stable and ready for the
   public, it increments MARKETING to `1`. With the recommended reset (rule 6) the
   first stable release is `1.0.0`; a team that does not reset carries its running
   numbers instead, so `0.5.3` becomes `1.5.3`. The exact moment of going stable is
   the team's call.

---

## Why use PaceVer

- **The number tells you the pace.** `2.5.0 → 2.5.3` shipped over the air and is
  already on devices. `2.5.3 → 2.6.0` is a new binary waiting on store review. You
  know the deployment story at a glance.
- **It fits store rules for free.** The binary carries `MARKETING.NATIVE.0` as its
  marketing version (the three integers the stores expect, with OTA pinned at `0` as
  the baseline); the live OTA number is composed on top at runtime.
- **It stays SemVer-compatible in form.** A PaceVer version is still an `X.Y.Z` string
  ordered by the same left-to-right precedence as SemVer, so tooling that sorts or
  compares SemVer-shaped versions keeps working. PaceVer changes what the three
  positions _mean_, not the format or the ordering.
- **It ends the MAJOR argument.** No more debating whether a redesign "deserves" a
  major bump. MARKETING is a free number you point at whatever your team cares about.
- **It matches the tooling you already use.** NATIVE lines up with your binary /
  runtime version; OTA lines up with your update channel (EAS Update, CodePush).

---

## When NOT to use PaceVer

PaceVer is for user-facing application clients that ship through a native channel and
an OTA channel (rule 12). It is the wrong tool when:

- **You are versioning a library, package, or API.** Anything that exposes a
  programmatic contract to other software should use SemVer, whose whole job is to
  signal compatibility. PaceVer says nothing about compatibility.
- **Your app has effectively one delivery channel.** A web app or PWA where every
  deploy goes out the same way has no native/OTA split for PaceVer to capture. Version
  it on its own terms.
- **Your audience keys off recency or magnitude.** If readers of your version number
  primarily want "how recent" (CalVer) or "how big" (SemVer MAJOR) rather than "which
  channel," PaceVer puts the signal they want in the wrong place.

Adopting PaceVer means giving up SemVer's compatibility contract and CalVer's
at-a-glance recency in exchange for an at-a-glance _pace_ signal. Make that trade on
purpose.

---

## Example: one app's version history

| Version | Channel | What shipped                                                                                                   |
| ------- | ------- | -------------------------------------------------------------------------------------------------------------- |
| `0.1.0` | NATIVE  | First build. The pre-release era begins.                                                                       |
| `0.1.1` | OTA     | Beta copy tweak, pushed over the air.                                                                          |
| `0.2.0` | NATIVE  | Added a camera module, new binary to testers, OTA resets.                                                      |
| `1.0.0` | NATIVE  | First public release on the App Store and Play Store. MARKETING → 1, everything resets.                        |
| `1.0.1` | OTA     | Typo fix, on devices in minutes.                                                                               |
| `1.0.2` | OTA     | JavaScript crash fix.                                                                                          |
| `1.1.0` | NATIVE  | Expo SDK upgrade, new binary to both stores, OTA resets.                                                       |
| `1.1.1` | OTA     | Restyled a screen.                                                                                             |
| `2.0.0` | NATIVE  | A full seasonal redesign the team wants to flag. MARKETING bump, new binary to both stores, everything resets. |

Every row tells you how it shipped from the number alone: a middle-digit change means
a trip through the store; a last-digit change means it was already on devices in
minutes.

---

## FAQ

**Where does the OTA number physically live?**
Not in the binary's store version. An OTA update can't rewrite that, and that's fine.
The app composes its version at runtime: it reads MARKETING and NATIVE from the binary
and OTA from the applied update's manifest (e.g. `Updates.manifest` / `extra` in
Expo). A fresh install with no update applied reports OTA `0`.

**Does a higher OTA number mean a bigger change?**
No. OTA counts over-the-air _releases_, not their size. A typo fix and a new screen
both bump OTA by one. If you care about magnitude, spend the MARKETING number on it.

**Does bumping MARKETING reset NATIVE and OTA to zero?**
By default, yes. A MARKETING bump goes to `X.0.0`, exactly as a NATIVE bump resets
OTA, so a redesign from `1.6.2` lands on `2.0.0`. This is the recommended convention,
but resetting on a MARKETING bump is not mandatory; see the next question.

**Can I skip the reset and let the numbers keep climbing?**
Yes. Instead of resetting, you may let MARKETING float as a pure label while NATIVE
and OTA keep climbing, so `1.6.2` becomes `2.6.2` rather than `2.0.0`. The tradeoff:
you give up the clean `X.0.0` era boundary, but you keep a single monotonic native and
OTA count across the whole life of the app, which some teams find easier to reason
about. Precedence still works either way, since versions compare left to right.
Whichever you pick, apply it consistently. (A new NATIVE build always resets OTA
regardless; that part is not optional, because a new binary starts a fresh OTA
lineage.)

**Can I bump MARKETING or NATIVE with an over-the-air update?**
No. MARKETING and NATIVE live in the installed binary (rule 8), so changing either
one means shipping a new native build. Only OTA moves without a new binary, which is
why a MARKETING or NATIVE change always lands with OTA back at `0`.

**Can I roll back a native release?**
No, and that's by design: like the app stores themselves, PaceVer has no concept of a
native rollback. NATIVE only ever moves forward (rule 2's precedence), and an OTA
release is scoped to the native build it was produced for (rule 9), so you can't drop
users back onto a previous binary's OTA lineage. To recover from a bad native build you
re-release the previous good build's content forward under a new NATIVE number, for
example shipping what was NATIVE 5 again as NATIVE 7, and go from there. OTA, by
contrast, can be rolled back within the same native build, as fast as it ships.

**A native change only affects iOS. Do I have to release Android too?**
Yes, and on purpose. A NATIVE bump ships a new binary to _every_ platform, even one
with no functional change, so all platforms stay on the same NATIVE build. This is
for OTA's sake: OTA releases are scoped to a runtime, so keeping every platform on a
common runtime lets one OTA release land everywhere under a single OTA number. The
seemingly redundant Android build usually isn't empty anyway: bumping NATIVE bumps the
runtime, so it's a real new binary, just without feature changes. The cost is one
extra store submission; the payoff is one coherent version line for the whole app.

**My app is also on the web. There's no native build there.**
PaceVer doesn't cover web. Web (and PWAs) has effectively one channel (every deploy
is over the air), so the native/OTA split doesn't apply. Version web on its own terms.

**Can I use PaceVer for my library or backend API?**
No. Use SemVer. PaceVer says nothing about API compatibility, which is the whole
point of versioning a library. PaceVer is for shipping apps, not contracts.

**Why isn't PaceVer versioned with PaceVer?**
Because PaceVer is for apps, and this is a document. A spec has no native build and no
over-the-air channel. It has a contract that other people rely on, which is exactly
what the Scope rule says to version with SemVer. So PaceVer eats its own rule and
ships as SemVer. SemVer versions itself; PaceVer versions itself with SemVer. Each
tool to its job.

**What about the very first release?**
Start at `0.1.0`. MARKETING `0` is the pre-release era, and you build through `0.x`
until the app is ready. When it's ready for the public, bump MARKETING to `1`. With
the recommended reset that lands you on `1.0.0`; if you don't reset, you carry your
running numbers to `1.y.z`. The recommended path is simply `0.1.0` → `1.0.0` → onward.
The exact moment doesn't really matter; it's a flag you raise when it feels true.

**I already ship under SemVer. How do I migrate?**
Easily, and without resetting. Keep your current version exactly as it is and start
reading its three positions as `MARKETING.NATIVE.OTA` from your next release. An app at
`3.4.1` just carries on: bump NATIVE for its next store binary, bump OTA for its next
over-the-air update. There's no renumbering and nothing to start over, because the
`0.1.0` start and the reset rules are only for brand-new projects. (You wouldn't want to
reset anyway: the stores reject a marketing version that decreases, and MARKETING is
monotonically non-decreasing by rule 3.)

---

## About

PaceVer was created in 2026 by [Pavlos Vinieratos](https://github.com/pvinis). The spec is
published at [pacever.org](https://pacever.org).

This specification is itself versioned with **SemVer**, not PaceVer. A spec is a
document with a contract people rely on, not an app shipping through native and OTA
channels, so by PaceVer's own Scope rule (rule 12) it uses the right tool for the
job.

Current version: **1.0.0** (SemVer).

## License

PaceVer, the Pace Versioning specification, is licensed under a
[Creative Commons Attribution 4.0 International License (CC BY 4.0)][cc-by]. You are free to
share and adapt it, including commercially, as long as you give appropriate credit.

Copyright © 2026 [Pavlos Vinieratos](https://github.com/pvinis).

[cc-by]: https://creativecommons.org/licenses/by/4.0/
