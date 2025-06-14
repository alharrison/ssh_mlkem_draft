---
title: "Module-Lattice Key Exchange in SSH"
abbrev: "TODO - Abbreviation"
category: std

docname: draft-harrison-sshm-mlkem-00
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
 -
    fullname: "Andrew Benhase"
    organization: Cisco
    email: "abenhase@cisco.com"
 -
    fullname: "Panos Kampanakis"
    organization: AWS
    email: "kpanos@amazon.com"

normative:
  RFC2119:
  RFC4251:
  RFC4253:
  FIPS203:

informative:
  RFC6234:
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
  I-D.draft-ietf-sshm-mlkem-hybrid-kex:
    -:
    target: https://datatracker.ietf.org/doc/draft-ietf-sshm-mlkem-hybrid-kex/
    title: PQ/T Hybrid Key Exchange in SSH
    author:
      -
        ins: P. Kampanakis
        name: Panos Kampanakis
      -
        ins: D. Stabila
        name: Douglas Stebila
      -
        ins: T. Hansen
        name: Torben Hansen
        

--- abstract

This document defines pure post-quantum key exchange methods based on Module-lattice post-quantum key encapsulation schemes for use in the SSH Transport Layer Protocol.

--- middle

# Introduction

Secure Shell (SSH) [RFC4251] is a secure remote login protocol. The key exchange protocol described in [RFC4253] supports an extensible set of methods. The security of traditional key exchange methods used in Secure Shell (SSH) [RFC4251] relies on the algorithms being too computationally complex to be broken. The development of quantum computers poses a threat to the complexity of these algorithms. Given sufficiently powerful quantum computers, these traditional algorithms would be vulnerable to attack. Additionally, the threat of "harvest-now-decrypt-later" attacks could creates a risk in the current landscape before sufficiently powerful quantum computers are available. In this attack, the data would be collected and decrypted by these quantum computers at a later date.

This document addresses the problem by proposing the use of post-quantum key encapsulation mechanisms (KEMs) to extend the SSH [RFC4253] key exchange. [I-D.draft-ietf-sshm-mlkem-hybrid-kex] introduces ML-KEM in PQ/T Hybrid mode [draft-ietf-pquip-pqt-hybrid-terminology] which combines the shared secrets established by an ECDH and a ML-KEM key exchange. This document uses ML-KEM in a single-algorithm scheme without combining it with a traditional ECDH exchange. 

In the context of the [NIST_PQ], key exchange algorithms are formulated as key encapsulation mechanisms (KEMs), which consist of three algorithms:

   *  'KeyGen() -> (pk, sk)': A probabilistic key generation algorithm, which generates a public key 'pk' and a secret key 'sk'.

   *  'Encaps(pk) -> (ct, ss)': A probabilistic encapsulation algorithm, which takes as input a public key 'pk' and outputs a ciphertext 'ct' and shared secret 'ss'.

   *  'Decaps(sk, ct) -> ss': A decapsulation algorithm, which takes as input a secret key 'sk' and ciphertext 'ct' and outputs a shared secret 'ss', or in some cases a distinguished error value.

The main security property for KEMs is indistinguishability under adaptive chosen ciphertext attack (IND-CCA2), which means that shared secret values should be indistinguishable from random strings even
given the ability to have arbitrary ciphertexts decapsulated.  IND-CCA2 corresponds to security against an active attacker, and the public key / secret key pair can be treated as a long-term key or reused.  A weaker security notion is indistinguishability under chosen plaintext attack (IND-CPA), which means that the shared secret values should be indistinguishable from random strings given a copy of the public key.  IND-CPA roughly corresponds to security against a passive attacker, and sometimes corresponds to one-time key exchange.

The post-quantum KEM discussed in this document is ML-KEM which is based on CRYSTALS-KYBER. [FIPS203] standardized the ML-KEM scheme in 2024 with three parameter variants, ML-KEM-512, ML-KEM-768, ML-KEM-1024. ML-KEM is a NIST approved mechanism that is believed to be secure against an attacker with a quantum computer.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

## ML-KEM Key Exchange Message Numbers
When using ML-KEM as the Key Exchange Method, the following private namespace message numbers are defined in this document:
    #define SSH_MSG_KEX_KEM_INIT               30
    #define SSH_MSG_KEX_KEM_REPLY              31

