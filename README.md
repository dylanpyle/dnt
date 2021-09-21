# d2n

Prototype for a Deno to Node/canonical TypeScript CLI tool.

This will output tsc compatible code from a Deno codebase that could then be sent to a bundler (or compiled by tsc) for npm distribution.

Example:

```bash
# clone a Deno-first repo
git clone https://github.com/dsherret/code-block-writer.git

# clone this repo
git clone https://github.com/dsherret/d2n.git

# run tool and output to ./code-block-writer/npm
cd d2n
cargo run -- ../code-block-writer/mod.ts --out ../code-block-writer/npm

# go to output directory, run tsc, and publish
cd ../code-block-writer/npm
tsc mod.ts --target ES2015 --module commonjs --declaration
npm publish
```

The file in *code-block-writer/npm/mod.ts* would contain module specifiers without extensions (in the main codebase they had `.ts` extensions):

```ts
import { CommentChar } from "./comment_char";
import { escapeForWithinString, getStringFromStrOrFunc } from "./utils/string_utils";
```

If you run with `cargo run -- ../code-block-writer/mod.ts --out ../code-block-writer/npm --keep-extensions` it will then contain:

```ts
import { CommentChar } from "./comment_char.js";
import { escapeForWithinString, getStringFromStrOrFunc } from "./utils/string_utils.js";
```

## Future Goals

1. Programmatic API available via Rust and Wasm (this would be used by bundlers to avoid writing the intermediary files to the disk).
1. Support Deno.json to get compiler options.
1. Handle mapping from remote specifiers to bare specifiers and transforming them in the file.
1. Handle dynamic imports (at least ones that are statically analyzable and maybe warn on others)
1. Support creating or modifying a package.json and using that for publish.

Notes from Kitson:

- We would need to rewrite triple slash references
- We might need to deal with the types in the tsconfig.json
- How do we cleanly supply a deno.ns lib so type checking works?
  - David: We will search for any Deno specific APIs and replace them with a node shim. To start, we can just import everything as long as there's not any conflicts.
- How do we handle remote URLs, data URLs and blob dynamic imports?
  - David: Just changed it to download everything for now, but in the future we can implement remote URL -> bare specifier mapping. Ideally this will be automatic, but in some cases the user will need to specify a bare specifier to use.
  - David: We could probably output data URLs to a file.
  - David: Blob dynamic imports... I'm not sure. Dynamic imports will be a problem if they're not statically analyzable, but we can warn the user about that when it happens.
- We should go from ./foo.ts to ./foo.js by default, with a flag to go from ./foo.ts to ./foo, assume people are supporting a browser or ESM Node.js
  - David: I'll change this to be the default later.
