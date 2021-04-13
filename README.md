# NOTE: THIS SPEC HAS BEEN WITHDRAWN FROM THE W3C DID METHOD REGISTRY
The future of developer provenance in Git repos is being implemented using [authentic data provenance logs](https://github.com/TrustFrame/authentic-data-specifications/blob/main/Authentic%20Data%20Provenance%20Log.md) instead of DID documents. DID documents do not support history and DID methods conflate identification with location and it has resulted in siloing and politics; [don't use DIDs](https://github.com/TrustFrame/authentic-data-specifications/blob/main/Authentic%20Data%20Provenance%20Log.md). The signing tools used for this will all communicate with Git using [a new protocol under development](https://github.com/TrustFrame/git-cryptography-protocol/blob/main/Git%20Cryptography%20Protocol.md).

# W3C DID Method Spec for `did:git:`

This repo is the central coordination point for work on the
[W3C DID method specification](https://w3c-ccg.github.io/did-spec/) for
Git repositories. If you would like to participate, we encourage you to join
the [mailing list](https://groups.io/g/did-git/topics) and make sure you're
familiar with decentralized identifiers (DID) and the problem we're trying to
solve by reading the [DID primer](https://w3c-ccg.github.io/did-primer/). This
work is broken up into three major areas: defining the specification, modifying
Git, and creating a new signing tool. Bellow are the outstanding TODO's for
each area.

## Spec TODO:

- [x] Add version to `did-git-spec.md` see [method registry requirement 2](https://w3c-ccg.github.io/did-method-registry/#the-registration-process).
- [x] Request that the spec be include in the [DID method registry](https://w3c-ccg.github.io/did-method-registry/).
- [x] Add terminology section to the spec to define repo ID, committer ID, etc.
- [x] Complete the sections on motivation.
- [ ] Finish the CRUD operations for both repo and committer DID strings.
- [x] Reconcile difference between `did-dir-spec.md` and `did-git-spec.md` to create standard for on-disk DID documents as keyring.

## Impl TODO:

- [x] Create a [fork](https://github.com/dhuseby/did-git-impl) of [git.git](git://git.kernel.org/pub/scm/git/git.git).

### Git Signing
- [x] Create a [`did-git-impl-signing` branch](https://github.com/dhuseby/did-git-impl/tree/did-git-impl-signing) off of [git.git `maint` branch](https://github.com/dhuseby/did-git-impl/tree/maint) for signing infrastructure changes.
- [ ] Rebase patches for signing infrastructure changes.
  - [x] Rebase documentation changes.
	- [ ] Rebase signing interface abstraction.
	- [ ] Rebase GPG and GPGSM "drivers".
	- [ ] Verify that all existing GPG unit tests pass.
- [ ] Submit signing fix patches to git.git mailing list.
  - [ ] Follow up on changes and discussion until it gets landed.
- [ ] Create BetterSign "driver".
- [ ] Create BetterSign unit tests.
- [ ] Submit BetterSign patches to git.git mailing list.
  - [ ] Follow up on changes and discussion until it gets landed.

### Git Porcelain

- [x] Create a [`did-git-impl-porcelain` branch](https://github.com/dhuseby/did-git-impl/tree/did-git-impl-porcelain) off of [git.git `maint` branch](https://github.com/dhuseby/did-git-impl/tree/maint) for creating git-did porcelain.
- [ ] Define the format of DID signatures in commit records. Proposal to use [Linked Data Proofs](https://github.com/dhuseby/did-git-spec/issues/9).
- [ ] Create a simple `git-did` porcelain command from the Python examples in the git.git repo.
- [ ] Add initial unit tests for `git-did` porcelain.
- [ ] Implement CRUD operations for repo DID strings.
- [ ] Implement CRUD operations for committer DID strings.
- [ ] Define and document the "meat-space" protocol for doing multiple signatures on a single commit. This is likely to be a threshold signature and the git-did porcelain should automate it as much as possible.

## BetterSign TODO:

- [x] Create a public [repo for BetterSign signing tool](https://github.com/dhuseby/bs).
- [x] Upload existing bs code.
- [x] Create a public [repo for the DIDDir library](https://github.com/dhuseby/diddir).
- [x] Upload existing diddir code.
- [ ] BetterSign
  - [ ] Refactor sign/verify functions to use [background-jobs](https://crates.io/crates/background-jobs)
	- [ ] Add DID string parser.
	- [ ] Add linked data structure (LDS) output format.
	- [ ] Add JSON web signature (JWS) output format.
- [ ] DIDdir
  - [ ] Add JSON-LD parser for DID documents.
	- [ ] Add Base58 crate for naming DID documents.
	- [ ] Add code to find DIDdir in local dir and ancestors.
	- [ ] Add git2 crate to handle Git commands for CRUD operations on DID docs.
