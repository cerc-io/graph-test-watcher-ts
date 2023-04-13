# Example Watcher

## Setup

First try the [stack orchestrator](https://github.com/cerc-io/stack-orchestrator) to quickly get started. Advanced users can see [here](https://github.com/cerc-io/watcher-ts/tree/main/docs/README.md) for instructions on setting up a local environment by hand.

Run the following command to install required packages:

```bash
yarn
```

Clone watcher-ts repo:

```bash
git clone git@github.com:cerc-io/watcher-ts.git
```

## Run

* Change directory to [watcher-ts/packages/graph-node](https://github.com/cerc-io/watcher-ts/tree/main/packages/graph-node)

* Deploy an `Example` contract:

  ```bash
  yarn example:deploy
  ```

* Set the returned address to the variable `$EXAMPLE_ADDRESS`:

  ```bash
  EXAMPLE_ADDRESS=<EXAMPLE_ADDRESS>
  ```

* In [watcher-ts/packages/graph-node/test/subgraph/example1/subgraph.yaml](https://github.com/cerc-io/watcher-ts/tree/main/packages/graph-node/test/subgraph/example1/subgraph.yaml):

    * Set the source address for `Example1` datasource to the `EXAMPLE_ADDRESS`.
    * Set the `startBlock` less than or equal to the latest mined block.

* Build the example subgraph:

  ```bash
  yarn build:example
  ```

* Change directory to [graph-test-watcher-ts](./)

* Set the subgraphPath in [environments/local.toml](./environments/local.toml) to filepath of `watcher-ts/packages/graph-node/test/subgraph/example1/build`

  If watcher-ts is cloned in `/home/dev` directory:

  ```toml
  [server]
    subgraphPath = "/home/dev/watcher-ts/packages/graph-node/test/subgraph/example1/build"
  ```

* In a new terminal, run the job-runner:

  ```bash
  yarn job-runner
  ```

* In an another terminal, run the watcher:

  ```bash
  yarn server
  ```

* The output from the block handler in the mapping code should be visible in the `job-runner` for each block.

* Run the following GQL subscription at the graphql endpoint http://127.0.0.1:3008/graphql

  ```graphql
  subscription {
    onEvent {
      event {
        __typename
        ... on TestEvent {
          param1
          param2
        },
      },
      block {
        number
        hash
      }
    }
  }
  ```

* Change directory to [watcher-ts/packages/graph-node](https://github.com/cerc-io/watcher-ts/tree/main/packages/graph-node)

* Trigger the `Test` event by calling an example contract method:

  ```bash
  yarn example:test --address $EXAMPLE_ADDRESS
  ```

  * A `Test` event shall be visible in the subscription at endpoint.

  * The subgraph entity `Category` should be updated in the database.

  * An auto-generated `diff-staged` entry `State` should be added.

* Run the query for entity in at the endpoint:

  ```graphql
  query {
    category(
      block: { hash: "EVENT_BLOCK_HASH" },
      id: "1"
    ) {
      __typename
      id
      count
      name
    }
  }
  ```

* Run the `getState` query at the endpoint to get the latest `State` for `EXAMPLE_ADDRESS`:

  ```graphql
  query {
    getState (
      blockHash: "EVENT_BLOCK_HASH"
      contractAddress: "EXAMPLE_ADDRESS"
      # kind: "checkpoint"
      # kind: "diff"
      kind: "diff_staged"
    ) {
      cid
      block {
        cid
        hash
        number
        timestamp
        parentHash
      }
      contractAddress
      data
    }
  }
  ```

* `diff` states get created corresponding to the `diff_staged` states when their respective blocks reach the pruned region.

* Change directory to [graph-test-watcher-ts](./)

* After the `diff` state has been created, create a `checkpoint`:

  ```bash
  yarn checkpoint create --address $EXAMPLE_ADDRESS
  ```

  * A `checkpoint` state should be created at the latest canonical block hash.

  * Run the `getState` query again at the endpoint with the output `blockHash` and kind `checkpoint`.

* All the `State` entries can be seen in `pg-admin` in table `state`.
