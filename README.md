<!--
 ~ Copyright (c) 2022 Espresso Systems (espressosys.com)
 ~ This file is part of the Configurable Asset Privacy for Ethereum (CAPE) library.
 ~
 ~ This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.
 ~ This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.
 ~ You should have received a copy of the GNU General Public License along with this program. If not, see <https://www.gnu.org/licenses/>.
 -->

# Configurable Asset Privacy for Ethereum (CAPE)

CAPE is an application of the [Configurable Asset Privacy (CAP)
protocol](https://github.com/EspressoSystems/cap/blob/main/cap-specification.pdf)
which enables digital assets to have customized privacy
properties. CAPE is a smart contract application that asset creators
can use to bring new assets with custom privacy into existence and to
generate CAPE versions of existing Ethereum assets, endowing ERC-20s
and eventually ERC-721s with privacy properties.

The [Cape Technical
Documentation](https://docs.cape.tech/espresso-systems/cape-technical-documentation/introduction)
describes the project at a high level.

**DISCLAIMER:** This software is provided "as is" and its security has not been externally audited. Use at your own risk.

---

<!-- run `md-toc` inside the nix-shell to generate the table of contents -->

**Table of Contents**

- [Configurable Asset Privacy for Ethereum (CAPE)](#configurable-asset-privacy-for-ethereum-cape)
  - [Obtaining the source code](#obtaining-the-source-code)
  - [Providing feedback](#providing-feedback)
- [Documentation](#documentation)
  - [CAP protocol specification](#cap-protocol-specification)
  - [CAPE Contract specification](#cape-contract-specification)
- [Environment](#environment)
  - [1. Install nix](#1-install-nix)
  - [2. Activate the nix environment](#2-activate-the-nix-environment)
  - [3. Verify installation](#3-verify-installation)
  - [4. direnv (Optional, but recommended for development)](#4-direnv-optional-but-recommended-for-development)
- [Build](#build)
  - [Static build](#static-build)
  - [Docker images](#docker-images)
- [CAPE Demo](#cape-demo)
  - [Local demo](#local-demo)
  - [Docker compose](#docker-compose)
- [Development](#development)
  - [Running Tests](#running-tests)
  - [Interactive development](#interactive-development)
  - [Linting & Formatting](#linting--formatting)
  - [Updating dependencies](#updating-dependencies)
  - [Rust specific development notes](#rust-specific-development-notes)
  - [Ethereum contracts](#ethereum-contracts)
  - [Testing against go-ethereum node](#testing-against-go-ethereum-node)
  - [Testing against hardhat node](#testing-against-hardhat-node)
    - [Separate hardhat node](#separate-hardhat-node)
    - [Hardhat node integrated in test command](#hardhat-node-integrated-in-test-command)
  - [Precompiled solidity binaries](#precompiled-solidity-binaries)
    - [Details about solidity compiler (solc) management](#details-about-solidity-compiler-solc-management)
  - [Alternative nix installation methods](#alternative-nix-installation-methods)
    - [Nix on debian/ubuntu](#nix-on-debianubuntu)
    - [Nix on Mac OS](#nix-on-mac-os)
  - [Git hooks](#git-hooks)
  - [Ethereum key management](#ethereum-key-management)
  - [Python tools](#python-tools)
  - [Gas Usage](#gas-usage)
    - [Gas Reporter](#gas-reporter)
    - [Gas usage of block submissions](#gas-usage-of-block-submissions)
  - [CI Tests](#ci-tests)
    - [Nightly CI builds](#nightly-ci-builds)
- [Deployment](#deployment)
  - [Linking to deployed contracts](#linking-to-deployed-contracts)
  - [Etherscan verification](#etherscan-verification)
  - [Official assets](#official-assets)
  - [Testnets](#testnets)
    - [Goerli](#goerli)
    - [Running the smoke tests](#running-the-smoke-tests)
- [Arbitrum on Goerli](#arbitrum-on-goerli)

## Obtaining the source code

    git clone git@github.com:EspressoSystems/cape.git

## Providing feedback

Feedback is welcome and can be provided by [creating a ticket](https://github.com/EspressoSystems/cape/issues/new).

# Documentation

## CAP protocol specification

A formal specification of the Configurable Asset Policy protocol can be found at [our CAP github repo](https://github.com/EspressoSystems/cap/blob/main/cap-specification.pdf)

## CAPE Contract specification

A specification of the CAPE _smart contract logic_ written in Rust can be found at `./doc/workflow/lib.rs`.

Extracting _API documentation_ from the solidity source is done using a javascript
tool called `solidity-docgen`.

To generate the documentation run

    make-doc

and observe the CLI output.

# Environment

This project has a lot of dependencies. The only tested installation method is
via the [nix](https://nixos.org) package manager.

## 1. Install nix

Installation instructions can be found [here](https://nixos.org/download.html).
If in a rush, running the following command and following the on-screen
instructions should work in most cases

    curl -L https://nixos.org/nix/install | sh

If the install script fails, it may be because the usage of `curl` in
the script has incorrect arguments. You may need to change

    fetch() { curl -L "$1" > "$2"; }

to

    fetch() { curl -L "$1" -o "$2"; }

Once the install script has failed, it may be necessary to manually
remove nix before trying again. See [Uninstallation](#uninstallation) below.

Some linux distros (ubuntu, arch, ...) have packaged `nix`. See the section
[Alternative nix installation methods](#alternative-nix-installation-methods)
for more information.

## 2. Activate the nix environment

To activate a shell with the development environment run

    nix-shell

from within the top-level directory of the repo.

Note: for the remainder of this README it is necessary that this environment is
active.

Once the `nix-shell` is activated the dependencies as well as the scripts in the
`./bin` directory will be in the `PATH`.

## 3. Verify installation

Try running some tests to verify the installation

    cape-test-geth

If this fails with errors that don't point to obvious problems please open an
issue on github. M1 Macs need to have node@16 installed to avoid memory allocation errors.

Note that these tests use `cargo test --release` which is slower for compiling but then faster for executing.

## 4. direnv (Optional, but recommended for development)

To avoid manually activating the nix shell each time the
[direnv](https://direnv.net/) shell extension can be used to activate the
environment when entering the local directory of this repo. Note that direnv
needs to be [hooked](https://direnv.net/docs/hook.html) into the shell to
function.

To enable `direnv` run

    direnv allow

from the root directory of this repo.

When developing `nix` related code it can sometimes be handy to take direnv out
of the equation: to temporarily disable `direnv` and manually enter a nix shell
run

    direnv deny
    nix-shell

# Build

To build the project run

    cargo build --release

The `--release` flag is recommended because without it many cryptographic
computations the project relies one become unbearably slow.

## Static build

To build a statically linked version of the project with musl64 as a libc on a `x86_64-linux-unknown-gnu` host:

```bash
nix develop .#staticShell -c cargo build --release
```

The resulting build artifacts end up in `target/x86_64-unknown-linux-musl`, and may be run on any linux computer as they will not depend on glibc or any shared libs (`*.so`).

## Docker images

To build the wallet or services Docker images locally run

    build-docker-wallet

or

    build-docker-services

inside a nix shell from the root directory of the repo.

For the CI build see the `docker-*` jobs in
[.github/workflows/build.yml](.github/workflows/build.yml).

# CAPE Demo

## Local demo

To run the CAPE demo locally, run

    cape-demo-local

## Docker compose

To run the docker compose demo, run

    cape-demo-docker

The `CAPE_SERVICES_IMAGE` or `CAPE_WALLET_IMAGE` env vars can be set to run the
demo with the locally built docker images:

    env CAPE_SERVICES_IMAGE=cape/services CAPE_WALLET_IMAGE=cape/wallet cape-demo-docker

# Development

## Running Tests

An running ethereum node is needed to run the tests. We support running against
a go-ethereum (`geth`) or hardhat node running on `localhost:8545`.

The simplest way to run all the tests against a nodes is to use the scripts

    cape-test-geth
    cape-test-hardhat

These scripts will

1. Start a corresponding node if nothing is found to be listening on the
   configured port (default: `8545`).
2. Run the tests.
3. Shut down the node (if it was started in 1.)

Note that running `js` tests against the `hardhat node` may take several
minutes.

The port of the node can be changed with `RPC_PORT`. For example,

    env RPC_PORT=8877 cape-test-geth

To run all the tests against both nodes

    cape-test-all

## Interactive development

To start the background services to support interactive development run the command

    hivemind

For the time being this is a `geth` node and a contract compilation watcher.

To add additional processes add lines to `Procfile` and (if desired) scripts to
run in the `./bin` directory.

## Linting & Formatting

Lint the code using all formatters and linters in one shot

    lint-fix

## Updating dependencies

Run `nix flake update` if you would like to pin other version edit `flake.nix`
beforehand. Commit the lock file when happy.

To update only a single input specify it as argument, for example

    nix flake update github:oxalica/rust-overlay

To make use of newly released `solc` version run

    cd nix/solc-bin
    ./update-sources.sh

## Rust specific development notes

To run only the rust tests run

    cargo test --release

Note that this requires compiled solidity contracts and a running geth node. For
development it's convenient to keep `hivemind` running in a separate terminal
for that purpose.

To connect to various chains use `RPC_URL` and `MNEMONIC` env vars. For example

    env MNEMONIC="$RINKEBY_MNEMONIC" RPC_URL=$RINKEBY_URL cargo test

To watch the rust files and compile on changes

    cargo watch

The command (`check` by default) can be changed with `-x` (for example `cargo watch -x test`).

## Ethereum contracts

To compile the contracts to extract the abi run the following command from the
root of the `cape` repo checkout:

    build-abi

Note: structs will only be included in the ABI if there is a public function
that uses them.

Instead of running `geth` and `build-abi` one can also run

    hivemind

From the root directory of the repo checkout. This will watch and recompile the
contracts when there are changes to any of the contract files.

To recompile all contracts

    hardhat compile --force

When removing or renaming contracts it can be useful to first remove the
artifacts directory and the compilation cache with

    hardhat clean

## Testing against go-ethereum node

Start the geth chain in separate terminal

    run-geth

When running tests against geth

- Tests run faster than with the hardhat node.
- The `console.log` statements in solidity **do nothing** (except consume a tiny amount of gas (?)).
- Failing `require` statements are shown in the`geth` console log.

The genesis block is generated with the python script `bin/make-genesis-block`.

If time permits replacing the `run-geth` bash script with a python script that
uses `make-genesis-block` and `hdwallet-derive` could be useful.

Note: when making calls (not transactions) to the go-ethereum node,
`msg.sender` is the zero address.

## Testing against hardhat node

You can choose to let hardhat start a hardhat node automatically or start a node
yourself and let hardhat connect to it.

Note: when making calls (not transactions) to the hardhat node, `msg.sender` is
set by the hardhat node to be the first address in `hardhat accounts`. Since
this is different from the behaviour we see with go-ethereum this can lead to
confusion for example when switching from go-ethereum to hardhat to debug.

### Separate hardhat node

Start the hardhat node in separate terminal

    hardhat node --network hardhat

When running tests against this hardhat node

- Tests are slow.
- The `console.log` statements in solidity show in terminal running the node.
- Failing `require` statements are shown in human readable form in the terminal running the node.

### Hardhat node integrated in test command

It's also possible to run the hardhat node and tests in one command

    hardhat --network hardhat test

- Tests are slow.
- The `console.log` statements are shown in in the terminal.
- Failing `require` statements are shown in human readable form in the terminal.

## Precompiled solidity binaries

Hardhat is configured to use the solc binary installed with nix (see
[nix/solc-bin/default.nix](nix/solc-bin/default.nix)) if matches the version
number. If hardhat downloads and uses another binary a warning is printed the
console.

### Details about solidity compiler (solc) management

The binaries used by hardhat, brownie, solc-select, ... from
https://solc-bin.ethereum.org/linux-amd64/list.json underwent a change in build
process from v0.7.5 to v0.7.6 ([this
commit](https://github.com/ethereum/solidity/commit/7308abc08475869cf7bc6a0654acb9d45bafc52a)).

The new solc binaries either fail to run or depend on files that aren't provide by nix.

Hardhat always "works" because it falls back to solcjs silently (unless running with `--verbose`)

    $ hardhat compile --verbose
    ...
    hardhat:core:tasks:compile Native solc binary doesn't work, using solcjs instead +8ms
    ...

The solcjs compiler is a lot slower than native solc and brownie (using py-solc-x),
solc-select do not have such a fallback mechanisms.

## Alternative nix installation methods

### Nix on debian/ubuntu

#### Installation

To install and setup `nix` on debian/ubuntu using [their nix
package](https://packages.debian.org/sid/nix-setup-systemd). The steps below
were tested on ubuntu 20.10.

    sudo apt install nix
    sudo usermod -a -G nix-users $USER # logout and login
    nix-channel --add https://nixos.org/channels/nixos-21.05 nixpkgs
    nix-channel --update
    source /usr/share/doc/nix-bin/examples/nix-profile.sh

The last line needs to be run once per session and is usually appended to
`.bashrc` or similar.

To test the installation, run

    $ nix-shell -p hello --run hello
    ...
    Hello, world!

#### Uninstallation

##### On linux

To remove `nix` manually,

    sudo apt purge --autoremove nix-bin nix-setup-systemd
    # Be careful with rm.
    rm -r ~/.nix-*
    sudo rm -r /nix
    # Reboot machine. (This step my not always be necessary.)

- Remove any lines added to `.bashrc` (or other files) during installation.
- If desired remove group `nix-users` and users `nixbld*` added by nix.

To remove `nix` with the aid of a script,

    # In the cape directory (where this README.md is)
    sudo bin/rmnix

### Nix on Mac OS

#### Installation

Run the following command:

```
sh <(curl -L https://nixos.org/nix/install)
```

See https://nixos.org/download.html#nix-install-macos

#### Uninstallation

This can be tricky on Mac OS.
Following the steps provided by the installation script above should work.

If however you run into https://github.com/NixOS/nix/issues/3900, rebooting the computer should enable you to delete the nix related directories.

## Git hooks

Pre-commit hooks are managed by nix. Edit [flake.nix](flake.nix) to manage the
hooks.

## Ethereum key management

The keys are derived from the mnemonic in the `TEST_MNEMONIC` env var.

- Hardhat has builtin mnemonic support.
- For geth we start an ephemeral chain but specify a `--keystore` location and
  load the addresses into it inside the `bin/run-geth` script.
- A simple python utility to derive keys can be found in `bin/hdwallet-derive`.
  For description of the arguments run `hdwallet-derive --help`.

## Python tools

We are using `poetry` for python dependencies and `poetry2nix` to integrate them
in the nix-shell development environment.

Use any poetry command e. g. `poetry add --dev ipython` to add packages.

## Gas Usage

### Gas Reporter

Set the env var `REPORT_GAS` to get extra output about the gas consumption of
contract functions called in the tests.

    env REPORT_GAS=1 hardhat test

### Gas usage of block submissions

Run

    cargo run --release --bin gas-usage

To show gas usage of sending various notes to the contract.

## CI Tests

You can replicate the same set of tests that the CI system does by running this command in your nix-shell

    run-ci-tests

### Nightly CI builds

There's a CI nightly job that runs the test suite via hardhat against the Rinkeby testnet.

2 things to note:

1. Currently the relevant contracts are deployed with each build, meaning we're not optimizing on gas costs per build.

2. The funds in the testnet wallet need to be topped off occassionally as the nightly tests consume gas over some period of time.

Based on a few successful test runs, the entire suite should consume roughly 0.079024934 gas. In case the wallet needs more funds, send more test ETH to:

```
0x2FB18F4b4519a5fc792cb6508C6505675BA659E9
```

# Deployment

The CAPE contract can be deployed with

    hardhat deploy

The deployments are saved in `contracts/deployments`. If you deploy to localhost
you have to use `hardhat deploy --reset` after you restart the geth node in
order to re-deploy.

## Linking to deployed contracts

In order to avoid re-deploying the library contracts for each test you can pass
the address obtained by running `hardhat deploy` as an env var

    env RESCUE_LIB_ADDRESS=0x5FbDB2315678afecb367f032d93F642f64180aa3 cargo test --release

## Etherscan verification

After running `hardhat deploy`, run the [etherscan-verify](./bin/etherscan-verify) script. For example

    hardhat deploy --network goerli
    etherscan-verify goerli

This requires the `ETHERSCAN_API_KEY` env var to be set. Keys can be found at
https://etherscan.io/myapikey.

## Official assets

CAPE can be deployed with an official asset library: a set of assets, supported by Espresso,
registered in the on-chain state, and collected into a signed asset library file which can be
distributed with the wallet. To generate the asset library for a deployment, make sure all of the
services are up and the following environment variables are set, indicating the deployment to
connect to:

- CAPE_EQS_URL
- CAPE_RELAYER_URL
- CAPE_ADDRESS_BOOK_URL
- CAPE_FAUCET_URL
- CAPE_CONTRACT_ADDRESS
- CAPE_WEB3_PROVIDER_URL

You should also set the following environment variables which allow the official asset library
generator to authenticate itself in various ways:

- CAPE_ASSET_LIBRARY_SIGNING_KEY: the signing key pair to use to authenticate the generated asset
  library.
- CAPE_ASSET_LIBRARY_CAPE_MNEMONIC: the mnemonic phrase to generate the CAPE wallet used to
  generate the assets. This is the wallet which will receive the minted domestic assets, so this
  mnemonic can be used to distribute minted domestic assets.
- CAPE_ASSET_LIBRARY_ETH_MNEMONIC: the mnemonic phrase to generate the Ethereum wallet used to
  deploy the assets. The first address generated by this mnemonic must be funded with enough ETH
  (on the chain where you're deploying) to pay the gas fees for sponsoring all of the wrapped
  assets in the official asset library.

You must also have a TOML file specifying the assets to create. There are already TOML files for
the official asset libraries for

- [the Goerli deployment](wallet/official_assets/cape_v1_official_assets.toml)
- [the local demo](wallet/official_assets/cape_demo_local_official_assets.toml)
- [the Arbitrum goerli deployment](wallet/official_assets/cape_v2_official_assets.toml)

These can always be modified or copied to create fully customizable asset
libraries.

Then, the command would look like

```
cargo run --release --bin gen-official-asset-library -- --assets /path/to/toml/file -o output
```

The assets will be deployed and the signed asset library written to `output`. To include the output
library in a distribution of the wallet GUI, it must make its way into
`$CAPE_WALLET_STORAGE/verified_assets` when the GUI is started. The [Goerli asset library](wallet/official_assets/cape_v1_official_assets.lib)
is automatically copied into `$CAPE_WALLET_STORAGE/verified_assets` whenever the wallet API Docker
container is built.

## Testnets

### Goerli

- Faucets to get Goerli Ether: [Simple](https://goerli-faucet.slock.it),
  [Social](https://faucet.goerli.mudit.blog/),[Goerli PoW Faucet](https://goerli-faucet.pk910.de/).
- Set the GOERLI_URL in the .env file. A project can be created at
  https://infura.io/dashboard/ethereum.
- Set the GOERLI_MNEMONIC (the Ethereum mnenmonic for your goerli wallet) in the
  .env file.

To deploy to goerli (if you get a `replacement fee too low` error, re-run it
until it exits without error):

    hardhat test --network goerli

The CAPE contract address is printed to the console right after deployment and
also saved in a file. To find it, run

```console
cat contracts/deployments/goerli/CAPE.json | jq -r '.address'
```

### Running the smoke tests

#### On the main testnet deployment

- Ensure the contract is deployed (e. g. `hardhat test --network goerli`) was run.
- Ensure _no_ transcations have been run on the CAPE contract after deployment.

Set the following environment variable in the .env file:

- `CAPE_FAUCET_MANAGER_MNEMONIC`: is the mnemonic for the Faucet manager's keys.

Then run

```bash
    export CAPE_CONTRACT_ADDRESS=$(cat contracts/deployments/goerli/CAPE.json | jq -r .address)
    env ETH_MNEMONIC="$GOERLI_MNEMONIC" CAPE_WEB3_PROVIDER_URL="$GOERLI_URL" cargo test --release -- smoke_tests --nocapture
```

#### With custom faucet manager mnemonic for testing

Run [smoke-test-goerli](./bin/smoke-test-goerli) with a custom mnemonic (here `TEST_MNEMONIC`):

```console
env MY_FAUCET_MANAGER_MNEMONIC="$TEST_MNEMONIC" smoke-test-goerli
```

If you get a `replacement fee too low` error, re-run the command up to 2 to 3 times.

To start from scratch and ignore existing deployments pass `--reset`:

```console
env MY_FAUCET_MANAGER_MNEMONIC="$TEST_MNEMONIC" smoke-test-goerli --reset
```

# Arbitrum on Goerli

To run the tests against Arbitrum Goerli follow these steps:

- Install [Metamask](https://metamask.io/) in your browser and restore with
  your GOERLI_MNEMONIC.
- Switch metamask to the Goerli network.
- Get some Goerli ethers at some faucet (see Goerli section above)
- Go to the [Arbitrum bridge](https://bridge.arbitrum.io/) and deposit your
  Goerli eth. Leave a bit for the ethereum gas fees. Wait a few minutes until
  your account is funded.
- Deploy the contracts

```
hardhat deploy --tags CAPE --network arbitrum_goerli --reset
```

- Run the tests

```
run-tests-arbitrum
```

This script will re-use the stateless libraries in order to save gas and speed
up test execution. To run a single test, pass the test name to the script as
argument, e.g. `run-tests-arbitrum backend::test::test_transfer`.

If you are getting `Too Many Requests` errors, you may want to set up an infura
project and use their arbitrum goerli RPC and use it via

```
CAPE_WEB3_PROVIDER_URL=https://arbitrum-goerli.infura.io/v3/... run-tests-arbitrum
```

To deploy mintable tokens that users can mint by sending Ether to it run

```
hardhat deploy --tags Token --network arbitrum_goerli --reset
```

The currently deployed token contracts on arbitrum goerli are

```console
WETH 0x4F1D9E040cf28A522ec79951cDb7B55c8aE4744E
DAI 0xBeec50ed16E3559afCD582cC98ed2b5F5DcA189E
USDC 0x9A4f4Ee35a8FfEE459B3187A372d422790fc8aAB
```
