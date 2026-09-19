# Protobuf Wire Decoder

Paste raw protobuf bytes as hex or base64 and read the message without the .proto — field numbers, wire types, every plausible interpretation, nested messages, and a generated schema skeleton. Runs entirely in your browser.

**Live:** <https://protobuf-decoder.slippylabs.com/>

## What it does

- Decodes any protobuf message from hex, base64 or `\x` escapes — no schema needed, the same job as `protoc --decode_raw` without installing anything.
- Shows **every** interpretation of each field, because the wire format does not record which was meant: a varint is shown as uint64, int64, sint64 (zigzag) and bool; an 8-byte field as fixed64, sfixed64 and double.
- Recurses into length-delimited fields that parse as nested messages, and also offers them as strings, bytes and packed repeated fields — all four look identical on the wire.
- Handles groups (wire types 3 and 4), which predate nested messages and are still legal.
- Hover a field to light up the bytes it came from, and get a `.proto` skeleton inferred from the structure.

## How it works

Everything is `BigInt`: a uint64 does not fit in a double, and quietly rounding someone's ID is the one thing a tool like this must not do.

Two details worth knowing. **Re-encoding is a self-check**: the page re-encodes what it decoded with canonical minimal varints and compares byte for byte, which proves the decode covered the whole input and tells you when the original was not canonical. And **a negative int32 is sign-extended to 64 bits before encoding**, so `-1` costs ten bytes on the wire and reads as 18446744073709551615 if you take it as unsigned — there is a sample for exactly that.

## Verification

Checked against three independent references:

- **Google's own `protobuf` library** encodes real messages; every field must come back with the right number, wire type and value, including the negative-int32 case.
- **300 fuzzed messages** from an encoder written here from the spec, covering every wire type, nested messages, packed repeated fields and groups — compared field by field, and each must re-encode to itself byte for byte.
- **`blackboxprotobuf`**, a second schema-less decoder, must agree about the structure.

Plus varint edge cases (10-byte values, non-minimal encodings, truncation) and nine malformed messages that must be rejected rather than half-read.

## Run it locally

A static site. No build step, no package manager, no dependencies:

```
git clone git@github.com:slippylabs/protobuf-decoder.slippylabs.com.git
cd protobuf-decoder.slippylabs.com
python3 -m http.server 8000
```

Then open <http://localhost:8000>.

---

Part of [Slippy Labs](https://slippylabs.com). Every tool is indexed at
[projects.slippylabs.com](https://projects.slippylabs.com).
