# TLAPM SDK for Workshop

A proof-checking environment for TLA+ specifications. It provides the TLA+
Proof Manager (`tlapm`), with the Zenon and Isabelle/TLA+ proof backends
packaged alongside and the Z3 SMT so the default proof workflow (SMT, then
Zenon, then Isabelle) works with no manual setup.

---

## Reference workshop

A minimal workshop:

```yaml
# workshop.yaml
name: proof-demo
base: ubuntu@24.04
sdks:
  - name: tlapm
    channel: latest/edge

actions:
  prove: tlapm --cleanfp --strict Proof.tla
```

This demonstrates checking a TLA+ proof with the SDK's default backend
workflow.

---

## Using the SDK

### Prerequisites, project layout

1. No prerequisite SDKs are required.
2. Put your TLA+ modules (`.tla` files) in your project directory. To try the
   SDK without an existing spec, create `Proof.tla`:

   ```tla
   ---- MODULE Proof ----
   EXTENDS TLAPS, Naturals
   THEOREM t == \A x \in Nat : x + 0 = x
     OBVIOUS
   ====
   ```

3. On launch and on every workshop refresh, the SDK installs `z3` from the
   Ubuntu archive and puts its own `bin/` directory on `PATH` system-wide.

### Check proofs

Once the workshop is ready:

```bash
workshop shell
tlapm --cleanfp --strict Proof.tla
```

A successful run ends with `All N obligations proved`. Proof fingerprints
(`.tlacache`) are written next to your sources under `/project` and survive
as part of your project files.

### Selecting a proof backend

TLAPS tries backends in the order SMT (Z3), Zenon, and Isabelle by default.
You can also name a backend explicitly in a `BY` clause:

```tla
THEOREM t == 2 + 2 = 4 BY Z3      \* SMT solver (Ubuntu's z3)
THEOREM t == TRUE /\ TRUE BY Zenon \* first-order tableau prover
THEOREM t == TRUE /\ TRUE BY Isa  \* Isabelle/TLA+ (tactic `auto`)
```

Timeouts can be tuned with `BY Z3T(60)`, `BY ZenonT(30)`, `BY IsaT(60)`,
and Isabelle tactics with `BY IsaM("blast")`. `tlapm --config` lists the
backends found in the workshop; `tlapm --help` documents all options.

The LS4 backend for propositional temporal logic (`BY PTL`) is also packaged.
Optional solvers that TLAPM knows about but this SDK does not ship (CVC4,
Yices, VeriT, SPASS, Zipperposition) report as missing, which is expected.

---

## Plugs (resources this SDK consumes)

This SDK doesn't define any plugs.

## Slots (resources this SDK provides)

This SDK doesn't define any slots.

---

## Dependency provenance

The SDK builds TLAPM from the maintained upstream source, pinned in the
`VERSION` file (currently the `1.6.0-pre` rolling prerelease line, commit
`85b548a`, which includes the reduced-Isabelle packaging of
[tlapm#292](https://github.com/tlaplus/tlapm/pull/292) and the heap
relocation fix of [tlapm#297](https://github.com/tlaplus/tlapm/pull/297)).

| Component | Source | Notes |
| --- | --- | --- |
| `tlapm` | `tlaplus/tlapm` source at the pinned commit | Built during the SDK build with Ubuntu's OCaml 4.14 |
| OCaml toolchain | Ubuntu 24.04 archive (`ocaml`, `opam`) | `opam` resolves TLAPM's OCaml library dependencies at build time from the opam repository; these are not version-locked |
| Dune | opam repository | Ubuntu 24.04 ships 3.14; TLAPM requires >= 3.15, so Dune comes from opam |
| Z3 | Ubuntu 24.04 archive (`z3` 4.8.12) | Installed at runtime by the `setup-base` hook. Upstream's build otherwise downloads a prebuilt Z3 binary from a GitHub release; that rule is removed in this SDK |
| Zenon | Vendored in the TLAPM source tree | Built during the SDK build |
| Isabelle/TLA+ | Official [Isabelle2025](https://isabelle.in.tum.de/website-Isabelle2025/) archive (SHA-256 pinned by upstream), plus the TLA+ object logic from the TLAPM source | Build-time only. The heaps (`Pure`, `TLA+`, `Options`) and a minimal Poly/ML runtime are built into the SDK; the JVM and the rest of the Isabelle distribution are neither shipped nor needed at runtime |
| LS4 (`BY PTL`) | `quickbeam123/ls4` v1.0 source, patched by upstream | Built from source during the SDK build |

Exceptions to the "Ubuntu archive first" rule are the Isabelle2025
distribution, the opam-resolved OCaml libraries, and the LS4 source archive;
each is listed above with its origin. The SDK deliberately does not use any
GitHub-hosted binary release tarball.

---

## Documentation and guidance

- [TLA+ Proof System documentation](https://proofs.tlapl.us/)
- [TLAPS tutorial](https://proofs.tlapl.us/doc/web/content/Documentation/Tutorial.html)
- [Backend tactics reference](https://proofs.tlapl.us/doc/web/content/Documentation/Tutorial/Tactics.html)
- [TLAPM sources](https://github.com/tlaplus/tlapm)

---

## Community and support

- TLA+ community: [TLA+ Google Group](https://groups.google.com/g/tlaplus)
- Workshop forum: [Discourse](https://discourse.ubuntu.com/)
- Please review our [Code of Conduct](https://ubuntu.com/community/ethos/code-of-conduct)
  before participating.

---

## Contributions

All contributions, including code, documentation updates, and issue reports,
are welcome!

- See [CONTRIBUTING](https://github.com/tlaplus/tlapm/blob/main/CONTRIBUTING.md)
  for guidelines on changes to the packaged software.
- Open issues or pull requests on the
  [official repository](https://github.com/marco6/tlapm-sdk).

---

## License and copyright

Copyright 2026 the TLAPM SDK authors.

The SDK packages the following components, each under its own license:

- TLAPM (main codebase): BSD-2-Clause — © 2008 INRIA & Microsoft Corporation,
  © 2023 Linux Foundation
- Zenon: BSD-style license — © 1997-2006 INRIA
- Isabelle2025: BSD-3-Clause — Technische Universität München
- Poly/ML 5.9 and the TLAPM `translate` directory (installed as
  `ptl_to_trp`): LGPL-2.1
- Z3: MIT — provided by the Ubuntu `z3` package at runtime, not shipped
