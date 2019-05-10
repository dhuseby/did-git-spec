Note: for everything marked (Optional),


# Abstract

Git is a revision control system designed to enable contributors to collaborate in a distributed fashion with no centralized repository. Git supports digitally signing changes with GPG/GPGSM but it is of limited utility because of the lack of widespread adoption of GPG and the infrastructure for key escrowing. This document defines the [DID Method](https://w3c-ccg.github.io/did-spec/#specific-did-method-schemes) for using a Git repository as a source of truth about the repository and its contributors.

# Introduction

The Git revision control tool is designed to function in a decentralized peer-to-peer fashion to facilitate collaboration in the frequently-disconnected world. Git uses a directed acyclic graph (DAG) of commits that represent the changes to the folders and files in the repository. Because it uses blockchain-like hash-linking of commits, Git is an effective tool for tracking the provenance of data inside the repository. Git also records the author and other meta data such as digital signatures with each commit linking identity of committers to each commit. Git repos therefore contain all of the information needed to serve as the single source of truth for the provenance of the data it contains and the identities of the contributors that created it.

## Motivation

### Developer Certificate of Origin Compliance

The Linux Foundation requires that all contributions to their open source projects are made by individuals that assert that their contribution adheres to the terms of the [Developer Certificate of Origin](https://developercertificate.org/). The contributor signals their assertion by using either Git's [`--signoff`](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---signoff) feature, [digital signature](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work) feature, or both when committing their changes.


### Verifiable Claims Against Open Source Contributions

### Authorship and Intellectual Property Rights

### Shared Overlay Repositories

The Overlay Data Capture Architecture aims to be a universal tool for data interoperability.  It provides for multi-dimensional schemas which can be dynamically linked to allow for data pooling and frictionless data sharing. In the case of multiple parties wanting to work together to create standard schemas for a particular use case (an industry consortium standardizing product labels, for instance) but who don't necessarily trust one another, this method provides the ability to enforce a governance framework for managing a shared repository while not giving any one entity the keys to the castle.  

# Git DID Method

The namestring that shall identify this DID method is `:git`.

A DID that uses this method MUST begin with the following prefix: `did:git`. Per the DID specification, this string MUST be in lowercase. The remainder of the DID, after the prefix, is the NSI specified below.

# Target System(s)

This DID method applies to Git beginning with the release of Git that supports the DID signing tool.

# Namespace Specific Identifier (NSI)

The `:git` namestring is defined by the following [ABNF](ftp://ftp.rfc-editor.org/in-notes/std/std68.txt):

```abnf
git-did = "did:git:commitid" 1*(":" keyid) 1*(";" did-service) 1*("/" did-path) 1*("?" did-query) 1*("#" did-fragment)
commitid = 40*(lowerhex)
keyid = 64*(lowerhex)
lowerhex = "0" / "1" / "2" / "3" / "4" / "5" / "6" / "7"
    / "8" / "9" / "a" / "b" / "c" / "d" / "e" / "f"
```

All Git DIDs consist of one lower case hexidecimal string that is the commit ID for the commit that added the `repo.did` document for the repo and an optional lower case hexidecimal string that is the SHA-256 hash of the public key associated with a DID document stored in the repo itself.

The commit ID portion becomes a globally unique identifier for the Git repository and is immutable for the life of the Git repository. The "constitution" file contains the list of key IDs of the maintainers

# (Optional) JSON-LD Context Definition

# (Optional) Core Data Model

# (Optional) Basic Concepts

# CRUD Operations

## Create

**1. Repository DID**

The operation of creating a git-did can be done at any point in a git repository.

A DID Document should be created at `.did/repo.did` and all maintainer DID documents added into the .did/ directory. All files are then signed and committed by one of the associated keys in the authentication block into the project.  The SHA1 of the commit is then used to create the git-did for this respository.

For example:

To create a git-did, you must create a `.did/repo.did` file that looks like this:
```jsonld
{
  "@context": "https://wsid.org/git-method/v1",
  "id": "did:git:<placeholder>", //should this be in the document?
  "service": [{
    "type": <gitService>,
    "serviceEndpoint": <Canonical URI for the respository>
    }],
  "authentication": [
      // original maintainers/controllers of this repository should be listed here to comply with governance.  Origin commit should be signed by one of these keys.  
    "<hash-of-pub-key>#controller-1",
    "<hash-of-pub-key>#controller-2",
    "<hash-of-pub-key>#controller-n",
}
```

Example:
```jsonld
{
  "@context": "https://wsid.org/git-method/v1",
  "id": "did:git:000000000000",
  "service": [{
    "type": <gitService>,
    "serviceEndpoint": "https://github.com/example/example.git"
    }],
  "authentication": [
    "123456789abcdefghi#controller-1",
    "123456789abcdefghj#controller-2",
    "123456789abcdefghk#controller-n"
    ]
}
```
**2. Contributor DID**

To create a Contributor DID, add public DID document into the .did/ directory and name it using a `<author key id>.did`

An example DID doc is shown below.
```jsonld
{
  "@context": "https://wsid.org/git-method/v1",
  "id": "did:git:<commit SHA1>:<author key id>",
  "publicKey": [{
    "id": "did:git:<commit SHA1>:<author key id>",
    "type": "<signature type>",
    "publicKeyBase58": "..."
  }
}
```
Example:
```jsonld
{
  "@context": "https://wsid.org/git-method/v1",
  "id": "did:git:abcde12345/hijklmn678919",
  "publicKey": [{
    "id": "did:git:abcde12345/hijklmn678919",
    "type": "ED25519SignatureVerification",
    "publicKeyBase58": "..."
  }
}
```
The commit adding your DID document should be signed by the key associated private key of the public key referenced in the DID document.

## Read

Notes: since we don't know the repo did string when the repo DID document is created, the repo DID document will not contain the "id" did string for the repo. The `git did read` operation will "resolve" the "id" by looking up the SHA1 hash of the commit that added the DID document to the repo and dynamically add the "id" member to what is rendered to the user.

## Update
**1. Repository DID**

There aren't currently any keys associated with the respository, so there will ne no need to update the Public Keys in the DID document.  The process to change the maintainers or the canonical endpoint should be defined in the `.did/governance.md` file.

**2. Contributor DID**

To update a Contributor DID document, remove the previous DID document from the repository and commit.  This commit should be signed by the private key of the DID document being removed.  Then create a new DID document with the appropriate information and commit to the repository.  This commit should be signed by the private key of the DID document being added.

TODO:

1. We need a way to verify that the person making each of these changes has the appropriate keys to prevent unauthorized updates.  Is there a cleaner way to do this?

## Delete (Revoke)

**1. Repository DID**

The DID for the repository can be deleted by removing the `.did/repo.did` file from the respository and performaing a commit.  The signature of this commit should correspond to one of the maintainers in the `.did/repo.did#authentication list.

**2. Contributor DID**

Deletion of a personal did will remove the file from `.did/<author key id>.did`.  The `<author did key>.did` is maintained in history which supports provability.  The signature over this change should correspond to one of the maintainers in the `.did/repo.did#authentication list or the private key of the associated DID document itself.

TODO:
    - What happens when re-established?  (confirm: new repo.did = new commit hash = new did?)
    - Is this how we give first-class status to branches/forks - e.g. fork/branch, delete repo.did, re-create new repo.did in each relevant position

# Security Considerations

This spec applies only to versions of git >= 2.13.0, which used hardened SHA-1. Versions less than v.2.13.0 are susceptible to a [critical hash collision vulnerability](https://github.com/git/git/blob/master/Documentation/technical/hash-function-transition.txt), and MUST NOT be used for implementing this method.

# Privacy Considerations

Since DIDs can be resolved by anyone, care should be taken to ensure the DID Document does not contain any sensitive personal information.

# Reference Implementations

# Implementation Notes

- the git did information will be stored in the .did directory, and will function similar to .gitignore
    - by not placing it in the .git directory, means that the information can participate in the git history and shallow clones
- Layout of .did dir?
    - `.did/repo.did`
        - include hash of governance.md?
    - `.did/<author key id>.did`
    - `.did/governance.md`
        - Considering moving this to the repo.did as a path
        - These are the contributor terms and conditions
        - acceptance of the governance model is indicated in each `.did/<author key id>.did` by including a signature over the governance.md
- Schemas (@context)
    - `.did/repo.did`
        - path to governance.md
        - canonical URL
        - list of maintainers
            - every element is `the <author key id>.did`
        - optional
            - equivId -> [points to external ids]
            - roll -> used to indicate maintainer/contributor status

# Resources
The Peer DID Method Spec is very similar in nature and can be used as a reference point.
(https://dhh1128.github.io/peer-did-method-spec/index.html)
