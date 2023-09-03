# Polkadot & Rise In Bootcamp - Final Project

Hello, I will be explaining what I have done to complete this final project.
It was stated in our Discord server that we actually only need to put our screenshots proving that our project is working.

## 1. Creating the blockchain

1. Forked the substrate-node-template repository
2. Switched to a new branch `git switch -c my-learning-branch`
3. Built and ran the node with `cargo build --release` and `./target/release/node-template --dev`.
4. Forked the frontend template named substrate-front-end-template
5. Installed the frontend using `yarn install`
6. Ran it using `yarn run`
7. Made a transaction using frontend.

## 2. Simulating a substrate network

1. Ran `./target/release/node-template purge-chain --base-path /tmp/alice --chain local` to remove the chain data of alice.
2. Started the chain with

   ```
   ./target/release/node-template \
   --base-path /tmp/alice \
   --chain local \
   --alice \
   --port 30333 \
   --rpc-port 9933 \
   --node-key 0000000000000000000000000000000000000000000000000000000000000001 \
   --telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
   --validator
   ```

   The `--ws-port 9945` flag was not working properly so it was not used.
   
4. Ran the `./target/release/node-template purge-chain --base-path /tmp/bob --chain local` to try to remove the old data of bob
5. Started the node using

   ```
   ./target/release/node-template \
   --base-path /tmp/bob \
   --chain local \
   --bob \
   --port 30334 \
   --rpc-port 9934 \
   --telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \   
   --validator \
   --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp
   ```

6. Each of our nodes (bob and alice) has discovered each other:

   ```
   discovered: 12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp /ip4/192.168.1.40/tcp/30333
   ```

   And now, we successfully have "1 peers".

   ```
   💤 Idle (1 peers), best: #2 (0x290d…a070), finalized #0 (0x2f6a…8b1a), ⬇ 0.6kiB/s ⬆ 0.6kiB/s   
   ```

## 3. Adding trusted nodes to a network.

1. Created another sudo user in my WSL
2. Generated aura and grandpa keys for each.

### Generating keys for the first user

1. To get the aura key, I ran:

```
./target/release/node-template key generate --scheme Sr25519 --password-interactive
```

2. To get the grandpa key, I ran:

```
./target/release/node-template key inspect --password-interactive --scheme Ed25519 "blue mind pact oppose drink clump stay recipe exile seed dinosaur skin"
```

### Generating keys for the 2nd account

I exactly followed the same steps as I did for the first user.

1. Aura key:

```
./target/release/node-template key generate --scheme Sr25519 --password-interactive
```

2. Grandpa key:

```
./target/release/node-template key inspect --password-interactive --scheme Ed25519 "elder glide lady sail drive cupboard exchange need cradle glove habit diet"
```

### customSpec.json

1. Exporting the file:

```
./target/release/node-template build-spec --disable-default-bootnode --chain local > customSpec.json
```

2. Changed the name to `"name": "My Custom Testnet"`
3. Added the accounts as authorities:

```
"aura": {
        "authorities": [
          "5E4CcVj4yqxngNfLH9krG5dSXSeZYnAkon1KKpipjASp2GSX",
          "5CrBmqJNP8PQ4E3bhJb8rwrd6GJrncGwLHQo2gLoqbjkcYAa"
        ]
      },
      "grandpa": {
        "authorities": [
          [
            "5FFLBaUQeNkRiD91S9Wv66BVcbiQH9V2iWV4NrkPvugmYexo",
            1
          ],
          [
            "5H5ZXWNaR83Cr4uDWM5oEwKWUEGyGF9GMdM1q6UGxMK2kxna",
            1
          ]
        ]
      }
```

4. Changed the custom spec to raw custom spec with:

```
./target/release/node-template build-spec --chain=customSpec.json --raw --disable-default-bootnode > customSpecRaw.json
```

### Adding keys to the keystore for node01

1:

```
./target/release/node-template key insert --base-path /tmp/node01 \
  --chain customSpecRaw.json \
  --scheme Sr25519 \
  --suri "blue mind pact oppose drink clump stay recipe exile seed dinosaur skin" \
  --password-interactive \
  --key-type aura
```

2:

```
./target/release/node-template key insert --base-path /tmp/node01 \
  --chain customSpecRaw.json \
  --scheme Ed25519 \
  --suri "blue mind pact oppose drink clump stay recipe exile seed dinosaur skin" \
  --password-interactive \
  --key-type gran
```

### Adding keys to the keystore for node02

