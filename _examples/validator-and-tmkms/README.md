# Validator and TMKMS

This example outlines how to run a node on Akash, connected to a TMKMS signer hosted elsewhere. This allows you to host the sensitive key material anywhere you trust with TMKMS, while running the actual blockchain node on Akash. It also allows quick and easy switching of the node used for signing, either between Akash deployments or between an Akash deployment and a node running elsewhere.

The ideal scenario would be as follows:

- TMKMS hosted on a secure server you trust. This could be at your home/office if you have reliable internet and servers, or in a data center you trust.
- A node running on Akash, which the TMKMS signer connects to to sign blocks.
- A backup node running on Akash, which the TMKMS signer can be switched to manually if required
- A second backup node running in another data center which can be used as a secondary backup option.

## Prerequisites

You should have a clear understanding of how TMKMS works. Ideally you run a validator already, and have already switched to TMKMS for your operations. Alternatively you run a validator but don't use TMKMS yet; in which case it would be best for you to setup TMKMS with your existing setup first, before migrating to Akash.

Some excellent guides on using TMKMS can be found at the following links:

- [TMKMS docs](https://github.com/iqlusioninc/tmkms)
- [Osmosis TMKMS setup guide](https://docs.osmosis.zone/developing/keys/tmkms.html#setup-tmkms)
- [Lavender Five notes](https://gist.github.com/dylanschultzie/c7c4eed531df0f004a50c5395e1604b3)
- [King Nodes Cosmos Tools guide](https://github.com/nullmames/cosmos-tools/tree/main/tmkms)

Once you have this setup, you should have a TMKMS signer, connected to an existing node. You should understand how to change the node the signer is connecting to, and you should have secured your key material.

## Caveats

There are currently two main caveats to using TMKMS with nodes hosted on Akash. These will be resolved with updates in the near future.

1. Currently a deployment using Persistent Storage cannot be updated. This presents an issue with TMKMS as when you set the `priv-validator-laddr` config, the node will not start until the signer is connected. This means you will be missing blocks while you wait for the node to sync, since you can't sync the node _then_ update the deployment to enable remote signing. Deployments that do not use Persistent Storage can update their deployment by changing the image.
2. Right now Akash cannot restrict port access to certain IP addresses. Ideally the RPC and `priv-validator-laddr` ports would be restricted to your signer and monitoring IP addresses. A future version of this guide will include an nginx proxy container to provide these IP restrictions, once an update has been rolled out to providers allowing access to the user's IP address.

Both of these issues should be resolved in the near future.

## Process

1. Spin up a node on Akash with persistent storage, using the [example deploy.yml](./deploy.yml). Note that the `priv-validator-laddr` config is already configured, which means the node won't start until the signer is connected. Also note we use Statesync here to ensure the node starts as quickly as possible.
2. Obtain the URL to access the `26658` remote signer port from the deployment.
3. Update KMS to point to the address obtained in #2. You will stop signing, but the node should recognise the connection and start to statesync.
4. Once the node has synced state and caught up with the tip of the blockchain, you will see TMKMS start to sign.
5. You are now signing blocks using a node hosted on Akash, with your key material secured in TMKMS.

Note the [example deploy.yml](./deploy.yml) uses Persistent Storage to ensure any container restarts retain the storage. This may or may not be necessary depending on your use case.