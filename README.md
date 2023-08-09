# Flow Wallet API Demo
This repository demonstrates basic usage of [Wallet API](https://github.com/flow-hydraulics/flow-wallet-api/) using Docker.

## Setup

**Note:** The [.env.testnet](./.env.testnet) file in this repository is already configured with an admin account on testnet. You can skip to the next session to get started.

Wallet API needs an admin account to create more accounts. We'll use the testnet faucet to create an account and update [.env.testnet](./.env.testnet).

```bash
flow keys generate
```

Go to https://faucet.flow.com/ to create and fund an account using the generated public key.

Configure [.env.testnet](./.env.testnet):

```bash
FLOW_WALLET_ADMIN_ADDRESS=<put address here>
FLOW_WALLET_ADMIN_PRIVATE_KEY=<put private key here>
```

## Running Wallet API

Wallet API is available as a Docker image. The [docker-compose.yml](./docker-compose.yml) file bundles it with a database. It also references an environment file ([.env.testnet](./.env.testnet)) as configuration.

To run:

```bash
docker-compose up -d
```

Verify:

```bash
docker ps
```

Check out [here](https://github.com/flow-hydraulics/flow-wallet-api/blob/main/configs/configs.go) for list of config options.

## Endpoints

Check out [API Documentation](https://flow-hydraulics.github.io/flow-wallet-api/) for list of endpoints.

### Version

```bash
curl http://localhost:3000/v1/debug
```

### Create New Account

```bash
curl -X POST "http://localhost:3000/v1/accounts" -H "Idempotency-Key: 1"
```

`Idempotency-Key` should be unique for each new API call. Since Wallet API is non-blocking by default, the above call will return a `jobId` that can be used to fetch the request's status:

```bash
curl http://localhost:3000/v1/jobs/160a2510-c0ef-467d-ae01-ad5a3b1ba616
```

When the job is `COMPLETE` the new account's address will be available in the `result` field.

You can also list all accounts:

```bash
curl http://localhost:3000/v1/accounts
```

**Note:** You can add `?sync=1` to the end of the URL to make a blocking transaction request. This will skip the job system to return a result.

### Transactions

You can send transactions from custodied accounts:

```bash
curl -X POST "http://localhost:3000/v1/accounts/0x33b6a9a32ae5cded/transactions" \
    -H "Content-Type: application/json" \
    -H "Idempotency-Key: 2" \
    -d '{"code":"transaction{prepare(a: AuthAccount){}}"}'
```

Remember to replace the account address with a valid custodial account. In response, a job will be created so you can check the result asynchronously. Transaction hash is also returned in `transactionId`.

```bash
curl http://localhost:3000/v1/jobs/278de8cb-14b2-413e-bf04-8a83b99c0338
```

List an accounts transactions:

```bash
curl http://localhost:3000/v1/accounts/0x33b6a9a32ae5cded/transactions
```

```bash
curl -X POST "http://localhost:3000/v1/accounts/0x33b6a9a32ae5cded/transactions?sync=1" \
    -H "Content-Type: application/json" \
    -H "Idempotency-Key: 3" \
    -d '{"code":"transaction{prepare(a: AuthAccount){}}"}'
```

**Note:** You can add `?sync=1` to the end of the URL to make a blocking transaction request. This will skip the job system to return a result.
