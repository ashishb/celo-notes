# Celo related notes

## Production

### How to deploy a new network?

```
export NETWORK_NAME=integration && celotooljs deploy initial testnet -e ${NETWORK_NAME} && celotooljs deploy initial contracts -e ${NETWORK_NAME}
```

### How to upgrade existing network?

```
export NETWORK_NAME=integration && celotooljs deploy upgrade testnet -e ${NETWORK_NAME} && celotooljs deploy upgrade contracts -e ${NETWORK_NAME}
```

## How to build geth for Android?

```
ANDROID_NDK=/usr/local/Caskroom/android-ndk/18/android-ndk-r18/ make android -j
```

Copy it over to monorepo to test the new geth with Android

```
cp -v ~/celo/geth/build/bin/geth.aar ~/celo/celo-monorepo2/node_modules/@celo/client/build/bin/geth.aar
```
