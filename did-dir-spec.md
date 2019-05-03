# DIDDir

DIDDir is a directory structure and set of rules inspired by DJB's MailDir for
the storage of identity information. DIDDir is similar to OpenGPG's keyring
except that instead of storing an opaque binary file with all identities,
DIDDir stores each identity in its own text file. The purpose is to make it
possible to manage a keyring of identities with a revision control system such
as Git.

## Motiation

For a while now, Git has had the ability to sign commits using GPG. This is
still fraught with issues that make the commit siging difficult to use and
limits its value.

The first issue is that the dependency on GPG means everybody verifying the
signed commits must have GPG installed on their computer. GPG is known for
being primarily a Linux/BSD tool with less-than-stellar support for Windows
and Mac. GPG also has a bunch of user experience issues that makes it difficult
for users to get commit signing and verification working properly.

The second issue is that the digital signatures on commits only adds value when
everybody verifying the signatures have the public keys of all developers that
sign their commits. And even if the verifier has all of the keys, they depend
on an trustworthy web-of-trust to ensure that the keys are actually valid. Git
commit signing assumes that the public key infrastructure (PKI) problem is
solved and we all know it isn't.

The third problem is that there is no support for multiple signatures in Git.
Ideally, when a project releases a new version, a new tag is created that is
signed by a release engineer and potentially other members of the project as
defined in the project's governance rules. This is the best possible signal
from the project members that the current release is the one that is official.

For commit signing to be more useful, all three of the above issues need to be
addressed and it is clear that even if Git is fixed, GPG won't be fixed any
time soon. That's why a new identity management system and signing tool need to
be created. The DIDDir spec is designed to support the creation of new signing
tools that work better with Git and other revision control systems to address
the problem.

## In-band Identity Management

To address the issue of distributing the public keys of commit signers, it
makes the most sense to store the public keys in the Git repository itself.
By putting the verification data in-band, cloning a repo is all that is needed
to have all of the data required to verify signed commits. GPG fails in this
regard because all of the keys are stored in personal keyrings on each user's
computer and there is no obvious way for those keyrings to be synchronized.
DIDDir aims to solve that.

One possible solution is to store a GPG keyring inside of the Git repository
that has signed commits. The difficulty with this is that GPG defaults to
merging keyrings specified on the command line with the user's personal keyring
in their home directory. This behavior can be changed by specifying the right
combination of GPG command line options but currently this requires every Git
user to set up custom commands for running GPG in their .gitconfig file.

Even if Git is fixed to use the correct GPG command line switches by default,
the keyring itself is a binary file. This makes it difficult to inspect and
manage the identities in the keyring as well as making merging impossible. The
best solution is to store the identies in text form with one identity per file
and all in a directory that is treated like a keyring. By using text and a
directory, the keyring can be managed by Git the same way that source code
files are, including merging, moving, adding and deleting. It also means that
`git blame` will work on the keyring files as well and the provenance of the
signature verification keys and their attendant meta data is managed in an
air-tight way.

There is a new W3C standard for storing identities in human readable and
machine parsable text called a [decentralized identifier (DID)
document](https://w3c-ccg.github.io/did-spec/). DIDDir specifies a way to
organize and manage DID documents in folder structure that can be used as a
keyring for a signing tool.

## DIDDir Structure

DIDDir is the answer to the in-band identity management problem outlined above.
The primary idea for DIDDir is that a folder serves as the root of a keyring of
identities. Within the DIDDir root is another folder called `dids` (DIDDir).
Inside of the DIDDir are files (DIDDoc), one for each identity that contains
the DID document. The name of each DIDDoc is arbitrary but a best practice is
to name each file with the public key identifier (PkId) associated with the
identity in the DID document.

Also inside of the DIDDir root is an optional folder called `aliases`
(AliasDir) that stores files (AliasFile) that map aliases to DIDDocs. Each file
in the AliasDir is named the same as the alias, the contents of the file is the
name of the DIDDoc that the alias maps to.

Also under the DIDDir root is a folder called `tmp` that is used for atomic
updates to the DIDDir to avoid corruption. This is also inspired by Maildir
and serves the same function. Whenever a new identity or alias is created, it
is first saved into the `tmp` folder and then the file rename command is used
to move it into the `ids` or `aliases` folders. The operating system file
rename operation is atomic on all operating systems and eliminates data races
and the possibility of a process crashing during the saving of an identity or
alias file and therefore corrupting the DIDDir data.

## PkId

A PkId is the shorthand version of a public key that is calculated from the
public key bytes itself. The encoding of a PkId is just the lower-case
hexidecimal encoding (LowerHex) of the SHA256 hash of the public key. This
allows for a uniform size naming scheme for DID documents that may have public
keys of different sizes and of different algorithms.

The ABNF of the PkId in LowerHex encoding is as follows:

```
pkid = 64*(lowerhex)
lowerhex = "0" / "1" / "2" / "3" / "4" / "5" / "6" / "7" / "8" / "9" / "a" 
  / "b" / "c" / "d" / "e" / "f"
```

## Aliases

The optional `aliases` folder under the DIDDir root contains files named with
the aliases and containing the name of the DIDDoc that it maps to. This is
handy for allowing users of DIDDir to specify identities by aliases instead of
the long and difficult-to-remember PkId strings. For instance, to create an
email alias for the DIDDoc, create a file named with the email address and
containing just DIDDoc name. Any number of aliases can be created for an
identity and they can be named anything you want. This is mostly a convenience
for applications built on top of DIDDir.

If an alias file named `default` is added to the `aliases` folder, the DIDDoc
it maps to is assumed to be the "default" identity. Applications that use
DIDDir may allow users to omit specifying an identity directly and the default
identity will be chosen automatically. This is a convencience to make DIDDir
applications simpler to run.

## Permissions

DIDDir requires that the root folder and all child folders be readable,
writable, and executable by only the owner. All files must only be readable and
writable by the owner. On Linux that means the folders are 700 and files are
600. All implementations must check both folder and file permissions and raise
and error/exception if any of the files are not correct.

## Default User DIDDir

Like GPG, DIDDir powered applications default to storing a personal DIDDir in
the user's personal data storage (e.g. $HOME/.local/share/keydir on Linux and 
{FOLDERID_LocalAppData}/rwds/keydir on Windows).
