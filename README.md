#SRP - Secure Remote Password

Implementation of the [SRP Authentication and Key Exchange System](http://tools.ietf.org/html/rfc2945).

http://tools.ietf.org/html/rfc5054

Work in progress.

Want to support:

- [done] srp function library that passes http://tools.ietf.org/html/rfc5054#appendix-B tests
- srp server
- javascript browser client

##Procedure

Carol the Client wants to share messages with Steve the server.

Carol and Steve agree on a large random number `N` and a generator
`g`.  These can be published in advance or better yet hard-coded in
their implementations.  They also agree on a cryptographic hashing
function `H`.

First, Steve generates some random salt, `s`.  Carol establishes a
password `P`.  Carol computes the verifier `v` as `g ^ H(s | H(I | ':'
| P)) % N`, where `I` is Carol's identity, and `|` denotes
concatenation.

Steve stores `I`, `s`, and `v`.  Carol remembers `p`.  This sequence
is performed once, after which Carol and Steve can use the SRP
protocol to share messages.

##Message Flow

First, Carol generates an ephemeral private key `a`.  She computes the
public key `A` as `g^a % N`.  She sends Steve `I` and `A`.

1. Client sends `I`, `A`

Steve looks up `v` and `s`.  Steve generates an ephemeral private key
`b` and computes the public key `B` as `k * v + g^b % N`, where `k` is
`H(PAD(g))`.  (`PAD` designates a function that left-pads a byte
string with zeroes until it is the same size as `N`.)  Steve sends `s`
and `B`.

2. Server replies with `s` and `B`

Both now compute the scrambling parameter `u` as `u = H(PAD(A) | PAD(B))`.

Now both Carol and Steve have the parameters they need to compute
their session key, `S`.

For Carol, the formula is:

```
S_client = (B - k * g^x) ^ (a + u * x)
```

For Steve, the formula is: 

```
S_server = (A * v ^ u) ^ b
```

They both now compute the shared session key, `K`, as `H(S)`.  (The
hash is taken to obscure any structure that may be visible in `S`.)

Now Carol and Steve must convince each other that their values for `K`
match.

##License

MIT