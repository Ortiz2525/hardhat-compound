# Compound V2 protocol deployment

## Installation

```
npm install @thenextblock/hardhat-compound
yarn add @thenextblock/hardhat-compound
```

The function `deployCompoundV2(CTokenDeployArg[], Signer): Promise<CompoundV2>`
deploys Comptroller, Interest rate models, SimplePriceOracle, and cTokens. cToken config is passed
in a form of `CTokenDeployArg`. Compound contract types are pre-generated by TypeChain.

```typescript
import { deployCompoundV2 } from '@thenextblock/hardhat-compound';

const cTokenDeployArgs = [
  {
    cToken: 'cUSDC',
    underlying: '0x1234...',
    underlyingPrice: '1000000000000000000000000000000',
    collateralFactor: '750000000000000000',
  },
];

const { cTokens, comptroller, interestRateModels, priceOracle } = await deployCompoundV2(
  cTokenDeployArgs,
  deployer
);
```

Full example is available at [https://github.com/thenextblock/hardhat-compound-example
](https://github.com/thenextblock/hardhat-compound-example).

## Usage in scripts

You can use this plugin in a Hardhat script to deploy.

```typescript

import { ethers } from "hardhat";
import { formatUnits, parseUnits } from "ethers/lib/utils";
import { assert } from "chai";
import { deployErc20Token, Erc20Token } from "@thenextblock/hardhat-erc20";
import { CTokenDeployArg, deployCompoundV2, Comptroller,} from "@thenextblock/hardhat-compound";

async function main() {

  const [deployer] = await ethers.getSigners();

  const UNI_PRICE = "25022748000000000000";
  const USDC_PRICE = "1000000000000000000000000000000";

  // Deploy USDC ERC20
  const USDC: Erc20Token = await deployErc20Token(
    {
      name: "USDC",
      symbol: "USDC",
      decimals: 6,
    }, deployer
  );

  // Deploy UNI ERC20
  const UNI: Erc20Token = await deployErc20Token(
    {
      name: "UNI",
      symbol: "UNI",
      decimals: 18,
    },deployer
  );

  const ctokenArgs: CTokenDeployArg[] = [
    {
      cToken: "cUNI",
      underlying: UNI.address,
      underlyingPrice: UNI_PRICE,
      collateralFactor: "500000000000000000", // 50%
    },
    {
      cToken: "cUSDC",
      underlying: USDC.address,
      underlyingPrice: USDC_PRICE,
      collateralFactor: "500000000000000000", // 50%
    },
  ];

  const { comptroller, cTokens, priceOracle, interestRateModels } =
  await deployCompoundV2(ctokenArgs, deployer);

  await comptroller._setCloseFactor(parseUnits("0.5", 18).toString());
  await comptroller._setLiquidationIncentive(parseUnits("1.08", 18));

  const { cUNI, cUSDC } = cTokens;

  console.log("Comptroller: ", comptroller.address);
  console.log("SimplePriceOralce: ", await comptroller.oracle());
  console.log("cUNI: ", cUNI.address);
  console.log("cUSDC: ", cUSDC.address);

}
...

```

[Full sample code](https://github.com/thenextblock/hardhat-compound-example/blob/main/scripts/sample.ts)

## Usage in tests

You can also use the deployCompoundV2 function from your Hardhat tests
[Full sample code](https://github.com/thenextblock/hardhat-compound-example/blob/main/test/index.ts)
