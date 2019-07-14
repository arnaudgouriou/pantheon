# Pantheon Enterprise Ethereum Client

[Pantheon](https://docs.pantheon.pegasys.tech/en/stable/) is an open-source [Ethereum](https://www.ethereum.org/) client developed under the Apache 2.0 license and written in Java. 
It runs on the Ethereum public network, private networks, and test networks such as Rinkeby, Ropsten, and Görli. 
Pantheon implements Proof of Work (Ethash) and Proof of Authority ([IBFT 2.0](https://docs.pantheon.pegasys.tech/en/latest/Consensus-Protocols/IBFT/) and [Clique](https://docs.pantheon.pegasys.tech/en/latest/Consensus-Protocols/Clique/)) consensus mechanisms.

You can use Pantheon to develop enterprise applications requiring secure, high-performance transaction processing in a private network.

Pantheon supports enterprise features including privacy and permissioning.

## Introduction
This chart deploys a private Ethereum network with PoA (IBFT 2.0) consensus onto a Kubernetes cluster using `Helm Chart` and `helmfile`. 
For further information on running a private network, refer to [Pantheon's documentation](https://docs.pantheon.pegasys.tech/en/stable/). 

In IBFT 2.0 networks, transactions and blocks are validated by approved accounts, known as validators. 
Validators take turns to create the next block. Existing validators propose and vote to add or remove validators.

Minimum Number of Validators IBFT 2.0 requires **4 validators** to be Byzantine fault tolerant.

This charts deploy 3 components:
* genesis Job: used to generate the genesis file and key-pairs associated 
* bootnode: used for Pantheon node discovery
* validator: validator Pantheon node for the IBFT 2.0 consensus protocol

## Prerequisites
- [Kubernetes](https://kubernetes.io/) 1.12+
- [Helm](https://helm.sh/docs/)
- [helmfile](https://github.com/roboll/helmfile)

## Installing the Chart
To install the chart in the namesapce with the name `my-namespace`:

`helmfile -n my-namespace -f helmfile.yaml apply`

The command deploys multi Pantheon nodes in PoA (IBFT 2.0) on the Kubernetes cluster in the default configuration. 
The configuration section lists the parameters that can be configured during installation.

> **Tip**: If there are problems to deploy, update your `Helm` and your `helmfile`

> **Tip**: List all releases using `helm list`

## Uninstalling the Chart
To uninstall/delete the deployment:

`helmfile -n my-namespace -f helmfile.yaml delete --purge`

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

Please have a look at [node-values.yaml](pantheon-node/values.yaml) and [genesis-values.yaml](pantheon-genesis/values.yaml) 
to show all parameters.

The following table lists the configurable parameters of the **Pantheon genesis** chart and their default values.

Parameter | Description | Default
--------- | ----------- | -------
`image.repository` | Container image repository | `pegasyseng/pantheon`
`image.tag` | Container image tag | `1.1.3`
`image.pullPolicy` | Container image pull policy | `IfNotPresent`
`validators.generated` | If true, generate automatically the key-pairs for validators | `true`
`config.genesis.blockchain.nodes.generate` | If true, generate the number of key-pairs | `true`
`config.genesis.blockchain.nodes.count` | The number of key-pairs generated for validators and inject into extraData | `4`
`config.genesis.blockchain.nodes.keys` | The list of private key to inject into extraData | `none`
`config.genesis.config.chainId` | The identifier of the private Ethereum network | `1981`
`config.genesis.config.constantinoplefixblock` | In private networks, the milestone block defines the protocol version for the network | `0`
`config.genesis.config.ibft2.blockperiodseconds` | Minimum block time in seconds. | `2`
`config.genesis.config.ibft2.epochlength` | Number of blocks after which to reset all votes. | `30000`
`config.genesis.config.ibft2.requesttimeoutseconds` | Timeout for each consensus round before a round change. | `10`
`config.genesis.config.extraData` | The extraData property is RLP encoded. | `0x`
`config.genesis.config.nonce` |  | `0x0`
`config.genesis.config.timestamp` |  | `0x58ee40ba`
`config.genesis.config.gasLimit` | Set the block size limit (measured in gas) | `0x47b760`
`config.genesis.config.difficulty` | Specify a fixed difficulty in private networks | `0x0`
`config.genesis.config.mixHash` | Hash for Istanbul block identification (IBFT 2.0). | `0x63746963616c2062797a616e74696e65206661756c7420746f6c6572616e6365`
`config.genesis.config.coinbase` | The coinbase address is the account to which mining rewards are paid. | `0x0000000000000000000000000000000000000000`
`config.genesis.config.alloc` | Predeploy contracts when starting Pantheon with Ether | `{config.genesis.config.alloc}`


The following table lists the configurable parameters of the **Pantheon node** chart and their default values.


Parameter | Description | Default
--------- | ----------- | -------
`image.repository` | Container image repository | `pegasyseng/pantheon`
`image.tag` | Container image tag | `1.1.3`
`image.pullPolicy` | Container image pull policy | `IfNotPresent`
`pantheon.genesis.name` | IMPORTANT: The name of the configMap to retrieve the genesis | `genesis-pantheon`
`pantheon.bootnode.enabled` | If true, the Pantheon node deployed will be a bootnode | `false`
`pantheon.bootnode.privKey` | the Pantheon bootnode private key. If not present, the key is automatically generated | ``
`pantheon.validators.enabled` | If true, the Pantheon node deployed will be a validator | `false`
`pantheon.validators.privKey` | the Pantheon validator private key. If not present, retrieve the key from genesis chart  | ``
`index` | The number of the validator (override by helmfile) | `0`
`replicaCount` | Warning: Should stay at this default value.  | `1`
`service.type` | Kubernetes service type | `ClusterIP`
`pantheon.persistentVolume.enabled` | If true, it's claim a persistent Volume | `false`
`pantheon.persistentVolume.size` | Size of the Volume | `2Gi`
`pantheon.persistentVolume.storageClass` | Storage class of the Volume | ``

For the other default parameters, see [node-values.yaml](pantheon-node/values.yaml) 

### Modify the number of validators
To modify the number of validators, you need to change values in two places.

Into the file `helmfile.yaml`, set (copy/past) the release section. 
You have to modify the `name` and the `index` value:

```yaml
  - name: validator-<INDEX_NUMBER>
    labels:
      component: validators
    namespace: {{ .Namespace }}
    chart: ./pantheon-node
    values:
      - ./values/validator.yaml
    set:
      - name: index
        value: <INDEX_NUMBER>
```

Into the file `values/genesis.yaml`, change the `count` number to specify how many validators you want in the `genesis` file:

```yaml
config:
  blockchain:
    nodes:
      count: <NUMBER_OF_VALIDATORS>
```
