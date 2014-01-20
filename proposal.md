# Generalized Hashcash Header

## BNF

    <hashcash-header> ::= <profile> ":" <fixed-value> ":" <counter>
    <fixed-value> ::= <params> ":" <resource>

## Description

In general, profiles are expected to perform a hash calculation using a function specified by the profile, using parameters specified in the "params" component, over the value of the entire header, to generate a Hashcash value corresponding to the given resource.

Valuation and validation of the calculated result are left to the relevant interpretation of the profile and how Hashcash is being used, and may be relative to any out-of-header values (such as the body of a message or request).

## Fields and meanings

### profile

What algorithm is being used to generate the hashcash. This supersedes hashcash 0 and 1's "version" parameter.

### params

Any parameters to the hashcash function. For profiles "0" and "1", this refers to the number of bits at the beginning of the resulting SHA-1 function asserted to be 0; a variable-memory-and-calculation intensive function like bcrypt may include it

### resource

A unique identifier for the corresponding hashcash value. Any number of colons past the second and before the last become part of the resource name.

Things that may be part of a resource:

- sender address for email.
- date the stamp was calculated.
- hash of a message's body.
- a random identifier for calculating multiple discrete values.
- salt/nonce required by a challenge.
  - If a salt has a special functionality in the design of a hash function, it may be included as part of the "params" instead.

### counter

A random value used to generate the resulting hashcash value.

## Headers and Usages

In contrast to Hashcash versions 0 and 1 which used a generalized "X-Hashcash" header, Generalized Hashcash functions should use more specific headers that describe how to valuate and validate a corresponding resource, relative to the "resource" and "params" strings.

Generalized Hashcash headers begin with the name "Hashcash" rather than "X-Hashcash", in accordance with RFC6648 (BCP178).

### Old Stamps (X-Hashcash)

    <resource> ::= <date> ":" <old-resource> ":" <ext> ? ":" <rand>
    <ext> is ignored in both "0" and "1" and should be omitted.

Original Hashcash stamps were designed to be pre-computed ahead of time and then attached to a message, with "old-resource" in practice effectively being the sender's email address, and "rand" being used 

Today, this kind of pre-computed value mining is usually reserved for generalized cryptocurrencies like Bitcoin: pre-calculating values tied exclusively to your own address for the purpose of sending email would generally be considered a waste of resources.

Sending a pre-computed or paid value could be done with something like a "Bitcoin-Stamp" header, that ties a message to a transaction on the Bitcoin network. Specifying such a header would be outside the scope of this specification.

### New Stamps (Hashcash-Stamp)

    <resource> ::= <recipients> ":" <message-id> ":" <body-hash> 

This form of hashcash stamp is specifically tied to a message. "message-id" is the value of the "Message-ID" header. <body-hash> is the result of running the same function used to calculate the hashcash value on the body of the message. "recipients" includes all addresses from the "To:" and "Cc:" headers, and the Hashcash-Stamp should only be considered valid for those recipients.

### Challenge-Response

As yet unspecified: see [Hashcash-for-Node](https://github.com/base698/Hashcash-for-Node) for possible inspiration.

## Valuation and Validation

An email system like SpamAssassin may judge mail using a "bitcoin-stamp" header to have a value relative to its age and difficulty relative to the power of the Bitcoin network at the time the message was received, and use that valuation to adjust the calculated "spamminess" of a message by a number of other factors.

In contrast, a system using Hashcash to

## Profiles

These are some suggested names

### "0" | "1" | "sha1"

params: the number of bits at the beginning of the resulting SHA-1 function asserted to be 0.

### "bitcoin"

Uses the Bitcoin double-256 hashcash function, in case you want to use something like an obsolete ASIC chip to calculate a hashcash stamp for something other than a blockchain block. In this, "params" refers to difficulty as it works in Bitcoin.

### "sha3"

## Open Issues

- Should parameters used exclusively for V&V really be part of params?
  - Perhaps "params" should be included in a different way (eg. dollars-sign-seperated after the scheme name, as per /etc/shadow), and "resource" then controls all V&V-relevant info.

### Hashcash-Stamp

- How the hash of the body is determined is unspecified, as bodies are known to get mangled in mail sending.
- How "recipients" is calculated / concatenated is undefined. (Can email addresses contain colons? What happens if a recipient is added after a message is sent)
- Should the stamp include other values, such as the Subject, or Sender? What are the consequences of not including those values?