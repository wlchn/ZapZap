# ZapZap
WIP: Zap Any LP/tokens

todo 

- More pools and tokens/LPs
- ETH UNI
- Frontend interface

## Zap in JavaScript(TypeScript)

``` JavaScript
// ============================== ZAP ==================================

// zapInToken
//   token1 -> token2  // need approve
//   token1 -> lp      // need approve

// zapIn
//   bnb -> token1  // payable
//   bnb -> lp      // payable

// zapOut
//   token1 -> bnb           // need approve
//   lp -> token1 + token2   // need approve

// deposit
//   bnb -> wbnb

// withdraw
//   wbnb -> bnb

// ============================== ZAP FUNCTIONS==================================

// zapInToken
//   token1 -> token2  // need approve
//   token1 -> lp      // need approve
export const zapInToken = async (
  accountAddress: string,
  fromAddress: string,
  fromAmount: BigNumber,
  toAddress: string
): Promise<string> => {
  const contract = getCachedContract(_env.ZapBSCAddress, ZapBSCABI);
  const data = contract.methods
    .zapInToken(fromAddress, fromAmount.toFixed(), toAddress)
    .encodeABI();
  const transactionId = await sendCommonTransaction(
    _env.ZapBSCAddress,
    accountAddress,
    data
  );
  return transactionId;
};

// zapIn
//   bnb -> token1  // payable
//   bnb -> lp      // payable
export const zapIn = async (
  accountAddress: string,
  bnbAmount: BigNumber,
  toAddress: string
): Promise<string> => {
  const contract = getCachedContract(_env.ZapBSCAddress, ZapBSCABI);
  const data = contract.methods.zapIn(toAddress).encodeABI();
  const transactionId = await sendCommonTransaction(
    _env.ZapBSCAddress,
    accountAddress,
    data,
    "0x" + bnbAmount.toString(16)
  );
  return transactionId;
};

// zapOut
//   lp -> token1 + token2   // need approve
//   token1 -> bnb           // need approve
export const zapOut = async (
  accountAddress: string,
  fromAddress: string,
  fromAmount: BigNumber
): Promise<string> => {
  const contract = getCachedContract(_env.ZapBSCAddress, ZapBSCABI);
  const data = contract.methods
    .zapOut(fromAddress, fromAmount.toFixed())
    .encodeABI();
  const transactionId = await sendCommonTransaction(
    _env.ZapBSCAddress,
    accountAddress,
    data
  );
  return transactionId;
};

export const depositBNBToWBNB = async (
  accountAddress: string,
  bnbAmount: BigNumber
): Promise<string> => {
  const contract = getCachedContract(_env.WBNBAddress, WBNBABI, "WBNB");
  const data = contract.methods.deposit().encodeABI();
  const transactionId = await sendCommonTransaction(
    _env.WBNBAddress,
    accountAddress,
    data,
    "0x" + bnbAmount.toString(16)
  );
  return transactionId;
};

export const withdrawWBNBToBNB = async (
  accountAddress: string,
  bnbAmount: BigNumber
): Promise<string> => {
  const contract = getCachedContract(_env.WBNBAddress, WBNBABI, "WBNB");
  const data = contract.methods.withdraw(bnbAmount.toFixed()).encodeABI();
  const transactionId = await sendCommonTransaction(
    _env.WBNBAddress,
    accountAddress,
    data
  );
  return transactionId;
};

export const zap = async (
  accountAddress: string,
  fromZapTokenName: string,
  toZapTokenName: string, // lp can be empty
  amount: BigNumber
): Promise<string> => {
  const fromZapToken = _env.zapTokens.find(z => z.name === fromZapTokenName);
  const toZapToken = _env.zapTokens.find(z => z.name === toZapTokenName);
  let txId = "";
  if (fromZapToken !== undefined) {
    if (fromZapToken.type === "lp") {
      // lp -> token1 + token2
      txId = await zapOut(accountAddress, fromZapToken.address, amount);
    } else if (toZapToken !== undefined) {
      if (fromZapToken.type === "bnb") {
        if (toZapTokenName === "WBNB") {
          // bnb -> wbnb
          txId = await depositBNBToWBNB(accountAddress, amount);
        } else {
          // bnb -> token1
          // bnb -> lp
          txId = await zapIn(accountAddress, amount, toZapToken.address);
        }
      }
      if (fromZapToken.type === "token") {
        if (toZapToken.type === "bnb") {
          if (fromZapTokenName === "WBNB") {
            // wbnb -> bnb
            txId = await withdrawWBNBToBNB(accountAddress, amount);
          } else {
            // token -> bnb
            txId = await zapOut(accountAddress, fromZapToken.address, amount);
          }
        } else {
          //   token1 -> token2
          //   token1 -> lp
          txId = await zapInToken(
            accountAddress,
            fromZapToken.address,
            amount,
            toZapToken.address
          );
        }
      }
    }
  }
  return txId;
};
```