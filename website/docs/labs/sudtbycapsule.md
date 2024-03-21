---
id: sudtbycapsule
title: Write an SUDT Script Using Capsule
---

## Introduction

[Capsule](https://github.com/nervosnetwork/capsule) is a set of tools for Rust developers to develop scripts (smart contracts) on CKB. Capsule covers the entire lifecycle of script development, including writing, debugging, testing, and deployment.

In this tutorial, you will learn how to write a SUDT script using Capsule. SUDT is the abbreviation of Simple User Defined Token, a minimalist standard for dapp developers to issue custom tokens on Nervos CKB. You can refer to the [SUDT RFC](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0025-simple-udt/0025-simple-udt.md) for more details.

In order to complete this tutorial:

* You need to be proficient in software development using Rust.
* You should be generally familiar with Nervos CKB.
* You should be open to learning about the bleeding edge of blockchain development.

In this tutorial you will:

* Setup your environment.
* Write an SUDT script.
* Write tests for your script.
* Deploy your script to the testnet.

If you run into an issues while working on this tutorial, feel free to contact us on [Nervos Talk](https://talk.nervos.org/) or [Discord](https://discord.gg/FKh8Zzvwqa).

## Setup Your Environment

### Setup a Local Dev Chain

You should be able to run a dev chain and know about how to use `ckb-cli` to send transactions.  If you do not, please refer to this tutorial：[How to use a development blockchain](basics/guides/devchain.md).  Please don't forget to add `ckb-cli` to the  PATH environment variable

### Install Capsule

#### Prerequisites

The following must be installed to use Capsule.

- Rust and Cargo - Capsule uses Rust's build system and package manager "cargo" to generate Rust contracts and run tests. Visit the Rust website for instructions on [how to install Rust](https://www.rust-lang.org/tools/install).
- Docker - Capsule uses a Docker container to build contracts in a controlled environment that can consistently reproduce binary identical scripts. Visit the Docker website for instructions on [how to install Docker](https://docs.docker.com/get-docker/).
- Cross - Capsule uses Cross to help cross-compile Rust contracts for RISC-V. Cross can be installed with Cargo by the following command:

```bash
cargo install cross --git https://github.com/cross-rs/cross
```

Note: The current user must have permission to manage Docker instances. For more information, see [Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user).

#### Installing Capsule

Capsule can be installed by downloading the precompiled binary, or by using `cargo` to build it from source. **Downloading the binary is generally recommended as the most quick and simple method.**

The precompiled binary can be downloaded from the [Capsule releases page](https://github.com/nervosnetwork/capsule/releases).

To build from source, execute the following `cargo` command:

```bash
cargo install ckb-capsule --git https://github.com/nervosnetwork/capsule.git --branch develop
```

#### Verifying the Capsule Installation

To verify that Capsule is installed correctly use Capsule's `check` command:

```bash
capsule check
```

<details>
<summary>(click here to view output)</summary>

```bash
------------------------------
cargo      installed
docker     installed
cross-util installed
ckb-cli    installed v1.5.0 (required v1.2.0)
------------------------------
```

</details>


### Creating a New Project in Capsule

Use Capsule's `new` command to generate a new project:

```bash
capsule new my-sudt
```

<details>
<summary>(click here to view output)</summary>

```bash
New project "my-sudt"
Created file "capsule.toml"
Created file "deployment.toml"
Created file "README.md"
Created file "rust-toolchain"
Created file "Cross.toml"
Created file "Cargo.toml"
Created file ".gitignore"
Initialized empty Git repository in ./my-sudt/.git/
Created "./my-sudt"
Created tests
New contract "my-sudt"
Rewrite Cargo.toml
Rewrite capsule.toml
Done
```

</details>

The project structure will look similar to the following:

```bash
ls my-sudt
```

<details>
<summary>(click here to view output)</summary>

```bash
build  capsule.toml  Cargo.toml  contracts  Cross.toml  deployment.toml  migrations  README.md  rust-toolchain  tests
```

</details>

The default contract can be found in the `my-sudt/contracts/my-sudt` directory. This a normal Cargo project:

```bash
ls my-sudt/contracts/my-sudt
```

<details>
<summary>(click here to view output)</summary>

```bash
Cargo.toml  src
```

</details>

In `my-sudt/contracts/my-sudt/src/entry.rs` you will find some pre-generated boiler-plate code:

<details>
<summary>(click here to view full code)</summary>

```rust
// Import from `core` instead of from `std` since we are in no-std mode
use core::result::Result;

// Import heap related library from `alloc`
// https://doc.rust-lang.org/alloc/index.html
use alloc::{vec, vec::Vec};

// Import CKB syscalls and structures
// https://docs.rs/ckb-std/
use ckb_std::{
    debug,
    high_level::{load_script, load_tx_hash},
    ckb_types::{bytes::Bytes, prelude::*},
};

use crate::error::Error;

pub fn main() -> Result<(), Error> {
    // remove below examples and write your code here

    let script = load_script()?;
    let args: Bytes = script.args().unpack();
    debug!("script args is {:?}", args);

    // return an error if args is invalid
    if args.is_empty() {
        return Err(Error::MyError);
    }

    let tx_hash = load_tx_hash()?;
    debug!("tx hash is {:?}", tx_hash);

    let _buf: Vec<_> = vec![0u8; 32];

    Ok(())
}

// Unit tests are supported.
#[test]
fn test_foo() {
    assert!(true);
}
```

</details>

The code above is boiler-plate code. It doesn't do very much, but it contains pieces that are very useful for most scripts, and we will use some of it later on.

### Building a Project

To build the project, enter the main project folder and use Capsule's `build` command:

```bash
cd my-sudt
capsule build
```

<details>
<summary>(click here to view output)</summary>

```bash
Building contract my-sudt
$ cross build -p my-sudt
info: downloading component 'rust-std' for 'riscv64imac-unknown-none-elf'
info: installing component 'rust-std' for 'riscv64imac-unknown-none-elf'
Unable to find image 'nervos/ckb-riscv-gnu-toolchain:focal-20230214' locally
focal-20230214: Pulling from nervos/ckb-riscv-gnu-toolchain
7608715873ec: Pull complete
a237f683deea: Pull complete
84750871be6c: Pull complete
3056067387e4: Pull complete
258c66296219: Pull complete
e8ea3d081562: Pull complete
32afd1383aeb: Pull complete
6f6c3bd439ca: Pull complete
Digest: sha256:5732afe996b2b88a37ed3c0e4deb948e9d2a6936fb4b52d4e7a0dc5ad330c306
Status: Downloaded newer image for nervos/ckb-riscv-gnu-toolchain:focal-20230214
   Compiling libc v0.2.150
   Compiling cfg-if v1.0.0
   Compiling blake2b-ref v0.3.1
   Compiling buddy-alloc v0.5.1
   Compiling molecule v0.7.5
   Compiling ckb-standalone-types v0.1.5
   Compiling cc v1.0.83
   Compiling ckb-std v0.14.3
   Compiling my-sudt v0.1.0 (contracts/my-sudt)
    Finished dev [unoptimized + debuginfo] target(s) in 2.25s
Copying to target directory
Done
```

</details>

* Note: You may see a warning similar to: `<jemalloc>: MADV_DONTNEED does not work (memset will be used instead)`. This is expected behaviour and will not cause any problems. It is caused by jemalloc under a QEMU env, which is expected if you're running non-native Docker containers under the `aarch64` architecture (for example an x86-64 container).

You will find a new generated contract binary in the `build/debug` directory:

```
ls build/debug
```

<details>
<summary>(click here to view output)</summary>

```bash
my-sudt
```

</details>

Congratulations, you have successfully created and built your first project! Now let's look at the script for an SUDT token.

## Writing an SUDT Script

The [SUDT standard](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0025-simple-udt/0025-simple-udt.md) is the most simple real script that is in use within the Nervos ecosystem. It is a minimalist token standard with only basic functionality. In this section we will rebuild this script piece by piece to show how it is logically constructed.

### Modes of Operation

The SUDT script supports two modes of operation, **Owner Mode** and **Normal Mode**. Each of these includes different verification rules:

* **Owner Mode**：If it is detected that the original creator of the token (the owner) is initiating the transaction, the script will enter owner mode. This is will disable certain checks which allows priviledged operations, such as minting new tokens. 
* **Normal Mode**: If owner mode is not active, then normal mode will be used. This is the default mode for all users, and enforces rules to prevent the minting of new tokens.

Later on we will describe how owner mode is detected. For now, knowing what these two modes are will be enough to understand the logic of the next sections.

Next, we will look at the libraries that are used:

### Reviewing the Libraries Used

If you open `contracts/my-sudt/Cargo.toml`, we have the following dependency:

```
[dependencies]
ckb-std = "0.14.0"
```

The `ckb-std` dependency is a special crate used to handling CKB syscalls when the script is executed in CKB-VM.

You may refer to Capsule's Wiki on [Rust Libraries](https://github.com/nervosnetwork/capsule/wiki/Rust-libraries) for more useful crates recommended for use when building scripts with Capsule.

> Note: You can use any existing Rust libraries in your scripts, which means there are a lot of crates that you can use in your project. However, crates must support `no-std` or they will not compile correctly.

### Load Script

At the beginning of the script, we need to check the SUDT script will operate in owner mode or normal mode. If it is owner mode, we skip the verification code otherwise we run additional validation to check the token amount.

To check for owner mode, we first need to load `args` of the current script. The `args` field contains the lock hash of the owner. This defines who the owner of the token is, and controls who gets to use owner mode to mint new tokens.

The code to load the `args` is already present in the boiler-plate code within `contracts/my-sudt/src/entry.rs`:

```rust
let script = load_script()?;
let args: Bytes = script.args().unpack();
```

We then add in the following code to check our `args` for owner mode.

```rust
if check_owner_mode(&args)? {
    return Ok(());
}
```

Below we have combined them within `main()`, but it won't work yet because we still have to define `check_owner_mode()`.

```rust
// contracts/my-sudt/src/entry.rs
fn main() -> Result<(), Error> {
    // load current script args
    let script = load_script()?;
    let args: Bytes = script.args().unpack();

    // check verification branch is owner mode or normal mode
    // return success if owner mode is true
    if check_owner_mode(&args)? {
        return Ok(());
    }

    // more verifications will go here later...
    return Ok(());
}
```

### Check Owner Mode

Next we define the `check_owner_mode()` function. This function will cycle through all the inputs, calculating the lock hash for each, and compare it to the script's `args` to see if the values match.

In the SUDT standard, when someone first create a new token, they set the `args` field to their own lock hash. This serves two purposes: The `args` define who owns the token, and it also provides a means to identify the token from other tokens.

Below is the code for `check_owner_mode()`.

```rust
use ckb_std::{
    high_level::{load_cell_lock_hash},
    ckb_constants::Source,
};

fn check_owner_mode(args: &Bytes) -> Result<bool, Error> {
    // With owner lock script extracted, we will look through each input in the
    // current transaction to see if any unlocked cell uses owner lock.
    for i in 0.. {
        // check input's lock_hash with script args
        let lock_hash = match load_cell_lock_hash(
            i,
            Source::Input,
        ) {
            Ok(lock_hash) => lock_hash,
            Err(SysError::IndexOutOfBound) => return Ok(false),
            Err(err) => return Err(err.into()),
        };
        // invalid length of loaded data
        if args[..] == lock_hash[..] {
           return Ok(true);
        }
    }
    Ok(false)
}
```

In the code above, we use the [load_cell_lock_hash()](https://docs.rs/ckb-std/latest/ckb_std/high_level/fn.load_cell_lock_hash.html) call to query for the lock hash of each input cell. The lock hash is a Blake2b hash of the actual bytes that comprise the lock script of the cell. What is important to know about this is the lock hash is a means of identifying a particular owner of a cell.

Our calls to `load_cell_lock_hash()` specify the cell index, denoted by `i`, as located in the transaction. The second parameter, `Source::Input`, specifies that we want to query for input cells of the transaction.

Our loop, `for i in 0..`, will continuously cycle through all the cells searching for a cell with the same lock hash as that which was specified in the `args`, `if args[..] == lock_hash[..]`. If we find a match we immediately return with `true` to indicate that owner mode is enabled. If we cycle through all the cells and don't find a match, eventually we will run out of input cells and receive an `IndexOutOfBound` error. At that point we return `false`, indicating we are in normal mode.

The pattern we are using here to determine the owner is known as "authorization delegation". Every input contains a lock script, and we are checking for the existence of a particular lock script instead of doing any kind of more complex authorization in our script. This works because we know that all the lock scripts on the input cells have already been authorized, because if they were not authorized the transaction would fail. This is called authorization delegation because our script is *delegating* the task of authorizing to a different lock script by checking for its existence instead of having to include more logic to handle the authorization itself.

### Check Token Amounts

If the owner mode is `false`, then we are in normal mode, and we continue verification. SUDT is a minimal standard, so the logic we need to verify is quite simple:

> The total amount of input tokens must be greater than or equal to the total amount of output tokens.

In other words, we are ensuring that the amount of SUDT tokens in this transaction never exceeds what we started with. You can transfer the tokens you have or burn them, but you cannot create more tokens from nothing.

We will define two methods:

* `collect_inputs_amount()`：Counts the total amount of input SUDT tokens.
* `collect_outputs_amount()`：Counts the total amount of output SUDT tokens.

Below is the code for both functions. There are a few important things to point out.

* The constant `UDT_LEN` is set to `16`, which is the size of a `u128`. This is needed because SUDT uses a unsigned 128-bit integer to store the amount of tokens in the data field of the cell.
* We are using `load_cell_data()`, which loads the data from the cell. This is how we read the amount of tokens present in the cell.
* We rely on `Source::GroupInput` and `Source::GroupOutput` as our sources. These are similar to `Source::Input` and `Source::Output`, but they provide a shortcut to filter the cells. `Input` and `Output` return all the input cells and output cells. Not all of the cells in the transactions may be relevant to what we are doing. We are only interested in SUDT token cells, and `GroupInput` and `GroupOutput` will only return cells with the same script as is currently executing. Since it is the SUDT script that is executing, `GroupInput` and `GroupOutput` will only return other SUDT cells.

```rust
const UDT_LEN: usize = 16;

fn collect_inputs_amount() -> Result<u128, Error> {
    // let's loop through all input cells containing current UDTs,
    // and gather the sum of all input tokens.
    let mut inputs_amount: u128 = 0;
    let mut buf = [0u8; UDT_LEN];

    // u128 is 16 bytes
    for i in 0.. {
        let data = match load_cell_data(i, Source::GroupInput) {
            Ok(data) => data,
            Err(SysError::IndexOutOfBound) => break,
            Err(err) => return Err(err.into()),
        };

        if data.len() != UDT_LEN {
            return Err(Error::Encoding);
        }
        buf.copy_from_slice(&data);
        inputs_amount += u128::from_le_bytes(buf);
    }
    Ok(inputs_amount)
}

fn collect_outputs_amount() -> Result<u128, Error> {
    // With the sum of all input UDT tokens gathered, let's now iterate through
    // output cells to grab the sum of all output UDT tokens.
    let mut outputs_amount: u128 = 0;
    let mut i = 0;

    // u128 is 16 bytes
    let mut buf = [0u8; UDT_LEN];
    for i in 0.. {
        let data = match load_cell_data(i, Source::GroupOutput) {
            Ok(data) => data,
            Err(SysError::IndexOutOfBound) => break,
            Err(err) => return Err(err.into()),
        };

        if data.len() != UDT_LEN {
            return Err(Error::Encoding);
        }
        buf.copy_from_slice(&data);
        outputs_amount += u128::from_le_bytes(buf);
    }
    Ok(outputs_amount)
}
```

The code in `collect_inputs_amount()` and `collect_outputs_amount()` will count of the total tokens we have so that we can later compare these values and determine if there is a problem with the amount.

Next we update the `error.rs` file to add a new custom error, `Amount`, which stands for "error with token amount". You can see this added below within the `Error` enum.

```rust
use ckb_std::error::SysError;

/// Error
#[repr(i8)]
pub enum Error {
    IndexOutOfBound = 1,
    ItemMissing,
    LengthNotEnough,
    Encoding,
    // Add customized errors here...
    Amount,
}

impl From<SysError> for Error {
    fn from(err: SysError) -> Self {
        use SysError::*;
        match err {
            IndexOutOfBound => Self::IndexOutOfBound,
            ItemMissing => Self::ItemMissing,
            LengthNotEnough(_) => Self::LengthNotEnough,
            Encoding => Self::Encoding,
            Unknown(err_code) => panic!("unexpected sys error {}", err_code),
        }
    }
}
```

Going back to `entry.rs`, we will add the following code to `main()` to check the token values.

```rust
let inputs_amount = collect_inputs_amount()?;
let outputs_amount = collect_outputs_amount()?;

if inputs_amount < outputs_amount {
    return Err(Error::Amount);
}
```

The code above should be fairly straightforward. We gather our token amounts, then ensure that `the total amount of input tokens is greater than or equal to the total amount of output tokens`. If it is not, then we return the `Amount` error.

Below is the complete code we have now.

```rust
fn main() -> Result<(), Error> {
    // remove below examples and write your code here

    let script = load_script()?;
    let args: Bytes = script.args().unpack();

    // return success if owner mode is true
    if check_owner_mode(&args)? {
        return Ok(());
    }

    let inputs_amount = collect_inputs_amount()?;
    let outputs_amount = collect_outputs_amount()?;

    if inputs_amount < outputs_amount {
        return Err(Error::Amount);
    }

    Ok(())
}
```

### Use QueryIter to Query Cells

In the previous code, we use `for` loop to iterate inputs and outputs. However, this does not take advantage of Rust's iterator syntax. Since iteration over cells is a common pattern in CKB programming, `ckb-std` provides a high-level iterator interface called [QueryIter](https://docs.rs/ckb-std/latest/ckb_std/high_level/struct.QueryIter.html).

QueryIter uses two arguments. The first is a loading function, the seconds is `Source`. Here is an example that loads the data from all GroupInput cells:  `QueryIter::new(load_cell_data, Source::GroupInput)`.

Below, we rewrite our functions in `entry.rs` to utilize `QueryIter`:

```rust
// Imports
use ckb_std::high_level::QueryIter;

// Our rewritten functions:
fn check_owner_mode(args: &Bytes) -> Result<bool, Error> {
    // With owner lock script extracted, we will look through each input in the
    // current transaction to see if any unlocked cell uses owner lock.
    let is_owner_mode = QueryIter::new(load_cell_lock_hash, Source::Input)
        .find(|lock_hash| args[..] == lock_hash[..]).is_some();
    Ok(is_owner_mode)
}

fn collect_inputs_amount() -> Result<u128, Error> {
    // let's loop through all input cells containing current UDTs,
    // and gather the sum of all input tokens.
    let mut buf = [0u8; UDT_LEN];

    let udt_list = QueryIter::new(load_cell_data, Source::GroupInput)
        .map(|data|{
            if data.len() == UDT_LEN {
                buf.copy_from_slice(&data);
                // u128 is 16 bytes
                Ok(u128::from_le_bytes(buf))
            } else {
                Err(Error::Encoding)
            }
        }).collect::<Result<Vec<_>, Error>>()?;
    Ok(udt_list.into_iter().sum::<u128>())
}

fn collect_outputs_amount() -> Result<u128, Error> {
    // With the sum of all input UDT tokens gathered, let's now iterate through
    // output cells to grab the sum of all output UDT tokens.
    let mut buf = [0u8; UDT_LEN];

    let udt_list = QueryIter::new(load_cell_data, Source::GroupOutput)
        .map(|data|{
            if data.len() == UDT_LEN {
                buf.copy_from_slice(&data);
                // u128 is 16 bytes
                Ok(u128::from_le_bytes(buf))
            } else {
                Err(Error::Encoding)
            }
        }).collect::<Result<Vec<_>, Error>>()?;
    Ok(udt_list.into_iter().sum::<u128>())
}
```

This concludes the walkthrough of the Rust implementation of the SUDT script. It really is that simple! If you would like to review the full script, you can find it on [GitHub](https://github.com/jjyr/my-sudt/blob/master/contracts/my-sudt/src/main.rs).

The Rust version is designed to be an identical implementation to what is used on the mainnet, but the real mainnet implementation is actually written in the C programming lange. The source code for the C implementation is also available on [GitHub](https://github.com/nervosnetwork/ckb-miscellaneous-scripts/blob/master/c/simple_udt.c).

### Build the Project

To compile the project into a RISC-V compatible binary that is suitable for smart contracts in CKB-VM, use the following command in a terminal while in the main project directory.

```bash
capsule build
``` 

Assuming no errors occur during compilation, we can find the output binary at `my-sudt/build/debug/my-sudt`. 

> Note: This is a debug binary that is suitable for testing only. It contains all the debug symbols in case we need to debug a problems. It is not a release executable that is suitable for a mainnet. We will cover that later in the tutorial.

## Testing CKB Scripts

The scripts we are creating are designed to be run on a blockchain inside CKB-VM. This can be difficult to setup, and doesn't work well with unit tests. The `ckb-testtool` crate is used to help fill this need. With this, we can construct transactions that run in an emulated environment that is much less difficult to setup and unit test.

### Examining the Default Tests

When created the project `my-sudt`, default tests were also created. The default tests use unlocked mock cells for testing.

To run the default tests, first use `capsule build` to build the scripts, then use `capsule test` to run the tests. The following error message will be received:

```
failures:

---- tests::test_basic stdout ----
thread 'tests::test_basic' panicked at 'pass verification: Error { kind: ValidationFailure(4)Script }', tests/src/tests.rs:52:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace

failures:
    tests::test_basic
```



<div style="color: red; font-size: 20px">
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

The current problem is the my-sudt example does not match the current boiler plate.

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
</div>





The error number `4` in `Error { kind: ValidationFailure(4)`  refers to `Error::Encoding`, which means the cell's data type is not `u128`. 

Let's check the default tests code  `tests/src/tests.rs`  to find out how to write the tests, then we can write new tests adapted `my-sudt`：

* In the beginning part, initialize the  `Context` which is a structure to simulate the chain environment. We can use `Context` to deploy exists cells and mock block headers.`deploy_contract` will return the  `out_point` of the script.

```rust
// deploy contract
    let mut context = Context::default();
    let contract_bin: Bytes = Loader::default().load_binary("my-sudt");
    let contract_out_point = context.deploy_contract(contract_bin);
```

* Then `build_script` is called with the script's `out_point` , this function returns the `Script` which uses our script as the code. `create_cell` creates an existing cell in the context, which uses our script as the `lock_script`.

*Please note the default tests assume the script is a lock_script, but in our case, `my-sudt` is a type_script. We'll fix it later.*

```rust
// prepare scripts
    let lock_script = context.build_script(&contract_out_point, Default::default()).expect("script");
    let lock_script_dep = CellDep::new_builder().out_point(contract_out_point).build();

    // prepare cells
    let input_out_point = context.create_cell(
        CellOutput::new_builder()
            .capacity(1000u64.pack())
            .lock(lock_script.clone())
            .build(),
        Bytes::new(),
    );
    let input = CellInput::new_builder()
        .previous_output(input_out_point)
        .build();
```

* After that, build two outputs cells and a transaction structure.It is necessary to include  `cell_deps` field in the transaction which should contain all the referenced scripts, in this case, we can only refer to `my-sudt`.  `complete_tx`  also implement `cell_deps`, while the field is already completed manually, this line is not necessary.

Please note that the  transaction's `outputs_data` must have the same length with the `outputs`, even the data is empty.

```rust
let outputs = vec![
        CellOutput::new_builder()
            .capacity(500u64.pack())
            .lock(lock_script.clone())
            .build(),
        CellOutput::new_builder()
            .capacity(500u64.pack())
            .lock(lock_script)
            .build(),
    ];

    let outputs_data = vec![Bytes::new(); 2];

    // build transaction
    let tx = TransactionBuilder::default()
        .input(input)
        .outputs(outputs)
        .outputs_data(outputs_data.pack())
        .cell_dep(lock_script_dep)
        .build();
    let tx = context.complete_tx(tx);
```

* Finally, verify the transaction:

```
// run
    context
        .verify_tx(&tx, MAX_CYCLES)
        .expect("pass verification");
```

### Writing New Tests

We should create mock SUDT cells and spend them for testing SUDT verification.
As `my-sudt` script is a `type_script` we need another script as `lock_script` for mock cells, it is recommended to use `always success` script returned `0`. `always success` is built-in in the `ckb-testtool`.


```rust
use ckb_testtool::{builtin::ALWAYS_SUCCESS, context::Context};

    // deploy always_success script
    let always_success_out_point = context.deploy_contract(ALWAYS_SUCCESS.clone());
```

Before writing the code, let's think about our  test cases:

1. Return success when input tokens equals to output tokens.
2. Return success when input tokens is greater than output tokens.
3. Return failure when input tokens is less than output tokens.
4. Return success when input tokens is less than output tokens with `owner mode` activated.

* Define `build_test_context` to build transactions. There are three args: 
    * The data type of  `inputs_token` and `outputs_token`  is  `u128`. The function can generate SUDT inputs cells and outputs cells according to the two args.
    *  `is_owner_mode` refers to the current transaction is in SUDT owner mode or normal mode.

* Deploy the SUDT and `always-success` scripts.

Please note that if `is_owner_mode` is true, we will set `lock_script`'s `lock_hash` as `owner script hash`; otherwise, we will set `[0u8; 32]` which implies can't enter into owner mode.

```rust
fn build_test_context(
    inputs_token: Vec<u128>,
    outputs_token: Vec<u128>,
    is_owner_mode: bool,
) -> (Context, TransactionView) {
    // deploy my-sudt script
    let mut context = Context::default();
    let sudt_bin: Bytes = Loader::default().load_binary("my-sudt");
    let sudt_out_point = context.deploy_contract(sudt_bin);
    // deploy always_success script
    let always_success_out_point = context.deploy_contract(ALWAYS_SUCCESS.clone());

    // build lock script
    let lock_script = context
        .build_script(&always_success_out_point, Default::default())
        .expect("script");
    let lock_script_dep = CellDep::new_builder()
        .out_point(always_success_out_point)
        .build();

    // build sudt script
    let sudt_script_args: Bytes = if is_owner_mode {
        // use always_success script hash as owner's lock
        let lock_hash: [u8; 32] = lock_script.calc_script_hash().unpack();
        lock_hash.to_vec().into()
    } else {
        // use zero hash as owner's lock which implies we can never enter owner mode
        [0u8; 32].to_vec().into()
    };

    let sudt_script = context
        .build_script(&sudt_out_point, sudt_script_args)
        .expect("script");
    let sudt_script_dep = CellDep::new_builder().out_point(sudt_out_point).build();

    //... more code below
//}
```

* Build inputs and outputs according to the `inputs_token` and `outputs_token` 

```rust
// ...
    // prepare inputs
    // assign 1000 Bytes to per input
    let input_ckb = Capacity::bytes(1000).unwrap().as_u64();
    let inputs = inputs_token.iter().map(|token| {
        let input_out_point = context.create_cell(
            CellOutput::new_builder()
                .capacity(input_ckb.pack())
                .lock(lock_script.clone())
                .type_(Some(sudt_script.clone()).pack())
                .build(),
            token.to_le_bytes().to_vec().into(),
        );
        let input = CellInput::new_builder()
            .previous_output(input_out_point)
            .build();
        input
    });

    // prepare outputs
    let output_ckb = input_ckb * inputs_token.len() as u64 / outputs_token.len() as u64;
    let outputs = outputs_token.iter().map(|_token| {
        CellOutput::new_builder()
            .capacity(output_ckb.pack())
            .lock(lock_script.clone())
            .type_(Some(sudt_script.clone()).pack())
            .build()
    });
    let outputs_data: Vec<_> = outputs_token
        .iter()
        .map(|token| Bytes::from(token.to_le_bytes().to_vec()))
        .collect();
    // ...
```

*  Finally construct the transaction and return it with context.

```
// build transaction
    let tx = TransactionBuilder::default()
        .inputs(inputs)
        .outputs(outputs)
        .outputs_data(outputs_data.pack())
        .cell_dep(lock_script_dep)
        .cell_dep(sudt_script_dep)
        .build();
    (context, tx)
```

Now the helper function `build_test_context` is finished, we can write our tests: 

```rust
#[test]
fn test_basic() {
    let (mut context, tx) = build_test_context(vec![1000], vec![400, 600], false);
    let tx = context.complete_tx(tx);

    // run
    context
        .verify_tx(&tx, MAX_CYCLES)
        .expect("pass verification");
}

#[test]
fn test_destroy_udt() {
    let (mut context, tx) = build_test_context(vec![1000], vec![800, 100, 50], false);
    let tx = context.complete_tx(tx);

    // run
    context
        .verify_tx(&tx, MAX_CYCLES)
        .expect("pass verification");
}

#[test]
fn test_create_sudt_without_owner_mode() {
    let (mut context, tx) = build_test_context(vec![1000], vec![1200], false);
    let tx = context.complete_tx(tx);

    // run
    let err = context.verify_tx(&tx, MAX_CYCLES).unwrap_err();
    assert_error_eq!(err, ScriptError::ValidationFailure(ERROR_AMOUNT));
}

#[test]
fn test_create_sudt_with_owner_mode() {
    let (mut context, tx) = build_test_context(vec![1000], vec![1200], true);
    let tx = context.complete_tx(tx);

    // run
    context
        .verify_tx(&tx, MAX_CYCLES)
        .expect("pass verification");
}
```

You may refer to [my-sudt tests](https://github.com/jjyr/my-sudt/blob/master/tests/src/tests.rs) for the full tests. Run `capsule test`  all tests will be passed.

## Deployment

### Run a dev chain and ckb-cli

You should be running a dev chain and know about how to use `ckb-cli` to send transactions before deployment. 

### Deploy

1. Update the deployment configurations

   Open  `deployment.toml` :

     *  `cells`  describes which cells to be deployed.

        * `name`: Define the reference name used in the deployment configuration.
        * `enable_type_id` : If it is set to `true` means create a `type_id` for the cell.
        * `location` :  Define the script binary path.

     *  `dep_groups`  describes which dep_groups to be created. Dep Group is a cell which bundles several cells as its members. When a dep group cell is used in `cell_deps`, it has the same effect as adding all its members into `cell_deps`. In our case, we don't need `dep_groups`.
     * `lock`  describes the `lock` field of the new deployed cells.It is  recommended to set `lock` to the deployer's address(an address that you can unlock) in the dev chain and in the testnet, which is easier to update the script.

2. Uncomment the configuration file and replace the cell name and location with `my-sudt`.

```
# [[cells]]
# name = "my_cell"
# enable_type_id = false
# location = { file = "build/release/my_cell" }

# # Dep group cells
# [[dep_groups]]
# name = "my_dep_group"
# cells = [
#   "my_cell",
#   "secp256k1_data"
# ]

# # Replace with your own lock if you want to unlock deployed cells.
# # The deployment code_hash is secp256k1 lock
# [lock]
# code_hash = "0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8"
# args = "0x0000000000000000000000000000000000000000"
# hash_type = "type"
```

3. Build release version of the script
   
  * The release version of script  doesn't  include debug symbols which makes the size smaller.

```
capsule build --release
```

4. Deploy the script 

```
capsule deploy --address <ckt1....>
```

If the `ckb-cli` has been installed and dev-chain RPC is connectable, you will see the `deployment plan`:

* `new_occupied_capacity` and `total_occupied_capacity`  refer how much CKB to store cells and data.
* `txs_fee_capacity` refers how much CKB to pay the transaction fee.

```
Deployment plan:
---
migrated_capacity: 0.0 (CKB)
new_occupied_capacity: 33629.0 (CKB)
txs_fee_capacity: 0.0001 (CKB)
total_occupied_capacity: 33629.0 (CKB)
recipe:
  cells:
    - name: my-sudt
      index: 0
      tx_hash: 0x8b496cb19018c475cdc4605ee9cef83cbfe578dce4f81f3367395906eba52c29
      occupied_capacity: 33629.0 (CKB)
      data_hash: 0xaa3d472025e6afefdf3f65c5f9beefd206b4283b30551baef83cbb4762e6d397
      type_id: ~
  dep_groups: []
Confirm deployment? (Yes/No)
```

5. Type `yes` or `y`  and input the password to unlock the account.

```
send cell_tx 8b496cb19018c475cdc4605ee9cef83cbfe578dce4f81f3367395906eba52c29
Deployment complete
```

Now the SUDT script has been deployed, you can refer to this script by using `tx_hash: 0xaa3d472025e6afefdf3f65c5f9beefd206b4283b30551baef83cbb4762e6d397 index: 0` as `out_point`(your `tx_hash` should be another value).

### Migration

If you want to update the script code and deploy again, you can simply run this command again:

```bash
capsule deploy --address ckt1qyq075y5ctzlgahu8pgsqxrqnglajgwa9zksmqdupd
```

The new script will be automatically migrated which means destroy the old script cells and create new cells.
You will find  `new_occupied_capacity` is `0` because `capacity` is already covered by the old script cells.Please don't forget the transaction fee you still need to pay it.

```
Deployment plan:
---
migrated_capacity: 33629.0 (CKB)
new_occupied_capacity: 0.0 (CKB)
txs_fee_capacity: 0.0001 (CKB)
total_occupied_capacity: 33629.0 (CKB)
recipe:
  cells:
    - name: my-sudt
      index: 0
      tx_hash: 0x10d508a0b44d3c1e02982f85a3e9b5d23d3961fddbf554d20abb4bf54f61950a
      occupied_capacity: 33629.0 (CKB)
      data_hash: 0xaa3d472025e6afefdf3f65c5f9beefd206b4283b30551baef83cbb4762e6d397
      type_id: ~
  dep_groups: []
Confirm deployment? (Yes/No)
```

## Next Steps

This is the end of our journey into writing a SUDT script by Capsule. Congratulations on making it to this milestone. If you have questions feel free to contact us on [Nervos Talk](https://talk.nervos.org/) or [Discord](https://discord.gg/FKh8Zzvwqa).
