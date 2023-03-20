# Interacting with Sushiswap smart contracts through blockexplorer

# Removing liquidity through Trident router (example on Arbitrum):

- Before removing the liquidity through the Trident router contract the users have to approve Trident router address as a spender for their SLP tokens. As users usually have already done that when trying to remove liquidity through the UI (problems occur with the removing, not with the approval), just check user’s address to be sure the Trident router has been approved. If there is a problem with the approval itself, have to do the following:
1. Go to the pool address on the block explorer and select “Write contract” ([https://arbiscan.io/address/0x176542be47040929a34591367f243f83c1ee13de#writeContract](https://arbiscan.io/address/0x176542be47040929a34591367f243f83c1ee13de#writeContract))
2. Connect your wallet (Connect to Web3)
3. Click on “approve”
4. In the first filed enter the Trident router address - 0xD9988b4B5bBC53A794240496cfA9Bf5b1F8E0523
5. In the second put the amount of SLP owned by the user (or put even higher one) - 5246699295914733 (unit256)
6. Click “write”

- **Interacting with the Trident router**
1. Go to the Trident router address on the block explorer and select “Write contract” ([https://arbiscan.io/address/0xd9988b4b5bbc53a794240496cfa9bf5b1f8e0523#writeContract](https://arbiscan.io/address/0xd9988b4b5bbc53a794240496cfa9bf5b1f8e0523#writeContract))
2. Connect your wallet (Connect to Web3) 
3. Scroll down to 3.burnLiquidty
4. In the first filed put 0 (it is always 0)
5. In the next put the address of the pair - 0x176542be47040929a34591367f243f83c1ee13de 
6. Next put the amount of liquidity - 5246699295914733

1. In the data put - 0x000000000000000000000000d4351aa99ed4ee99b6b100c720f981ddc154bd980000000000000000000000000000000000000000000000000000000000000000 *
2. And in minWithdrawals (tuple[]) put - [["0x61de0041bd4c2951b3274028a86daaacc2260949","2306000000000000"],["0x82af49447d8a07e3bd95bd0d56f35241523fbab1","1191000000000000"]] ** (the addresses of both tokens in that pair and the minimal amounts (in unit256) the user is willing to accept) - min amounts should be at least 0.5% less then the actual amounts.
3. After that click "Write”

* Data consist of the address (usually the user’s address) and the destination (Bento balance or wallet) where the SLP should be unwinded. Both (address and destination) consist of 64 characters, so 0s where there is no value, and add 0x at the front. So the first 64 characters (after 0x) represent the users address (but without 0x). In that case user address is 0xd4351aa99ed4ee99b6b100c720f981ddc154bd98. So we take d4351aa99ed4ee99b6b100c720f981ddc154bd98 (40 characters) and add 24 zeros to the left. Destination is determined by the last digit in the data - 0 for unwinding in BentoBox, 1 for unwinding in wallet. So we take 0 or 1 and add 63 zeros to the left. Combine both (address and destination) and add 0x to the left.

** First address is the address of token0 and second one is the address of token1. Can check which of tokens is token0 and which one is token1 in the pool contract on the block explorer.

For some tokens with broken liquidity fee mechanics withdrawing to wallet will not work (if the token is paired with ETH), as it is not possible to unwind to ETH and have to receive weth, so should withdraw to user’s BentoBox balance and after that the user may withdraw the WETH to their wallet. (app.sushi.com→Portfolio→Account. The assets should be shown under “Bento” balance. Click on the asset and select “Withdraw to wallet)

![Screenshot 2023-03-13 at 15.39.40.png](https://github.com/CarpeCryptum/pics/blob/main/Screenshot%202023-03-18%20at%2019.42.32.png)

If transaction cost shown on the wallet is too high (like 1 eth) there is some problem. User should check if there are no spaces in front/after some of the values he entered and also check if the token addresses positions correspond to the sequence - token0 address should be the first one and token1 address should be the second one. If everything is alright with the inputs, but MM still gives high transaction cost or throw some other error should simulate the transaction on Tenderly and try to figure what the exact problem is.

On Tenderly the inputs are almost the same, but first have to specify the contract address (Trident router) and blockchain (Arbitrum)

![Screenshot 2023-03-13 at 16.17.10.png](https://github.com/CarpeCryptum/pics/blob/main/Screenshot%202023-03-13%20at%2016.17.10.png)


Click on “or Use Fetched Contract ABI” and select “burnLiquidity” function from the drop down menu:

![Screenshot 2023-03-13 at 16.17.44.png](https://github.com/CarpeCryptum/pics/blob/main/Screenshot%202023-03-13%20at%2016.17.44.png)

Then just put all the data  - same as on the contract example (without the first 0), just  minWithdrawals is a bit different and have to be like this: 

[{ "token":"0x61de0041bd4c2951b3274028a86daaacc2260949", "amount":"0" }, { "token":"0x82af49447d8a07e3bd95bd0d56f35241523fbab1", "amount":"0" }]*

*  Here the min amounts are set to 0 as this is the best way to check if the problem is with slippage or it is something else. If sim is successful with 100% slippage (0 min amounts), then start playing with the numbers till find the minimum slippage that works.

![Screenshot 2023-03-13 at 16.16.17.png](https://github.com/CarpeCryptum/pics/blob/main/Screenshot%202023-03-13%20at%2016.16.17.png)

Don’t forget to put user’s address in the “From” field (on the right)

**********************************REMOVING LIQUIDITY FROM LEGACY POOLS THROUGH V1 ROUTER CONTRACT**********************************

Just as with Trident pools, the users have to approve V1 router address as a spender for their SLP tokens - same procedure as described above.

- I**nteracting with V1 router** - on V1 router there are different functions for the different types of pools. RemoveLiquidity (for pools where none of the tokens is the chain native one), RemoveLiquidityEth (for pools where one of the tokens is the chain native one) and remove liquidtyEthSupportingFeeOnTransferTokens. In the example (on Arbitrum chain) will use removeLiquidity function, but the principle is the same and for the rest.
1. Go to the V1 router address on the block explorer and select “Write contract” ([https://arbiscan.io/address/0x1b02da8cb0d097eb8d57a175b88c7d8b47997506#writeContract](https://arbiscan.io/address/0x1b02da8cb0d097eb8d57a175b88c7d8b47997506#writeContract))
2. Connect your wallet (Connect to Web3) 
3. Scroll to 3.removeLiquidity
4. In “tokenA” put the token0 address - 0x7d28ef69bd63557ef61e96404f27cadd6c2832fd
5. In “tokenB” put token1 address - 0xfd086bc7cd5c481dcc9c85ebe478a1c0b69fcbb9
6. In “liquidity” put the amount of SLP owned by the user - 62245 (unit256)
7. In “amountAMin” put the minimum amount of token0 the user agrees to receive - 1960 (that’s 196 in unit256, as the token is with 1 decimal) *
8. In “amountBMin” put the minimum amount of token1 the user agrees to receive - 1960000 (that’s 1.96 in unit256, as the token - USDT - is with 6 decimals) *
9. In “to (address)” put the user address - 0xd1134f6e28ab3b75500378b47250d6d88e0c1bb0
10. In “deadline” put the end time (in unix timestamp) after which the transaction should be rejected if not executed within the set time period, can set timestamp like 5-10 mins in the future - 1678787791
11. Click “write”

![Screenshot 2023-03-14 at 11.30.12.png](https://github.com/CarpeCryptum/pics/blob/main/Screenshot%202023-03-14%20at%2011.30.12.png)
* In that particular case the min amounts are set with 4% slippage as the user is the only owner of liquidity in that pool. Defining the minimum possible slippage is possible by running simulations on Tenderly - the procedure is the same as when simulating removing liquidity through Trident router. Just have to select the V1 router address and in the V1 case the input is absolutely the same as on the block explorer

![Screenshot 2023-03-14 at 11.34.51.png](https://github.com/CarpeCryptum/pics/blob/main/Screenshot%202023-03-14%20at%2011.34.51.png)

************NOTE!************ Function “remove liquidtyEthSupportingFeeOnTransferTokens” can be used for saving assets accidentally sent directly to the v2 router contract. If user have sent token XYZ to the router contract, just add liquidity (or create a new pair if needed) to the XYZ/”native chain token” pair and after that remove the liquidity with the “remove liquidtyEthSupportingFeeOnTransferTokens” function. Usually bots are doing that too fast to be able to rescue, but if on some side chain or amount is not too big, there is a chance to save users asset.