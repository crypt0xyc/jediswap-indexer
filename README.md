# JediSwap Indexer and GraphQL API with Apibara

This repository shows how to use Apibara to index JediSwap - a decentralized exchange (DEX) on StarkNet - and build a GraphQL API to access the data.

The indexer is separated from the API server, so that it becomes easier to deploy and scale in a production environment. There should always only be a single instance of the indexer running, but there can be multiple API servers side by side to serve a larger number of users.

The project uses:
 * [Apibara Python SDK](https://www.apibara.com/docs/python-sdk) for receiving StarkNet data.
 * MongoDB for storage.
 * [Strawberry GraphQL](https://strawberry.rocks/) for the GraphQL API server.


## Setting up

Create a new virtual environment for this project. While this step is not required, it is _highly recommended_ to avoid conflicts between different installed packages.

    python3 -m venv .venv

Then activate the virtual environment.

    source .venv/bin/activate

Then install `poetry` and use it to install the package dependencies.

    python3 -m pip install poetry
    poetry install

Start MongoDB using the provided `docker-compose` file:

    docker-compose up

Note: You can use any managed MongoDB like MongoDB Atlas.


## Getting started

This example shows how to index JediSwap on StarkNet. You can change it to index any Uniswap-V2-like DEX by changing the configuration in `src/swap/indexer/jediswap.py`.

Once setup, start indexing by running the following command:

```sh
poetry run swap-indexer indexer \
    --stream-url <stream-url> \
    --mongo-url <mongo-url> \
    --rpc-url <rpc-url>
```

where:

 * `<stream-url>`: the Apibara stream, should be `mainnet.starknet.stream.apibara.com` (if that doesn't work, try `mainnet.starknet.a5a.ch`)
 * `<mongo-url>`: mongodb connection url. If you use the provided docker compose file, use `mongodb://apibara:apibara@localhost:27017`.
 * `<rpc-url>`: the StarkNet RPC url. You can use Infura for this.

Note: Instead of supplying the arguments through the CLI, they can also be passed as environment variables (requires only slight changes to the code).

The indexer will then start indexing your DEX.

Start the GraphQL API server with:

```sh
poetry run swap-indexer server \
    --mongo-url <mongo-url>
```

where `<mongo-url>` is the same connection string you used above.
The GraphQL API is served at `http://localhost:8000/graphql` and
it includes a GrapihQL page for testing the API.


## Production deployment & scaling

The repository contains a `docker-compose.prod.yml` file with a good starting point for a production deployment.

Here are some changes you should be considering:

 * Use a managed MongoDB, for example MongoDB Atlas.
 * Run the indexer and the API server on separate machines.


As the number of users grows, you may need to scale your API server:

 * Deploy multiple instances of the API server behind a load balancer.
 * Replicate the database and connect the API servers to the replicas.
 * Cache entities with e.g. redis.


## License


   Copyright 2022 GNC Labs Limited

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
