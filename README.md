# OpenZeppelin Foundry Upgrades

Foundry library for deploying and managing upgradeable contracts, which includes upgrade safety checks.

> **Warning**
> This repository contains experimental code. It is available as a technology preview and its functionality is subject to change. Breaking changes may be introduced at any point while it is in preview.

## Installing

Run these commands:
```
forge install OpenZeppelin/openzeppelin-foundry-upgrades
forge install OpenZeppelin/openzeppelin-contracts-upgradeable
```

Set the following in `remappings.txt`, replacing any previous definitions of these remappings:
```
@openzeppelin/contracts/=lib/openzeppelin-contracts-upgradeable/lib/openzeppelin-contracts/contracts/
@openzeppelin/contracts-upgradeable/=lib/openzeppelin-foundry-upgrades/src/
```

> **Note**
> The above remappings mean that both `@openzeppelin/contracts/` (including proxy contracts deployed by this library) and `@openzeppelin/contracts-upgradeable/` come from your installation of the `openzeppelin-contracts-upgradeable` submodule and its subdirectories, which includes its own transitive copy of `openzeppelin-contracts` of the same release version number. This format is needed for Etherscan verification to work. Particularly, any copies of `openzeppelin-contracts` that you install separately are NOT used.

## OpenZeppelin Defender integration

See [DEFENDER.md](DEFENDER.md)

## Version Limitations

This library only supports proxy contracts and upgrade interfaces from OpenZeppelin Contracts versions 5.0 or higher.

## Before Running

This library uses the [OpenZeppelin Upgrades CLI](https://docs.openzeppelin.com/upgrades-plugins/1.x/api-core) for upgrade safety checks, which are run by default during deployments and upgrades.

If you want to be able to run upgrade safety checks, the following are needed:
1. Install [Node.js](https://nodejs.org/).
2. Configure your `foundry.toml` to include build info and storage layout:
```
[profile.default]
build_info = true
extra_output = ["storageLayout"]
```
3. If you are upgrading your contract from a previous version, add the `@custom:oz-upgrades-from <reference>` annotation to the new version of your contract according to [Define Reference Contracts](https://docs.openzeppelin.com/upgrades-plugins/1.x/api-core#define-reference-contracts) or specify the `referenceContract` option when calling the library's functions.
4. Run `forge clean` before running your Foundry script or tests.
5. Include `--ffi` in your `forge script` or `forge test` command.

If you do not want to run upgrade safety checks, you can skip the above steps and use the `unsafeSkipAllChecks` option when calling the library's functions. Note that this is a dangerous option meant to be used as a last resort.

## Usage

Import the library in your Foundry scripts or tests:
```
import {Upgrades} from "openzeppelin-foundry-upgrades/Upgrades.sol";
```

Then call functions from [Upgrades.sol](src/Upgrades.sol) to run validations, deployments, or upgrades.

### Examples

Deploy a UUPS proxy:
```
address proxy = Upgrades.deployUUPSProxy(
    "MyContract.sol",
    abi.encodeCall(MyContract.initialize, ("arguments for the initialize function"))
);
```

Deploy a transparent proxy:
```
address proxy = Upgrades.deployTransparentProxy(
    "MyContract.sol",
    INITIAL_OWNER_ADDRESS_FOR_PROXY_ADMIN,
    abi.encodeCall(MyContract.initialize, ("arguments for the initialize function"))
);
```

Call your contract's functions as normal, but remember to always use the proxy address:
```
MyContract instance = MyContract(proxy);
instance.myFunction();
```

Upgrade a transparent or UUPS proxy and call an arbitrary function (such as a reinitializer) during the upgrade process:
```
Upgrades.upgradeProxy(
    transparentProxy,
    "MyContractV2.sol",
    abi.encodeCall(MyContractV2.foo, ("arguments for foo"))
);
```

Upgrade a transparent or UUPS proxy without calling any additional function:
```
Upgrades.upgradeProxy(
    transparentProxy,
    "MyContractV2.sol",
    ""
);
```

> **Warning**
> When upgrading a proxy or beacon, ensure that the new contract either has its `@custom:oz-upgrades-from <reference>` annotation set to the current implementation contract used by the proxy or beacon, or set it with the `referenceContract` option, for example:
> ```
> Options memory opts;
> opts.referenceContract = "MyContractV1.sol";
> Upgrades.upgradeProxy(proxy, "MyContractV2.sol", "", opts);
> // or Upgrades.upgradeBeacon(beacon, "MyContractV2.sol", opts);
> ```

Deploy an upgradeable beacon:
```
address beacon = Upgrades.deployBeacon("MyContract.sol", INITIAL_OWNER_ADDRESS_FOR_BEACON);
```

Deploy a beacon proxy:
```
address proxy = Upgrades.deployBeaconProxy(
    beacon,
    abi.encodeCall(MyContract.initialize, ("arguments for the initialize function"))
);
```

Upgrade a beacon:
```
Upgrades.upgradeBeacon(beacon, "MyContractV2.sol");
```

### Deploying and Verifying

Run your script with `forge script` to broadcast and deploy. See Foundry's [Solidity Scripting](https://book.getfoundry.sh/tutorials/solidity-scripting) guide.

> **Important**
> Include the `--sender <ADDRESS>` flag for the `forge script` command when performing upgrades, specifying an address that owns the proxy or proxy admin. Otherwise, `OwnableUnauthorizedAccount` errors will occur.

> **Note**
> Include the `--verify` flag for the `forge script` command if you want to verify source code such as on Etherscan. This will verify your implementation contracts along with any proxy contracts as part of the deployment.
