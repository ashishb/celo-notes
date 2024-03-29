# Celo related notes

  * [Production](#production)
  * [Geth](#geth)
  * [Mobile](#mobile)

<!-- Use http://ecotrust-canada.github.io/markdown-toc/ for TOC generation -->

## Production

### How to deploy a new network

#### Thorough but slow

```
export NETWORK_NAME=integration &&
 celotooljs deploy initial testnet -e ${NETWORK_NAME} &&
 celotooljs deploy initial contracts -e ${NETWORK_NAME}
```

#### Fast while keeping slow things like load balancers intact

1. Set the network name - `export NETWORK_NAME=integration`
2. Deploy the network - `celotooljs deploy upgrade testnet -e ${NETWORK_NAME} --reset`
3. Deploy blockscout - `celotooljs deploy initial blockscout -e ${NETWORK_NAME}`
4. Check that blocks are being mined - `open https://${NETWORK_NAME}-blockscout.celo-testnet.org/blocks`
5. (optional) Check that syncing works - `packages/celotool $ GETH_DIR=~/celo/geth ./geth_tests/integration_network_sync_test.sh ${NETWORK_NAMe} ultralight`
6. Deploy the contracts - `celotooljs deploy initial contracts -e ${NETWORK_NAME} --verbose`. This command buffers its output and therefore, you won't see anything for ~40 mins.
7. Test that contract interaction works - `packages/contractkit $ yarn build ${NETWORK_NAME} && yarn test`

### How to upgrade existing network

```
export NETWORK_NAME=integration && celotooljs deploy upgrade testnet -e ${NETWORK_NAME} && celotooljs deploy upgrade contracts -e ${NETWORK_NAME}
```

### How to modify verbosity of running geth

1. Log onto the running pod: `kubectl exec --namespace alfajoresstaging alfajoresstaging-validators-66 -i -t /bin/sh`
2. Execute `geth attach --exec "debug.verbosity(5)"`
3. Once you are done testing. Reset the verbosity back to a low value: `geth attach --exec "debug.verbosity(2)"`

### How to test modifications to geth by deploying a testnet

1. Checkout geth - [https://github.com/celo-org/geth](https://github.com/celo-org/geth)
2. Make your modifications to geth
3. Build your new docker image - `docker build -f Dockerfile -t gcr.io/celo-testnet/geth:$USER .`. Here the image being build is `gcr.io/celo-testnet/geth` and it is tagged with your username. You can use any other username as well as long as it is unique.
4. Push that to Google Cloud registry - `docker push gcr.io/celo-testnet/geth:$USER`
5. Now in celo-monorepo, modify `.env` file, change `GETH_NODE_DOCKER_IMAGE_TAG` to the tag from step 2.
6. If you want to change bootnode, which normally isn't required then, in `.env` file, change `GETH_BOOTNODE_DOCKER_IMAGE_REPOSITORY` tag to `gcr.io/celo-testnet/geth` and `GETH_BOOTNODE_DOCKER_IMAGE_TAG` to the tag from step 2.
7. Now, start your testnet as described earlier in this document.

### How to publish new npm packages

#### Test

```
# For testing contractkit package before publishing it.
# To test utils, change packages/contractkit to packages/utils
# To test cli, change packages/contractkit to packages/cli
$ docker run -v $PWD/packages/contractkit:/tmp/npm_package -it --entrypoint bash node:8
# Inside docker
$ cd /tmp && yarn add /tmp/npm_package
```

### Publish

Packages whose name starts with prefix `@celo/` are scoped. They can only be published by those who are member of [celo](https://www.npmjs.com/settings/celo/members) group. All of our packages are scoped.

```
packages/contractkit $ yarn publish --access=public
```

### How to test Circle CI jobs locally before deploying

`brew cask install docker && brew install circleci` - one time setup
Now use `circleci config validate` to validate the config file at `$PWD/.circleci/config.yml`. The checks are basic and can still miss bugs.

1. Start Docker
2. Use `circleci local execute --job test-npm-package-install` to run `test-npm-package-install` job locally on Circle CI.
3. Local execution does not support workflows, so, `install_dependencies` step would be skipped. To emulate access to the monorepo, mount the celo-monorepo dir to workspace directory (`/home/circleci/app`).

```
celo-monorepo $ circleci local execute --job test-npm-package-install -v $PWD:/home/circleci/app
```

### How to access [Bulldozer](https://github.com/palantir/bulldozer) deployment for Celo

Instance on GCP: [https://console.cloud.google.com/compute/instancesDetail/zones/us-west1-b/instances/ashishb-bulldozer-testing](https://console.cloud.google.com/compute/instancesDetail/zones/us-west1-b/instances/ashishb-bulldozer-testing)
SSH: `gcloud beta compute --project "celo-testnet" ssh --zone "us-west1-b" "ashishb-bulldozer-testing"` and then `screen -r`
GitHub app: [https://github.com/apps/bulldozer-deployment-for-celo](https://github.com/apps/bulldozer-deployment-for-celo)
Bulldozer was built and started from source code using `cd /home/ashishb/go/src/bulldozer && ./godelw run bulldozer server | tee /tmp/bulldozer.log` and it uses ashishb's auth token for authentication for now.


#### Two Limitations of Bulldozer

1. It won't "update" unless the code changes. So, adding a new label won't cause it to trigger the rebase. This is a [known issue](https://github.com/palantir/bulldozer#bulldozer-isnt-updating-my-branch-when-it-should-what-could-be-happening)
2. Bulldozer has no UI or status updates. There is no indication that it is working on a PR except in Bulldozer logs. Another [known issue](https://github.com/palantir/bulldozer/issues/70).

### Docker cleanup

`docker image prune` to prune dangling images and `docker image prune -a` to prune images with no associated containers.

`docker system prune` is even more aggressive - see [https://stackoverflow.com/a/36584376](https://stackoverflow.com/a/36584376)

### PR submitted by outsiders in Celo repo

```
gh pr | sort | uniq | grep '@' | grep -v -E '(@MarkusBerlin|@PratyushRT|@aaronmgdr|@anikaraghu|@annakaz|@arielgabizon|@asaj|@ashishb|@bohan-zhang|@bricewang|@catgu5|@celo-ci-bot-user|@celogitbook|@cla-bel|@cmcewen|@coreycelo|@drklee3|@flashman1234|@isabellewei|@jansel|@jarmg|@jeanregisser|@jmrossy|@kevjue|@kobigurk|@m-chrzan|@marekolszewski|@martinvol|@mcortesi|@medhak1|@misterz1977|@moisternw|@mstraka100|@nambrot|@nicholasguo|@nityas|@nvesely|@nvtaveras|@rcroessmann|@rohit-dua|@sallyjyl|@shellygr|@sissnad|@stigi|@timmoreton|@tjlahey|@tkporter|@trianglesphere|@tromer|@wilcoxjay|@witoff|@yerdua|@yorhodes)'
```

## Geth

### How to build geth for Mac OS

1. `make -j geth` or,
2. `celotooljs geth build` ([celotooljs](https://github.com/celo-org/celo-monorepo/tree/master/packages/celotool))

### How to build geth for Android

```
NDK_VERSION=android-ndk-r19c ANDROID_NDK=ndk_bundle/android-ndk-r19c make ndk_bundle ; NDK_VERSION=android-ndk-r19c ANDROID_NDK=ndk_bundle/android-ndk-r19c make -j android
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

### How to debug celocli

Put `DEBUG=*` prefix before calling `yarn run celocli` to print detailed logging.

## Mobile

### How to build a release binary for Android - signed by debug signatures

```
# First, go to celo-monorepo/packages/mobile/android directory,
# Copy your personal debug.keystore to fake a release key signer
packages/mobile/android $ cp ~/.android/debug.keystore celo-release-key.keystore

# Build it
packages/mobile/android $ CELO_RELEASE_STORE_PASSWORD='' ./gradlew assembleRelease --info

# This produces an unsigned (I am not sure why?) apk in "~/celo/celo-monorepo/packages/mobile/android/app/build/outputs/apk/release/app-release-unsigned.apk"
# Sign it using apksigner
packages/mobile/android $ java -jar apksigner.jar sign --ks ~/.android/debug.keystore --ks-key-alias androiddebugkey ~/celo/celo-monorepo/packages/mobile/android/app/build/outputs/apk/release/app-x86-release-unsigned.apk # (Password: android)

# Install
packages/mobile/android $ adb install app/build/outputs/apk/release/app-x86-release-unsigned.apk  # (usually, use x86 for emulator and arm for mobile device)
```

### Typescript debugging tips

1. Use `JSON.stringify()` to print objects. It won't work for circular objects, use `util.inspect()` for those objects.
