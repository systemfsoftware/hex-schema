# hex-schema

Effect Schema codecs for hexadecimal strings and bytes. Each schema decodes a hex string into one normalised form and encodes it back to its wire format: lowercase, `0x`-prefixed, colon-separated (the fingerprint form, such as `AB:CD:0F`), or a `Uint8Array`.

## Quick start

```ts
import { ColonHex, PrefixedHex, Uint8ArrayFromPrefixedHex } from '@systemfsoftware/hex-schema'
import { Schema as S } from 'effect'

S.decodeUnknownSync(PrefixedHex)('0xabcd') // 'abcd'
S.decodeUnknownSync(ColonHex)('ab:cd:0f') // 'abcd0f'
S.decodeUnknownSync(Uint8ArrayFromPrefixedHex)('0x0aff') // Uint8Array(2) [ 10, 255 ]
```

`PrefixedHex` rejects uppercase digits, so `S.decodeUnknownSync(PrefixedHex)('0xABcd')` throws.

## Install

`@systemfsoftware` packages are not published to npm. They come from nix.systemfsoftware.com, the package registry for `@systemfsoftware` packages.

1. Add this repository as a flake input:

   ```nix
   inputs.hex-schema.url = "github:systemfsoftware/hex-schema";
   ```

2. Substitute from the cache. This repository's `flake.nix` sets the same keys:

   ```nix
   nixConfig = {
     extra-substituters = "https://nix.systemfsoftware.com";
     extra-trusted-public-keys = "nix-cache-1:wXA8I4DayNW9q6enUyQA/P/nT38pGSTLt374t3gsn7M=";
   };
   ```

3. Take the package tarballs through the consumer route in [pnpm-release-management](https://github.com/systemfsoftware/pnpm-release-management). That repository provides `lib.mkProducerTarballs`, the call this repository's `flake.nix` makes for its own producers, and `lib.mkPnpmConsumerStore`.

4. Install the peer from npm. You bring `effect` yourself:

   ```bash
   pnpm add effect@^4
   ```

The repository publishes two packages. [`@systemfsoftware/hex-schema`](packages/hex-schema/README.md) holds the schemas. [`@systemfsoftware/effect-schema-extensions`](packages/effect-schema-extensions/README.md) re-exports everything in `@systemfsoftware/hex-schema`, from its root and from its `./hex-schema` subpath, so either one gives you the same names.

## Schemas

| Schema                      | Accepts                                                              | Decodes to           | Encodes to                 |
| --------------------------- | -------------------------------------------------------------------- | -------------------- | -------------------------- |
| `StrictHex`                 | lowercase hex, no prefix; empty allowed                              | the same string      | the same string            |
| `HexString`                 | optional `0x`; any case; any length                                  | lowercase, no prefix | the same string            |
| `PrefixedHex`               | `0x`, then lowercase hex; `0x` alone is empty                        | lowercase, no prefix | `0x`-prefixed              |
| `ColonHex`                  | 1 or 2 hex digits per byte, colon-separated, any case; empty allowed | lowercase, no colons | uppercase, colon-separated |
| `HexBytes`                  | hex string of even length, either case                               | `Uint8Array`         | lowercase hex              |
| `Uint8ArrayFromPrefixedHex` | `0x`, then lowercase hex of even length                              | `Uint8Array`         | `0x`-prefixed hex          |

`HexString`, `PrefixedHex` and `ColonHex` decode to branded strings, so a plain `string` does not type-check where one of them is required.

Two differences matter when you mix schemas:

- `HexString` accepts an odd number of digits, such as `abc`. The byte schemas, `HexBytes` and `Uint8ArrayFromPrefixedHex`, reject it.
- `HexString` and `HexBytes` accept uppercase digits. `PrefixedHex` and `Uint8ArrayFromPrefixedHex` reject them.

## Contributing

Development setup and workflow are in [CONTRIBUTING.md](CONTRIBUTING.md).

## Support

Report bugs and ask questions in the [issue tracker](https://github.com/systemfsoftware/hex-schema/issues).

## License

[Apache-2.0](packages/hex-schema/LICENSE)
