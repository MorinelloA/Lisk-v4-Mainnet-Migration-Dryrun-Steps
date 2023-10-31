# Lisk v4 Mainnet Migration Dryrun Steps

## Setup a new server with Lisk Core v3.1.0 node against the mainnet

1. Please follow the instructions to setup a new server with Lisk Core v3.1.0:
    - [https://lisk.com/documentation/lisk-core/setup/npm.html](https://lisk.com/documentation/lisk-core/setup/npm.html)
2. Kindly ensure that the node config has the following property set and restarted:
    - `backup.height: 20504210`
    - Expected time of the block generation for the specified height is **Thursday, November 2, 2023 2:03:20 PM (CEST, Europe/Berlin)**

3. **Optionally,** to closely replicate the mainnet migration and to be able to use all the flags on the Lisk Migrator as you would in production and auto-generate the necessary commands to enable the validators on the migrated node **:**
   
    a. **On your original mainnet node in production**:
      - Copy your existing `forging.delegates` information from your config file to the new server
      - Export your mainnet forging information:
        - `lisk-core -v`
          - **Expected output:** `lisk-core/3.1.0 darwin-arm64 node-v18.16.1`
        - `lisk-core forger-info:export --output ./my-forger-info.json`
        - `lisk-core forging:status --pretty`
          - **Expected output:** `[{
             "address":"89aa5fc8861d392f60662f76a379cc348fe97d28",
             "forging":true,
             "height":670237,
             "maxHeightPrevoted":670159,
             "maxHeightPreviouslyForged":670187,
           }]`

    b. Copy the `forging.delegates` information from your config, my-forger-info.json file generated above, and exported forging information to your new server you setup in Step 1 above

    c. On your new server, where you intend to run the migrator
      - Copy the `forging.delegates` information into your node (custom) config file
      - Start the lisk-core v3.1.0 node with the (updated/custom) config file as usual
      - Import the forger information
        - **Commands** : `lisk-core forger-info:import --output ./my-forger-info.json`
      - However, ***DO NOT ENABLE FORGING*** on the new node to ensure you're not double forging and not get PoM'd!!!

    d. All the forging related information for Lisk Core v3 must be available in the documentation for your usage here: [https://lisk.com/documentation/run-blockchain/forging.html](https://lisk.com/documentation/run-blockchain/forging.html)

## Download and extract Lisk Migrator patch (On the new server)

1. Download the patched version of Lisk Migrator and the corresponding hash. Execute the following commands to download the appropriate build
  - `PLATFORM=$(uname | tr '[:upper:]' '[:lower:]')`
  - `ARCH=$(uname -m | sed 's/x86\_64/x64/')`
  - `curl -sSO https://lisk-mainnet-dryrun.fra1.cdn.digitaloceanspaces.com/lisk-migrator-v2.0.0-rc.4-dryrun-patch-$PLATFORM-$ARCH.tar.gz`
  - `curl -sSO https://lisk-mainnet-dryrun.fra1.cdn.digitaloceanspaces.com/lisk-migrator-v2.0.0-rc.4-dryrun-patch-$PLATFORM-$ARCH.tar.gz.SHA256`
2. Verify the integrity of the downloaded tarball
  - `shasum -a256 -c lisk-migrator-v2.0.0-rc.4-dryrun-patch-$PLATFORM-$ARCH.tar.gz.SHA256`
  - **Expected output:** `lisk-migrator-v2.0.0-rc.4-dryrun-patch-darwin-arm64.tar.gz: OK`
    - Your output might vary based on the PLATFORM and ARCH of your server
    - Run `echo $PLATFORM $ARCH` to confirm
3. Extract the tarball
  - `tar -xf lisk-migrator-v2.0.0-rc.4-dryrun-patch-$PLATFORM-$ARCH.tar.gz`
  - Should be extracted to `lisk-migrator` directory in your current directory
4. Change to the extracted directory
  - `cd lisk-migrator`
5. Verify the version
  - `./bin/lisk-migrator -v`
  - **Expected output:** `lisk-migrator/2.0.0-rc.4-dryrun-patch.18c5964 darwin-arm64 node-v18.16.1`

## Generate the new Mainnet genesis block (On the new server)

1. Please follow the steps in the Migration guide starting from **Section 2. Migration Steps**
  - [https://lisk.io/documentation/lisk-core/v4/management/migration.html#migration-steps](https://lisk.io/documentation/lisk-core/v4/management/migration.html#migration-steps)

- **IMPORTANT INFORMATION:**
  - When starting the migrator, kindly provide the following snapshot height:
    - `--snapshot-height 20504210`
  - The patched migrator will download Lisk Core `v4.0.0-rc.7-dryrun-patch` from Lisk's NPM registry to continue with the block generation
    - lisk-core -v
**Expected output:** `lisk-core/4.0.0-rc.7-dryrun-patch darwin-arm64 node-v18.16.1`

  - The number of initRounds is modified to **167** instead of the standard 587
    - This translates to a **bootstrap period** of approx **2 days** within which the validator BLS keys have to be registered

## Running Lisk Core against the migrated network (On the new server)

1. If you did not participate in the migration dry run or ran the migrator without the **--auto-start-lisk-core-v4** flag, and want to start running a new node on the migrated network, please specify the following flag when starting the node in addition to the ones you want to:
  - `--genesis-block-url=https://lisk-mainnet-dryrun.fra1.cdn.digitaloceanspaces.com/genesis_block.json.tar.gz`

**Example:** lisk-core start -n mainnet -c ./custom-config.json --genesis-block-url=https://lisk-mainnet-dryrun.fra1.cdn.digitaloceanspaces.com/genesis\_block.json.tar.gz

## Running Lisk Service against the migrated Mainnet (On the new server)

1. To run Lisk Service against the migrated mainnet, please follow the setup guide:
  - [https://liskhq.github.io/lisk-docs/lisk-service/0.7/setup/docker.html](https://liskhq.github.io/lisk-docs/lisk-service/0.7/setup/docker.html)
  - [https://liskhq.github.io/lisk-docs/lisk-service/0.7/setup/source.html](https://liskhq.github.io/lisk-docs/lisk-service/0.7/setup/source.html)
2. Before starting Lisk Service, **please ensure** that the following environment variables are set as specified below for Lisk Service to work properly:
  - **Blockchain Connector**
    - `GENESIS_BLOCK_URL='https://lisk-mainnet-dryrun.fra1.cdn.digitaloceanspaces.com/genesis_block.json.tar.gz'`
  - **Blockchain Indexer**
    - `MAINCHAIN_SERVICE_URL=https://mainnet-service.liskdev.net`

**OTHER INFORMATION:**

- Lisk Service against the migrated network will be available on [https://mainnet-service.liskdev.net/api/v3/network/status](https://mainnet-service.liskdev.net/api/v3/network/status)
