# Safe Core SDK migration

The package formerly known as `Safe Core SDK` has been refactored as the `protocol-kit`. With this refactor the aim is to add more tools on a logical way to make the Safe Core SDK to grow and enable new functionality for developers. Even many things remain the same, we took the chance to make deeper changes that will need from developer action in order to be compatible with the new package form.

## Protocol Kit v0.1 migration

On the first release (v0.1) of the `protocol-kit` the breaking changes were kept to the minimum, being the biggest ones on how to handle the `imports`. All the functionality remains the same as on the former Safe Core SDK.


### Migration steps

First of all install the new dependency and remove the deprecated dependencies

```bash
yarn add @safe-global/protocol-kit@0.1
yarn remove @safe-global/safe-core-sdk @safe-global/safe-ethers-lib @safe-global/safe-web3-lib
```

Notice that `safe-ethers-lib`and `safe-web3-lib` packages are not necessary any more. Now these libs can be exported from the `protocol-kit` package. This is the only change you should find when using `protocol-kit@v0.1`.

```typescript
import Safe, { SafeFactory } from '@safe-global/safe-core-sdk'

import EthersAdapter from '@safe-global/safe-ethers-lib' // You will have this one if you use Ethers

import Web3Adapter from '@safe-global/safe-web3-lib' // You will have this one if you use web3js
```

Your previous configuration may look like the above. You need to update it to the one below

```typescript
import Safe, { EthersAdapter, SafeFactory } from '@safe-global/protocol-kit' // If you use Ethers

import Safe, { Web3Adapter, SafeFactory } from '@safe-global/protocol-kit' // If you use web3js
```


## Protocol Kit v1.0 migration

As the package is big and widely used, we decided to split the migration in two steps to minimize inconvenience to developers. If you are already using the formerly known as Safe Core SDK, we recommend that you first install `protocol-kit@v0.1` and follow the migration steps above. Once it's done you can continue migrating to v1.0. In case something is not working as you would expect with `protocol-kit@v1.0` you can roll back to v0.1 without having to rollback `import` changes again.

### Migration steps

First of all update your `protocol-kit` dependency and make sure to also update the `types` lib to v2.

```bash
yarn add @safe-global/protocol-kit@1.0 @safe-global/safe-core-sdk-types@2
```

If you are using the `types` library, once you rebuild your project should start getting some guidance on the necessary changes. Here is a summary of those you may find:

#### Safe `getAddress` method is now async

The `getAddress` method from the Safe object (safeSdk.getAddress()) is now async

```typescript
import Safe from '@safe-global/protocol-kit'

const safeSdk = await Safe.create({ ethAdapter, safeAddress })

// Before
const safeAddress = safeSdk.getAddress()

// After
const safeAddress = await safeSdk.getAddress()
```

#### SafeFactory `deploySafe` method

The method now expects to receive the `saltNonce` parameter directly, instead inside a `SafeDeploymentConfig` object

 ```typescript
import { SafeFactory } from '@safe-global/protocol-kit'

const safeFactory = await SafeFactory.create({ ethAdapter })

 // Before
const safeDeploymentConfig: SafeDeploymentConfig = { saltNonce }
const safeSdk = await safeFactory.deploySafe({ safeAccountConfig, safeDeploymentConfig })

// After
const saltNonce = '<YOUR_CUSTOM_VALUE>'
const safeSdk = await safeFactory.deploySafe({ safeAccountConfig, saltNonce })
 ```

#### Gas properties are now typed as strings

In case you are setting `safeTxGas`, `baseGas` or `gasPrice` on your app you would be using Javascript numbers. Now these values should be set as strings.
In case you are using the `estimageGas` method, now you can expect it to return a string instead a Javascript number.

#### Async methods on EthersAdapter and Web3Adapter

Independent of the Adapter you may be using the following methods are async now on both:
 - getSafeContract
 - getSafeProxyFactoryContract
 - getMultiSendContract
 - getMultisendCallOnlyContract
 - getCompatibilityFallbackHandlerContract
 - getSingMessageLibContract
 - getCreateCallContract
 - estimateGas

### New functionality on v1.0

#### The Safe SDk instance can now receive a `predictedSafe` config.

