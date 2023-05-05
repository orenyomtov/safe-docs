# Counterfactual deployment

Because on how the Safe deployment takes place using `CREATE2` opcode, it's possible considering the deployment configuration to anticipate the address where a Safe will be deployed. This can enable interesting flows like deploying a Safe only when the user is ready to interact.

## Using a Safe before deploying it

Instead of directly deploying the Safe, lets use the `protocol-kit` first to prepare some transactions that can be executed once the Safe is deployed. Let's start initializing the `protocol-kit` instance from the configuration.

```typescript
import { ethers } from 'ethers'
import Safe, {
  EthersAdapter,
  predictSafeAddress,
  SafeAccountConfig,
  SafeFactory
} from '@safe-global/protocol-kit'

const provider = new ethers.providers.JsonRpcProvider('...')
const signerWallet = new ethers.Wallet('<PRIVATE_KEY>', provider)
const ethAdapter = new EthersAdapter({ ethers, signerOrProvider: signerWallet })

// Prepare the Safe deployment configuration
const signerAddress = await signerWallet.getAddress()
const owners = [signerAddress]
const threshold = 1

const safeAccountConfig: SafeAccountConfig = {
    owners,
    threshold
}

// Predict the Safe address based on the given configuration
const safeAddress = await predictSafeAddress({
  ethAdapter,
  safeAccountConfig,
  safeDeploymentConfig
})

const safeSdk = await Safe.create({ ethAdapter, predictedSafe })
```

Now that we already have a `protocol-kit` instance let's prepare some some transactions before the Safe is deployed.

```typescript
import {
  getSafeContract
} from '@safe-global/protocol-kit'

// FIXME check if this would work with Safes pending to be deployed
const safeContract = await getSafeContract({
  ethAdapter: ethAdapter,
  safeVersion,
  customSafeAddress: safeAddress
})

const standardizedSafeTx = await safeSdk.createTransaction(
  safeTransactionData,
  options
)

const signedSafeTx = await safeSdk.signTransaction(standardizedSafeTx)

const transactionData = safeContract.encode('execTransaction', [
  signedSafeTx.data,
  signedSafeTx.encodedSignatures()
])
```

Finally lets send the deployment and the first transactions to the blockchain

```typescript
import {
  getMultiSendCallOnlyContract
} from '@safe-global/protocol-kit'

const multiSendCallOnlyContract = await getMultiSendCallOnlyContract({
  ethAdapter: this.#ethAdapter,
  safeVersion
})

const safeSingletonContract = await getSafeContract({
  ethAdapter: this.#ethAdapter,
  safeVersion
})

const initializer = await encodeSetupCallData({
  ethAdapter: ethAdapter,
  safeContract: safeContract,
  safeAccountConfig: predictedSafe.safeAccountConfig
})

const safeProxyFactoryContract = await getProxyFactoryContract({
  ethAdapter: ethAdapter,
  safeVersion
})

const safeDeploymentTransaction: MetaTransactionData = {
  to: safeProxyFactoryContract.getAddress(),
  value: '0',
  data: encodeCreateProxyWithNonce(
    safeProxyFactoryContract,
    safeSingletonContract.getAddress(),
    initializer
  ),
  operation: OperationType.Call
}
const safeTransaction: MetaTransactionData = {
  to: safeAddress,
  value: '0',
  data: transactionData,
  operation: OperationType.Call
}

const multiSendData = encodeMultiSendData([safeDeploymentTransaction, safeTransaction])
encodedTransaction = multiSendCallOnlyContract.encode('multiSend', [multiSendData])
```