1:

```
./target/release/node-template key insert --base-path /tmp/node02 \
  --chain customSpecRaw.json \
  --scheme Sr25519 \
  --suri "elder glide lady sail drive cupboard exchange need cradle glove habit diet" \
  --password-interactive \
  --key-type aura
```

1. Added the Account 1 grandpa keys to the node01 keystore.

```
./target/release/node-template key insert --base-path /tmp/node02 \
  --chain customSpecRaw.json \
  --scheme Ed25519 \
  --suri "elder glide lady sail drive cupboard exchange need cradle glove habit diet" \
  --password-interactive \
  --key-type gran
```

### Starting the nodes

1. Ran the first node:

```
  ./target/release/node-template \
  --base-path /tmp/node01 \
  --chain ./customSpecRaw.json \
  --port 30333 \
  --rpc-port 9933 \
  --validator \
  --rpc-methods Unsafe \
  --name node01 \
  --password-interactive
```

2. Ran the second node:

```
  ./target/release/node-template \
  --base-path /tmp/node02 \
  --chain ./customSpecRaw.json \
  --port 30333 \
  --rpc-port 9933 \
  --validator \
  --rpc-methods Unsafe \
  --name node02 \
  --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWS7v1tPw4b3isyRKxPsx3D4em9n2AwcFiAKb3M8TXPAwp\
  --password-interactive
```

Nodes has discovered each other succesfully

## 8. Smart contracts

### Environment setup

1. Installed necessary dependencies as it was stated in the document
2. Updated everything before starting
2. Installed the `cargo-contract`

   ```
   cargo install --force --locked cargo-contract --version 3.2.0
   ```
3. Downloaded the `substrate-contracts-node`

### Creating the smart contract

4. Ran `cargo contract new flipper` to create a contract named flipper
5. Tested with `cargo test`
6. Compiled it the contract with `cargo contract build`

### Deploying the smart contract

7. Ran `substrate-contracts-node`
8. Ran `./substrate-contracts-node --log info,runtime::contracts=debug 2>&1`
9. Ran:

   ```
   cargo contract instantiate -x --skip-confirm --constructor new --args "false" --suri //Alice --salt $(date +%s)
   ```
   
### Interacting with the smart contract

8. For the get function in the smart contract:

   ```
   cargo contract call --contract 5HciRHsxpEWfJLGXwe4CHNEpyveJ9T4BtS5TKvJBMUfztrtu --message get --suri //Alice --dry-run
   ```
   
9. For the flip function in the smart contract:

   ```
   cargo contract call --contract 5HciRHsxpEWfJLGXwe4CHNEpyveJ9T4BtS5TKvJBMUfztrtu --message flip --suri //Alice
   ```
   which outputs

### All the screenshots about the process:

![Screenshot_161](https://github.com/zorbeytorunoglu/riseInFinalProject/assets/6390409/a8f631e4-ce13-4fb6-8025-50c5df430b6f)
![Screenshot_162](https://github.com/zorbeytorunoglu/riseInFinalProject/assets/6390409/b0720f5e-b5e9-4b00-bd35-6a18f1556cda)
![Screenshot_163](https://github.com/zorbeytorunoglu/riseInFinalProject/assets/6390409/b6e16482-2a27-4965-9a1b-542aac1e6ba7)
![Screenshot_164](https://github.com/zorbeytorunoglu/riseInFinalProject/assets/6390409/ea7e7c1d-268b-44d5-b074-37bfa666a3e5)
![Screenshot_165](https://github.com/zorbeytorunoglu/riseInFinalProject/assets/6390409/f79bc506-9e8b-4c5e-8071-62c8f5b4e3b8)
![Screenshot_166](https://github.com/zorbeytorunoglu/riseInFinalProject/assets/6390409/09d4c4b0-805b-4adb-b41f-b88f94d56204)
![Screenshot_167](https://github.com/zorbeytorunoglu/riseInFinalProject/assets/6390409/8587c5e5-b162-49aa-8bcf-4b5a1a240e0c)
![Screenshot_168](https://github.com/zorbeytorunoglu/riseInFinalProject/assets/6390409/5f4842ee-77c4-4dfc-8098-7c51fa708ea8)
![Screenshot_160](https://github.com/zorbeytorunoglu/riseInFinalProject/assets/6390409/69d68223-5a71-4454-97c3-ae6e9c5ef592)

Let me know if you ever need any further details about my final project. Thank you! It was a great bootcamp experience!
