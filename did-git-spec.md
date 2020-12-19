# Git DID Method Specification
### A Git repository decentralized identifier method

*This version:*
* [https://github.com/dhuseby/did-git-spec](https://github.com/dhuseby/did-git-spec)

*Latest published version:*
* N/A

*Contributors in alphabetical order:*
* [Kim Duffy](https://github.com/kimdhamilton)
* [John Hopkins](https://github.com/phoniks)
* [Dave Huseby](https://github.com/dhuseby)
* [Duane Johnson](https://github.com/canadaduane)
* [Thomas Shelton](https://github.com/twshelton)
* [Manu Sporny](https://github.com/msporny)
* [Orie Steele](https://github.com/OR13)

*Participate:*
* [GitHub dhuseby/did-git-spec](https://github.com/dhuseby/did-git-spec)
* [File a bug](https://github.com/dhuseby/did-git-spec/issues)
* [Commit history](https://github.com/dhuseby/did-git-spec/commits/master)
* [Pull requests](https://github.com/dhuseby/did-git-spec/pulls)
* [Discussion list](https://groups.io/g/did-git/topics)

## About
This [DID method spec](https://w3c-ccg.github.io/did-spec/#specific-did-method-schemes)
conforms to the requirements in the DID specification currently published by
the W3C Credentials Community Group. For more information about DIDs and DID
method specifications, please see the [DID Primer](http://bit.ly/2RX0xm2) and
[DID Spec](https://w3c-ccg.github.io/did-spec/).

## Abstract 

Git is a revision control system designed to enable contributors to collaborate
in a distributed fashion with no centralized repository. Git supports digitally
signing changes with GPG/GPGSM but it is of limited utility because of the lack
of widespread adoption of GPG and the infrastructure for key escrowing. This
document defines the "git" [DID
Method](https://w3c-ccg.github.io/did-spec/#specific-did-method-schemes) for
using a Git repository as a source of truth about the repository identity and
its contributors' identities.

## Status of This Document
This document was published as an Editor's Draft.

The [discussion list](https://groups.io/g/did-git/topics) and [GitHub
issues](https://github.com/dhuseby/did-git-spec/issues) are the preferred way
to discuss issues with this specification.

Publication as an Editor's Draft does not imply endorsement by the W3C
Membership. This is a draft document and may be updated, replaced or obsoleted
by other documents at any time. It is inappropriate to cite this document as
other than a work in progress.

## Introduction

The Git revision control tool is designed to function in a decentralized
peer-to-peer fashion to facilitate collaboration in the frequently-disconnected
world. Git uses a directed acyclic graph (DAG) of commits that represent the
changes to the folders and files in the repository. Because it uses
blockchain-like hash-linking of commits, Git is effectively a blockchain and
distributed ledger with the patch review and merge process functioning as the
consensus mechanism. This makes it a great tool for tracking the provenance of
data inside the repository. Git also records the author and other meta data
such as digital signatures with each commit linking identity of committers to
each commit. Git repos therefore contain all of the information needed to serve
as the single source of truth for the provenance of the data it contains and
the identities of the contributors that created it.

## Motivation

### Cryptographic Signing of Git Commits is Broken

For a while now, Git has had the ability to sign commits using GPG. The feature
is fraught with issues that make the commit siging difficult to use and
limits its value. Even the Git developers themselves recognize the faults of
the GPG signing system. In the Git project documentation on submitting patches
they state:

```
Do not PGP sign your patch. Most likely, your maintainer or other people on the
list would not have your PGP key and would not bother obtaining it anyway.
Your patch is not judged by who you are; a good patch from an unknown origin
has a far better chance of being accepted than a patch from a known, respected
origin that is done poorly or does incorrect things.
```

The first issue is that Git depends solely on GPG for signing and verifying
digital signatures. GPG is known for being primarily a Linux/BSD tool with
less-than-stellar support for Windows and Mac. GPG also has a bunch of user
experience issues that makes it difficult for users to get commit signing and
verification working properly.

The second issue, as stated by the Git developers, is that digital signatures
on commits only add value when everybody has the public keys of all commit
signers. Even if the verifier has all of the keys, the validity of the keys
depends on an air-tight web-of-trust and assumes that the public key
infrastructure (PKI) problem is solved--we all know it isn't.

For commit signing to be more useful, all of the above issues need to be
addressed and it is clear that even if Git is fixed, GPG won't be fixed any
time soon. That's why a new identity management system and signing tool is
needed. This spec aims to support the creation of new commit signing regime for
Git that adds real value.

#### In-band Identity Management

To address the issue of distributing the public keys of commit signers, it
makes the most sense to store the public keys in the Git repository itself.
By putting the verification data in-band, cloning a repo is all that is needed
to have all of the data required to verify signed commits. GPG fails in this
regard because all of the keys are stored in personal keyrings outside of the
repo and there is no obvious way for those keyrings to be synchronized.

One possible solution is to store a GPG keyring inside of the Git repository.
The difficulty with this is that the keyring itself is a binary file. This
makes it difficult to inspect and manage the identities in the keyring as well
as rendering merging impossible. It also prevents the repo from tracking
changes to individual identities over time and providing a clear history and
provenance over signing keys.

The best solution is to store identies in text form with one identity per file.
This is how Git likes files to be. Not only does that give us a good history
for the identity files but if all identity files are stored in one directory,
that directory can function like a keyring in the GPG sense. Text-based
identities enable Git to manage them the same way that it does source code
files including merging, moving, adding and deleting giving users a clear
trail of provenance evidence. It also means that `git blame` will work on the
identity files themselves. The provenance of the signature verification keys
and their attendant meta data when modified with self-signed commits creates a
cryptographically verifiable system of management.

As part of this spec, the standard for storing text-based identities in a Git
repo is described. The standard uses the new W3C standard for storing
identities in human readable and machine parsable text called a [decentralized
identifier (DID) document](https://w3c-ccg.github.io/did-spec/). The
specification for the organization of the DID documents in a Git repo is
detailed in the Implementation details section at the bottom of this document.

### Intellectual Property Provenance

Many open source organizations such as the Linux Foundation have legal
requirements that all code contributions are made by developers under the terms
of either the [Developer Certificate of
Origin](https://developercertificate.org/) or a [Contributor License
Agreement](https://identity.linuxfoundation.org/projects/cncf). Under the DCO
regime, developers are supposed to signal their assertion by using Git's
[`--signoff`](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---signoff)
feature. However, over time this has proven to be a less-than-perfect solution
since there are many ways in which commits sneak into a repo without a proper
sign-off. Under the CLA regime, and external mechanism is needed to for
signing up companies, employees, and individual contributors. None of that
information is stored in the repository and there is no easy way to know who
can and cannot contribute. More importantly, there is no easy way to know who
could and could not contribute at some given time in the past.

In theory, requiring that all commits to be digitally signed by committer
identities stored in the repository would be a much more fool-proof way of
maintaining the IP provenance regime for a project. But due to the problems
with GPG outlined in previous sections, the existing Git and GPG digital
signing and identity system is broken. 

The motivation for this DID method is to build a better solution for digitally
signing commits. By storing the identities in the repo as DID documents and
building a new signing tool that understands DID's and DID docs, all of the
problems that plague the existing Git and GPG system can be fixed. With DID's
and DID docs, we get a new IP provenance regime for Git repos that is both
cryptographically verifiable and also automatically enforceable using Git hook
scripts.

### Verifiable Claims Against Open Source Contributions

When used for open source development, Git repos are publicly accessible on the
internet. Part of this new DID-based regime is to create a globally unique
decentralized identifier for each repo that defines the scope of the identities
stored within. As detailed farther down, the repo DID is defined using the
SHA-1 commit hash for the commit that creates the initial set of DID documents
and establishes the DID regime in the repo. Because the SHA-1 hash is over the
commit metadata and includes the hashes of parent commits, it functions as a
"content address" that also verifies the integrity of the data.

One interesting side effect of this new DID regime is that cryptographically
verifiable claims and zero-knowledge proofs can be constructed about the repo
and the contributors to the repo. The easiest kind of proof would be to prove
that a contributor worked on a particular repo. But more complicated proofs can
also be constructed showing that a contributor was the author of specific code
down to the character.

These proofs become even more useful if a project wants to maintain a canonical
repository on a platform such as Github. If the project anchors a record in a
public block chain such as Sovrin or Bitcoin that specifies both the repo DID
and the canonical URL for the repo then verfiable claims and zero-knowledge
proofs against the repo become globally verifiable. The identities stored in
the repo would then be usable by any other internet service and could function
as like the ubiquitous "Sign on with Google/Facebook/Twitter/etc" except this
would be "Sign on with Hacker Reputation".

## Terminology

### Decentralize Identifier (DID)

As described in the [Decentralized identifiers (DIDs)
specification](https://w3c-ccg.github.io/did-spec/), a DID is defined as
consisting of three parts:

1. The URL scheme identifier: "did".
2. The DID method identifier. In this case "git".
3. The DID method-specific identifiers.

### Commit ID

A Git commit ID is the SHA-1 digest of every important aspect of a commit
including the following:

1. The contents of the commit.
2. Commit date.
3. Committer's name and email address.
4. Log message.
5. The commit ID of the parent commit(s).

The Git documentation contains a [more detailed
description](https://git-scm.com/book/en/v1/Getting-Started-Git-Basics#Git-Has-Integrity)
of what is included in a commit ID. In the context of this method spec, a
commit ID serves as a content address for a specific commit.

### Repo ID

A repo ID is the commit ID of the commit that establishes the DID regime in a
repo. When initializing the DID regime a "did" folder and a number of DID
documents are created and committed to the repo. The commit ID for that commit
is the repo ID.

### Contributor ID

A contributor ID is the commit ID of the commit that added the contributors'
DID document to the repo. This is so that the contributor ID is not only tied
to a point in the repo history but it also remains stable over time. Once an
identity is established in a repo it will be valid as long as the repo remains.

Aliases in the DIDdir can be used to facilitate quickly looking up a DID
document by key ID.

### Key ID

A key ID is the [Base58](https://en.wikipedia.org/wiki/Base58)-encoded,
[SHA-256](https://en.wikipedia.org/wiki/SHA-2) digest of the contributor's
public key stored in their DID document in the repo.

## Git DID Method

The namestring that shall identify this DID method is `:git`.

A DID that uses this method MUST begin with the following prefix: `did:git`.
Per the DID specification, this string MUST be in lowercase. The remainder of
the DID, after the prefix, is the NSI specified below.

This DID method applies to Git beginning with the release of Git that supports
an external DID signing tool and the `git did` procelain command.

### Namespace Specific Identifier (NSI)

The `did:git` namestring is defined by the following
[ABNF](ftp://ftp.rfc-editor.org/in-notes/std/std68.txt):

```abnf
git-did = "did:git:repo-id" 1*(":" contributor-id) 1*(";" did-service) 
          1*("/" did-path) 1*("?" did-query) 1*("#" did-fragment)
repo-id = commit-id
contributor-id = commit-id
commit-id = 40*(lowerhex)
lowerhex = "0" / "1" / "2" / "3" / "4" / "5" / "6" / "7" / "8" / "9" / "a" /
           "b" / "c" / "d" / "e" / "f"
```

In the context of this method spec, there are two valid DID formats:

1. The DID for just the repo: `did:git:<Repo ID>`.
2. The DID for a contributor identity in the repo: `did:git:<Repo ID>:<Contributor ID>`.

[DID parameter names](https://w3c-ccg.github.io/did-spec/#generic-did-parameter-names)
are used on the end of a DID for an identity to select specific subjects within
the contributor's DID document.

### (Optional) JSON-LD Context Definition

### (Optional) Core Data Model

### Basic Concepts

Interracting with a Git repo using the CRUD operations on a Git DID is done
using a new Git porcelain command called `git did`. The porcelain command is
called using the following syntax:

```
$ git did <crud-command> <did> [options]
```

The crud-command is one of the following: `create`, `read`, `update`, and
`delete`. The operations do different things depending on the DID that is
passed as the first parameter. There is one special case when the repo DID
regime is being established that the DID is unknown and therefore omitted. To
establish the DID regime over a repo run the following command either inside of
the repository or with the `--repo` option:

```
$ git did create
$ git did create --repo=/foo/bar/baz
```

## CRUD Operations

### Create 

#### Repository DID

The operation of establishing the DID regime over a repo can happen at any time
in the history of the repo. To do so, a number of DID document templates need
to be added to the repo. The DID Document template for the repo is named
`did/repo.did`. All of the maintainer DID document templates must be added into
the did/ directory as well. All files are then signed and committed by one of
the associated keys in the authentication block into the project. The SHA-1 of
the commit is then used to create the git-did for this respository. 

For example:

To create a git-did, you must create a `did/repo.did` file that looks like this:
```jsonld
{
  "@context": "https://wsid.org/git-method/v1",
  "id": "did:git:$repo-id",
  "service": [{
    "type": <gitService>,
    "serviceEndpoint": <Canonical URI for the respository>
    }],
  "authentication": [
      // Original maintainers/controllers of this repository should be listed
      // here to comply with governance. Thecommit should be signed by one of
      // these keys.  
    "<key-id of controller 1>",
    "<key-id of controller-2>",
    "<key-id of controller-n>"
  ]
}
```

When executing a read operation on the repo DID, the repo.did document will
have the `$repo-id` string dynamically replaced with the commit ID of the
commit in which the repo.did file was added to the repo. For example, the
result of a read on the repo DID would look something like this:

```jsonld
{
  "@context": "https://wsid.org/git-method/v1",
  "id": "did:git:625557b5a9cdf399205820a2a716da897e2f9657",
  "service": [{
    "type": "git",
    "serviceEndpoint": "https://github.com/example/example.git"
    }],
  "authentication": [
    "123456789abcdefghj#controller-1",
    "123456789abcdefghj#controller-2",
    "123456789abcdefghk#controller-n"
    ]
}
```
#### Contributor DID

To create a Contributor DID, add public DID document into the did/ directory
and name it using a `<author>.did`. The name of the file is arbitrary and

An example contributor DID template is shown below.
```jsonld
{
  "@context": "https://wsid.org/git-method/v1",
  "id": "did:git:625557b5a9cdf399205820a2a716da897e2f9657:$contributor-id", 
  "publicKey": [{
    "id": "did:git:625557b5a9cdf399205820a2a716da897e2f9657:$contributor-id#1",
    "type": "Ed25519VerificationKey2018",
    "publicKeyBase58": "DpS9NGz4dXmQZ27VeLzsC5DpJzUBMztQCon6JBAQAu7q"
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
The commit adding your DID document should be signed by the private key
associated with the public key referenced in the DID document.

### Read

Notes: since we don't know the repo did string when the repo DID document is
created, the repo DID document will not contain the "id" did string for the
repo. The `git did read` operation will "resolve" the "id" by looking up the
SHA-1 hash of the commit that added the DID document to the repo and dynamically
add the "id" member to what is rendered to the user.

### Update 

#### Repository DID

There aren't currently any keys associated with the respository, so there will
be no need to update the Public Keys in the DID document.  The process to
change the maintainers or the canonical endpoint should be defined in the
`did/governance.md` file.

#### Contributor DID

To update a Contributor DID document, update the current DID document in the
repository and rename the file to reflect the new public key.  Once all changes
have been made, commit and sign with the private key of the DID document before
it was changed.

### Delete (Revoke)

#### Repository DID

The DID for the repository can be deleted by removing the `did/repo.did` file
from the respository and performaing a commit.  The signature of this commit
should correspond to one of the maintainers in the `did/repo.did#authentication
list. 
    
#### Contributor DID

Deletion of a personal did will remove the file from `did/<author key id>.did`.
The `<author did key>.did` is maintained in history which supports provability.
The signature over this change should correspond to one of the maintainers in
the `did/repo.did#authentication list or the private key of the associated DID
document itself.
    
TODO:
- [ ] What happens when re-established?  (confirm: new repo.did = new commit hash = new did?)
- [ ] Is this how we give first-class status to branches/forks - e.g. fork/branch, delete repo.did, re-create new repo.did in each relevant position

## Security Considerations

This spec applies only to versions of git >= 2.13.0, which used hardened SHA-1.
Versions less than v.2.13.0 are susceptible to a [critical hash collision
vulnerability](https://github.com/git/git/blob/master/Documentation/technical/hash-function-transition.txt),
and MUST NOT be used for implementing this method.

## Privacy Considerations

Since DIDs can be resolved by anyone, care should be taken to ensure the DID
Document does not contain any sensitive personal information. One primary
consideration is how do we support the deleting of personal information in the
repo since it is effectively a blockchain. The answer is the same in this
context as it is for other blockchain based DID systems: we don't store any
personal information in the DID document. DID documents in Git repos should be
limited to just public keys and service endpoints for key recovery.

## Reference Implementations

## Implementation Notes

### DIDdir Structure

DIDdir is the answer to the in-band identity management problem outlined in the
motivation section above. The primary idea for DIDdir is that a folder serves
as the root of a keyring of identities. Within the DIDdir root are identity
files (DID-doc), one for each identity. The name of each DID-doc is the public
key identifier (key-id) associated with the identity in the DID document. The
key-id is the Base58 encoded, SHA-256 digest of the identity public key.

Also inside of the DIDdir root is a folder called `aliases` (Alias-dir) that
stores files (Alias-doc) that map aliases to DID docs. Each file in the
Alias-dir is named the same as the alias and the contents of the file is the
name of the DID-doc that the alias maps to.

Also under the DIDdir root is a folder called `tmp` that is used for atomic
updates to the DIDdir to avoid corruption. This is inspired by
[Maildir](https://en.wikipedia.org/wiki/Maildir) and serves the same function.
Whenever a new DID-doc or Alias-doc is created, it is first saved into the
`tmp` folder and then the file rename command is used to move it into the
DIDdir root or the `aliases` folders. The operating system file rename
operation is atomic on all operating systems and eliminates data races and the
possibility of a process crash causing corruption.

#### Key-ID

A key-id is the shorthand version of a public key that is calculated from the
public key bytes itself. A key-id is the Base58 encoding of the SHA-256 digest
of the public key. This allows for a uniform size naming scheme for DID
documents that may have public keys of different sizes and of different
algorithms.

The ABNF of the key-id is as follows:

```
key-id = 44*(base58)
base58 = "1" / "2" / "3" / "4" / "5" / "6" / "7" / "8" / "9" / "A" / "B" /
         "C" / "D" / "E" / "F" / "G" / "H" / "J" / "K" / "L" / "M" / "N" /
         "P" / "Q" / "R" / "S" / "T" / "U" / "V" / "W" / "X" / "Y" / "Z" /
         "a" / "b" / "c" / "d" / "e" / "f" / "g" / "h" / "i" / "j" / "k" /
         "m" / "n" / "o" / "p" / "q" / "r" / "s" / "t" / "u" / "v" / "w" /
         "x" / "y" / "z"
```

#### Aliases

The `aliases` folder under the DIDdir root contains files named with the
aliases and containing the name of the DID-doc that it maps to. This is handy
for allowing users of DIDdir to specify identities by aliases instead of the
long and difficult-to-remember key-id strings. For instance, to create an email
alias for a DID-doc, create a file named with the email address and containing
just DID-doc name. Any number of aliases can be created for an identity and
they can be named anything you want. This is mostly a convenience for
applications built on top of DIDdir.

If an alias file named `default` exists, the DIDDoc it maps to is assumed to be
the "default" identity. Applications that use DIDdir may allow users to omit
specifying an identity directly and the default identity will be chosen
automatically. This is a convencience to make DIDdir applications simpler to
run.

#### Permissions

DIDdir requires that the root folder and all child folders be readable,
writable, and executable by only the owner. All files must only be readable and
writable by the owner. On Linux that means the folders are 700 and files are
600. All implementations must check both folder and file permissions and raise
an error/exception if any of the files are not correct.

#### Default User DIDdir

Like GPG, DIDdir powered applications default to storing a personal DIDdir in
the user's personal data storage (e.g. $HOME/.local/share/keydir on Linux and 
{FOLDERID_LocalAppData}/rwds/keydir on Windows). However unlike GPG, DIDdir
applications first look in the present working directory and its ancestors
looking for a Git repo root and then looking for a `did` directory within it.
If it finds a `did` dir, it assumes that DIDdir is the primary DIDdir. DIDdir
apps also include the user's personal DIDdir as a secondary source and falls
back to that when the DID doc specified by the DID is not found in the primary
DIDdir.

### Implementation Details

- the git did information will be stored in the `did` directory, and will function similar to README.md + LICENSE + CONTRIBUTORS boilerplate currently recommended.
    - by not placing it in the .git directory, means that the information can participate in the git history and shallow clones
- Layout of `did` directory?
    - `did/repo.did`
        - include hash of governance.json?
    - `did/<author key id>did`
    - `did/governance.json`
        - Considering moving this to the repo.did as a path
        - These are the contributor terms and conditions
        - acceptance of the governance model is indicated in each `did/<author key id>.did` by including a signature over the governance.md
- Schemas (@context)
    - `did/repo.did`
        - path to governance.json
        - canonical URL
        - list of maintainers
            - every element is `the <author key id>.did`
        - optional
            - equivId -> [points to external ids]
            - roll -> used to indicate maintainer/contributor status

## Resources

The [Peer DID Method
Spec](https://dhh1128.github.io/peer-did-method-spec/index.html) is very
similar in nature and can be used as a reference point.
