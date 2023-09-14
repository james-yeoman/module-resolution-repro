# What

Reproduction of an issue with Custom Conditions.

# Problem

While `BWIP-JS`'s export map is used by `vite` for module resolution (if you set `resolve.browserField`
to `false` like in [`app`'s vite config](./packages/app/vite.config.ts)), Typescript/`tsc` doesn't allow for
cascading custom conditions.

As an example of the usecases, my team has a rendering abstraction for shapes. It's kind of like `BWIP-JS`'s `DrawingContext` interface, since it lets us define label objects using shape primitives. For barcodes, we have a `DrawingContext` implementation that wraps our rendering abstraction.

The upside is that `BWIP-JS` doesn't need to know if we're rendering to an `SVG` or to a `PDF`. The downside is that `BWIP-JS` doesn't know if we're rendering to an `SVG` or to a `PDF`...

The determining factor for the `exports` map resolution is whether the rendering library is being imported in
the application for a preview (`SVG`), or in the print-time renderer on the backend (`PDF`).

This was all well and good until I decided to try to migrate our Typescript codebase from using
`"moduleResolution": "Node"` to `"moduleResolution": "Bundler"`.

The conditions that `tsc` uses for `exports` map resolution are `import` and `types`, when `moduleResolution`
is set to `"Bundler"`. When `moduleResolution` is set to `"Node"`, the `exports` map isn't even consulted.
Instead, the `main` and `browser` fields are used (which explains why it worked before changing
`moduleResolution`).

This made it impossible for Typescript to determine which type declaration file to resolve, which therefore
broke our linting.
