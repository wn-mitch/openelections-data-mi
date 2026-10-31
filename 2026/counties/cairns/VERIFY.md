Verifying a Cairns snapshot bundle
==================================

This directory holds a snapshot bundle: a CAR file named after the snapshot
record, and this file. The bundle holds everything needed to recompute the
snapshot's reading without contacting any server.

Quick check
-----------

With the `cairns` CLI built from the commit named in the snapshot's
`readingImpl`:

    cairns bundle verify <rkey>.car

It prints the snapshot, the number of records it proved, and whether the
recomputed `inputRoot` and the `sites`, `gaps`, `output` and `credit` blobs
match the snapshot record. Any mismatch is an error.

What the check does
-------------------

1. Every block in the CAR must hash (sha256) to its CID. Blocks are DAG-CBOR
   or raw.

2. The CAR's root is a manifest (DAG-CBOR, version `cairns/bundle/v1`). It
   lists every record the bundle proves, with the commit that holds it.

3. For each record, the commit must be signed by the `#atproto` key in its
   author's DID document, which the manifest includes, and the commit's
   Merkle search tree must map `<collection>/<rkey>` to the record's CID.
   The DID documents are as fetched when the bundle was built. Checking
   that the key was the author's at commit time needs the DID's history
   (for did:plc, `https://plc.directory/<did>/log/audit`).

4. The snapshot's `seenThrough` record and every `seen` record before it must
   form a chain from round 0: each names the one before it at its CID and
   starts where it ended. Each leaf document must hash to the value its
   record names and its leaves must match `firstSeenRoot`.

5. Each input record must appear in a window at its CID. A record first seen
   in window (a, b] counts as seen in the epoch that holds round b, and an
   author's first epoch is the earliest epoch of any leaf it authored.

6. The reading runs on those inputs with the manifest's beacons, each of
   which must verify against the drand quicknet key. Its `inputRoot` and
   blobs must equal the snapshot record's.

What the bundle leaves out
--------------------------

Source files are not in the bundle. Each source record names its content's
hash and URLs, so fetch the file and compare the hash. Check modules named
by the project or policy are included as raw blocks, and their CIDs must
match the check records. Lexicon JSON is included as raw blocks.

What it can't show
------------------

A bundle proves that these records existed, were dated, and produce this
reading. It can't prove that no other record belonged in the reading: an
indexer chose the inputs. Compare against the `seen` leaves, which list every
record the timekeeper's indexer saw, to look for records left out.
