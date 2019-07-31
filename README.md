# Celo related notes

## Production

### How to deploy a new network


#### Thorough but slow
```
export NETWORK_NAME=integration && celotooljs deploy initial testnet -e ${NETWORK_NAME} && celotooljs deploy initial contracts -e ${NETWORK_NAME}
```

#### Fast while keeping slow things like load balancers intact
```
export NETWORK_NAME=integration && celotooljs deploy upgrade testnet -e ${NETWORK_NAME} --reset
```

### How to upgrade existing network

```
export NETWORK_NAME=integration && celotooljs deploy upgrade testnet -e ${NETWORK_NAME} && celotooljs deploy upgrade contracts -e ${NETWORK_NAME}
```

## Geth

### How to build geth for Mac OS

1. `make -j geth` or,
2. `celotooljs geth build` ([celotooljs](https://github.com/celo-org/celo-monorepo/tree/master/packages/celotool))


### How to build geth for Android

```
ANDROID_NDK=/usr/local/Caskroom/android-ndk/18/android-ndk-r18/ make android -j
```
This produces a binary at `build/bin/geth.aar`. Copy it over to monorepo to test the new geth with Android

```
cp -v ~/celo/geth/build/bin/geth.aar ~/celo/celo-monorepo2/node_modules/@celo/client/build/bin/geth.aar
```

### How to test geth

1. `make -j test` - this runs, primarily, the unittests which came  from go-ethereum open-source package. To run an individual test, say tests for `eth/downloader.go`, run `build/env.sh go run build/ci.go test ./eth/downloader`
2. See more end-to-end defined in `.circleci/config.yml`

### Calling Smart Contracts from geth

Sometimes, you want to call a smart contract to fetch some data from the blockchain or modify the state. Here's a small primer on how to do that.

#### What you need

 1. [`EVM`](https://github.com/celo-org/geth/blob/master/core/vm/evm.go#L108) object
2. Location of the contract `contractAddress` - either the contract is pre-compiled and deployed at a [fixed location](https://github.com/celo-org/geth/blob/master/params/protocol_params.go#L107) or the location is passed as a geth command-line parameter.
3. Function selector - The function signature like `balanceOf(address)` is hashed and trimmed to get a function selector, see an [example](https://github.com/celo-org/geth/blob/c7e03ac465dbe8b8c8b70fa09aac267b7d624d19/core/state_transition.go#L328).
4. ABI - Combining the function selector and arguments converted to ABI gives you the full `transactiondata`. This [blog post](https://medium.com/@hayeah/how-to-decipher-a-smart-contract-method-call-8ee980311603) is a good introduction. Here's how we [implement](https://github.com/celo-org/geth/blob/c7e03ac465dbe8b8c8b70fa09aac267b7d624d19/core/state_transition.go#L346-L362) it
5. Choose the caller. Set the required caller `caller := vm.AccountRef(common.HexToAddress("0x0" /* caller address */))`. Depending on the call code, the address might have to fulfill a specific criteria or any address might work.

#### Make the call

If the call is view-only and won't modify the state, use `evm.StaticCall`. Signature is `returnValue, leftoverGas, err := evm.StaticCall(caller, contractAddress, transactionData, gasLimit)`. If we are calling random untrusted functions, pass a reasonable `gasLimit` since `gasPrice` for view calls is 0.

Example of a view call: [balanceOf](https://github.com/celo-org/geth/blob/c7e03ac465dbe8b8c8b70fa09aac267b7d624d19/core/state_transition.go#L233)

If the call is state-modifying then use `evm.Call`. Signature is `ret, leftoverGas, err := evm.Call(caller, contractAddress, transactionData, gasLimit, moneyToTransfer)`. We can, in principle, pass the full gas as the gas limit here since `gasPrice` for non-view calls is not 0, so, the user will end up paying the price for these calls.

Example of a state-modifying call: [credit/debit Stable Token balance](https://github.com/celo-org/geth/blob/c7e03ac465dbe8b8c8b70fa09aac267b7d624d19/core/state_transition.go#L268)


## Mobile

### How to build a release binary for Android - signed by debug signatures
```
CELO_RELEASE_STORE_PASSWORD='' ./gradlew assembleRelease --info && echo android | java -jar ~/src/adb-enhanced/src/apksigner.jar sign --ks ~/.android/debug.keystore --ks-key-alias androiddebugkey ~/celo/celo-monorepo/packages/mobile/android/app/build/outputs/apk/release/app-x86-release-unsigned.apk
```