# Key Exchange Method: ML-KEM
The client sends SSH_MSG_KEX_KEM_INIT. With this, the client sends C_INIT which is the ephemeral client ML-KEM public key, C_PK. C_PK represents the 'pk' output of the post-quantum KEM's 'KeyGen' at the client.

The server sends SSH_MSG_KEX_KEM_REPLY. With this, the server sends S_REPLY which is the concatenation of S_CT. S_CT is the ciphertext 'ct' output of the 'Encaps' algorithm generated by the server which encapsulates a secret to the client public key C_PK. Before producing S_CT, the server MUST perform the encapsulation key checks defined in Section 6.2 of [FIPS203], and abort using a disconnect message (SSH_MSG_DISCONNECT) with a SSH_DISCONNECT_KEY_EXCHANGE_FAILED as the reason, if they fail.

C_PK and S_CT are used to establish the shared secret, K_PQ. K_PQ is the post-quantum shared secret decapsulated from S_CT. Before decapsulating, the client MUST check if the ciphertext S_CT length matches the selected ML-KEM variant. The client MUST abort using a disconnect message (SSH_MSG_DISCONNECT) with a SSH_DISCONNECT_KEY_EXCHANGE_FAILED as the reason if the S_CT length does not match the ML-KEM variant or decapsulation fails for any other reason.

The derivation of encryption keys is done from the shared secret K_PQ according to Section 7.2 in [RFC4253] with a modification on the exchange hash H. The hash H is the result of computing the HASH, where HASH is the hash algorithm specified in the named key exchange method name, over the concatenation of the following 

      string V_C, client identification string (CR and LF excluded)
      string V_S, server identification string (CR and LF excluded)
      string I_C, payload of the client's SSH_MSG_KEXINIT
      string I_S, payload of the server's SSH_MSG_KEXINIT
      string K_S, server's public host key
      string C_INIT, client message octet string
      string S_REPLY, server message octet string
      string K_PQ, SSH ML-KEM shared secret




## ML-KEM Key Exchange Method Names
The ML-KEM key exchange method names defined in this document (to be used in SSH_MSG_KEXINIT [RFC4253]) are

    mlkem512-sha256
    mlkem768-sha256
    mlkem1024-sha384

Below we define 

### mlkem512-sha256
mlkem512-sha256 defines the ml-kem-512 C_PK public key and ciphertext S_CT from the client and server respectively which are encoded as octet strings. The K_PQ shared secret is decapsulated from the ciphertext S_CT using the client post-quantum KEM private key as defined in [FIPS203]. K_PQ is encoded as mpint [RFC4251].

The HASH function used in this key exchange [RFC4253] is SHA-256 [nist-sha2] [RFC6234]

### mlkem768-sha256
mlkem768-sha256 defines the ml-kem-768 C_PK public key and ciphertext S_CT from the client and server respectively which are encoded as octet strings. The K_PQ shared secret is decapsulated from the ciphertext S_CT using the client post-quantum KEM private key as defined in [FIPS203]. K_PQ is encoded as mpint [RFC4251].

The HASH function used in this key exchange [RFC4253] is SHA-256 [nist-sha2] [RFC6234]

### mlkem1024-sha384
mlkem1024-sha384 defines the ml-kem-1024 C_PK public key and ciphertext S_CT from the client and server respectively which are encoded as octet strings. The K_PQ shared secret is decapsulated from the ciphertext S_CT using the client post-quantum KEM private key as defined in [FIPS203]. K_PQ is encoded as mpint [RFC4251].

The HASH function used in this key exchange [RFC4253] is SHA-384 [nist-sha2] [RFC6234]


# Security Considerations

The security of ML-KEM is based on the presumed difficulty of solving the Module Learning With Errors (MLWE) problem, based on the computational problems in module lattices. [FIPS203]

# IANA Considerations

IANA is requested to register new method names "mlkem512-sha256", "mlkem768-sha256", "mlkem1024-sha384", and to be registered by IANA in the "Key Exchange Method Names" registry for SSH [IANA-SSH] with a reference field to this RFC and the "OK to implement" field of "MAY".

--- back

# Acknowledgments
{:numbered="false"}

Open Quantum Safe has an existing implementation of ML-KEM based key encapsulation methods in all three parameter variants. Their fork of OpenSSH (OQS-SSH) contains an implementation using these algorithms for SSH key exchange algorithms. The authors would like thank Open Quantum Safe for their example implementations of postquantum algorithms.
