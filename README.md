# yarn-plugin-nixify

**WORK IN PROGRESS**

Generates a Nix expression to build a Yarn v2 project (not using zero-install).

## Usage

```sh
# Install the plugin.
yarn plugin import https://raw.githubusercontent.com/stephank/yarn-plugin-nixify/main/dist/yarn-plugin-nixify.js

# Run Yarn as usual.
yarn

# Build your project with Nix.
nix-build
```

Running `yarn` with this plugin enabled will generate two files:

- `yarn-project.nix`: This file is always overwritten, and contains a basic
  derivation for your project.

- `default.nix`: Only generated if it does not exist yet. This file is intended
  to be customized with any project-specific logic you need.

When building the basic derivation without any customization, it'll run `yarn`
inside your project, copy your project to `$out/libexec/<name>`, and setup
wrappers in `$out/bin` for any executables declared in the `bin` field of your
top-level `package.json`.

Some examples of how to customize your build from within `default.nix`:

```nix
{ pkgs ? import <nixpkgs> { } }:

let

  # Example of providing a different source.
  src = pkgs.lib.cleanSource ./.;

  project = pkgs.callPackage ./yarn-project.nix {

    # Example of selecting a specific version of Node.js.
    nodejs = pkgs.nodejs-14_x;

  } src;

in project.overrideAttrs (oldAttrs: {

  # If your top-level package.json doesn't set a name, you can set one here.
  name = "myproject";

  # Example of adding dependencies to the environment.
  # Native modules sometimes need these to build.
  buildInputs = oldAttrs.buildInputs ++ [ pkgs.python3 ];

  # Example of invoking a build step in your project.
  buildPhase = ''
    yarn build
  '';

})
```

## Hacking

```sh
# In this directory:
yarn
yarn build-dev

# In your test project:
yarn plugin import /path/to/yarn-plugin-nixify/dist/yarn-plugin-nixify.dev.js
```

(Alternatively, add a direct reference in `.yarnrc.yml`. This will likely only
work if the Nix sandbox is disabled.)
