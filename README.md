# W3C DID Method Spec for `did:git:`

This repo is the central coordination point for work on the
[W3C DID method specification](https://w3c-ccg.github.io/did-spec/) for
Git repositories. If you would like to participate, we encourage you to join
the [mailing list](https://groups.io/g/did-git/topics) and make sure you're
familiar with decentralized identifiers (DID) and the problem we're trying to
solve by reading the [DID primer](https://w3c-ccg.github.io/did-primer/).

## Spec TODO:

- [ ] Add version to `did-git-spec.md` see [method registry requirement 2](https://w3c-ccg.github.io/did-method-registry/#the-registration-process).
- [ ] Request that the spec be include in the [DID method registry](https://w3c-ccg.github.io/did-method-registry/).
- [ ] Add terminology section to the spec to define repo ID, committer ID, etc.
- [ ] Complete the sections on motivation.
- [ ] Finish the CRUD operations for both repo and committer DID strings.
- [ ] Reconcile difference between `did-dir-spec.md` and `did-git-spec.md` to create standard for on-disk DID documents as keyring.

## Impl TODO:

- [x] Create a [fork](https://github.com/dhuseby/did-git-impl) of [git.git](git://git.kernel.org/pub/scm/git/git.git).
- [x] Create a [`did-git-impl-signing` branch](https://github.com/dhuseby/did-git-impl/tree/did-git-impl-signing) off of [git.git `maint` branch](https://github.com/dhuseby/did-git-impl/tree/maint) for signing infrastructure changes.
- [x] Create a [`did-git-impl-porcelain` branch](https://github.com/dhuseby/did-git-impl/tree/did-git-impl-porcelain) off of [git.git `maint` branch](https://github.com/dhuseby/did-git-impl/tree/maint) for creating git-did porcelain.
- [ ] Rebase patches for signing infrastructure changes.
- [ ] Define the format of DID signatures in commit records.
- [ ] Create a simple `git-did` porcelain command from the Python examples in the git.git repo.
- [ ] Define and document the "meat-space" protocol for doing multiple signatures on a single commit. This is likely to be a threshold signature and the git-did porcelain should automate it as much as possible.

