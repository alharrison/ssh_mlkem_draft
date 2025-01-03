---
title: "Module-Lattice Key Exchange in SSH"
abbrev: "TODO - Abbreviation"
category: info

docname: draft-harrison-mlkem-ssh-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "alharrison/ssh_mlkem_draft"
  latest: "https://alharrison.github.io/ssh_mlkem_draft/draft-harrison-mlkem-ssh.html"

author:
 -
    fullname: "Alexander Harrison"
    organization: Cisco
    email: "aleharri@cisco.com"

normative:
  RFC2119:
  RFC4251:
  RFC4253:
  RFC6234:
  FIPS203:

informative:
  IANA-SSH:
    -: ta
    target: https://www.iana.org/assignments/ssh-parameters/ssh-parameters.xhtml
    title: "Secure Shell (SSH) Protocol Parameters"
    author:
      -
        org: IANA
    date: 2021
  FIPS203:
    -:
    target: https://doi.org/10.6028/NIST.FIPS.203
    title: "Module-Lattice-based Key-Encapsulation Mechanism Standard"
    author:
      -
        org: National Institute of Standards and Technology (NIST)
    seriesinfo:
      FIPS PUB: 203
    date: 12 Aug 2024


--- abstract

This document defines Post-Quantum key exchange methods based on Module-lattice post-quantum key encapsulation schemes.  These methods are defined for use in the SSH Transport Layer Protocol.

--- middle

# Introduction

Secure Shell (SSH) [RFC4251] is a secure remote login protocol. The key exchange protocol described in [RFC4253] supports an extensible set of methods. The security of traditional key exchange methods used in Secure Shell (SSH) [RFC4251] relies on the algorithms being too computationally complex to be broken. The development of quantum computers poses a threat to the complexity of these algorithms. Given sufficiently powerful quantum computers, these traditional algorithms would be vulnerable to attack.

Additionally, the threat of "harvest-now-decrypt-later" attacks could creates a risk in the current landscape before sufficiently powerful quantum computers are available. In this attack, the data would be collected and decrypted by these quantum computers at a later date.

This document addresses the problem by proposing the use of post-quantum key encapsulation mechanisms (KEMs) to extend the SSH [RFC4253] key exchange. The post-quantum KEM discussed in this document is ML-KEM which is based on CRYSTALS-KYBER. [FIPS203] standardized the ML-KEM scheme in 2024 with three parameter variants, ML-KEM-512, ML-KEM-768, ML-KEM-1024. ML-KEM is a NIST approved mechanism that is believed to be secure against an attacker with a quantum computer.


# Conventions and Definitions

{::boilerplate bcp14-tagged}
{::boilerplate bcp78-tagged}
{::boilerplate bcp79-tagged}

# Key Exchange Method: ML-KEM
When using ML-KEM as the Key Exchange Method, the shared secret key would be exchanged as described in section 3.1 of [FIPS203]. The initiating party, the client would begin with the generation of the (public) encapsulation and (private) decapsulation key. The (public) encapsulation key is then shared with the server. The server uses this public encapsulation key to encapsulate the shared secret key into the ciphertext and transmits this ciphertext back to the client. The client then uses their previously generated private key to decapsulate the shared secret received from the server's ciphertext.

## ML-KEM Key Exchange Message Numbers
When using ML-KEM as the Key Exchange Method, the following existing namespace message numbers MAY be used:
         #define SSH_MSG_KEX_ECDH_INIT               30
         #define SSH_MSG_KEX_ECDH_REPLY              31

## ML-KEM Key Exchange Method Names
The ML-KEM key exchange method names defined in this document (to be used in SSH_MSG_KEXINIT [RFC4253]) are
ml-kem-512-sha256
ml-kem-768-sha256
ml-kem-1024-sha384

### ml-kem-512-sha256
ml-kem-512-sha256 defines the ml-kem-768 public key and ciphertext from the client and server respectively encoded as octet strings. The shared secret is decapsulated from teh ciphertext using the client post-quantum KEM private key as defined in [FIPS203].

The HASH function used in this key exchange [RFC4253] is SHA-256 [nist-sha2] [RFC6234]

### ml-kem-768-sha256
ml-kem-768-sha256 defines the ml-kem-768 public key and ciphertext from the client and server respectively encoded as octet strings. The shared secret is decapsulated from teh ciphertext using the client post-quantum KEM private key as defined in [FIPS203].

The HASH function used in this key exchange [RFC4253] is SHA-256 [nist-sha2] [RFC6234]

### ml-kem-1024-sha384
ml-kem-1024-sha384 defines the ml-kem-768 public key and ciphertext from the client and server respectively encoded as octet strings. The shared secret is decapsulated from teh ciphertext using the client post-quantum KEM private key as defined in [FIPS203].

The HASH function used in this key exchange [RFC4253] is SHA-384 [nist-sha2] [RFC6234]


# Security Considerations

The security of ML-KEM is based on the presumed difficulty of solving the Module Learning With Errors (MLWE) problem, based on the computational problems in module lattices. [FIPS203]

# IANA Considerations

IANA is requested to register new method names "ml-kem-512-sha256", "ml-kem-768-sha256", "ml-kem-1024-sha384", and to be registered by IANA in the "Key Exchange Method Names" registry for SSH [IANA-SSH] with a reference field to this RFC and the "OK to implment" field of "May".

--- back

# Acknowledgments
{:numbered="false"}

Open Quantum Safe has an existing implementation of ML-KEM based key encapsulation methods in all three parameter variants. Their fork of OpenSSH contains an implementation using these algorithms for SSH key exchange algorithms.
