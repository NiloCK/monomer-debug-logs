This is similar to [reverted-deposit-tx](../reverted-deposit-tx/readme.md), but with a more specific / useful revert error message.

Desire: zero in on the cause of the revert.

This run used [this version](https://github.com/NiloCK/gethtx/tree/232aaf64cdd67c3058ebd836d2934281775ebd6e) of the gethtx util.

Specific modification is the 1e18/2 value for [`gasLimit`](https://github.com/ethereum-optimism/optimism/blob/14f76fb3c171bd133389b7c5a2c2792f51dccfea/packages/contracts-bedrock/src/L1/OptimismPortal.sol#L493) - used to specify how much L2 gas to buy.

Motivation was that previous gas limit may have been a problem. The updated amount is outrageously large.

```golang
depositTx, err := portal.DepositTransaction(
		&bind.TransactOpts{
			From: userAddress,
			Signer: func(addr common.Address, tx *types.Transaction) (*types.Transaction, error) {
				signed, err := types.SignTx(tx, signer, userPK)
				if err != nil {
					return nil, err
				}
				return signed, nil
			},
			GasPrice:  price,
			GasLimit:  1e6,
			GasFeeCap: nil,
			GasTipCap: nil,
			Nonce:     big.NewInt(int64(nonce)),
			Value:     big.NewInt(1e18), // 1 eth
			Context:   nil,
			NoSend:    false,
		},
		userAddress,
		big.NewInt(1e18/2), // 0.5 eth
		1e18/2,             // very large - should not trigger a revert - does! too big!
		false,
		[]byte{},
	)
```

The change did not prevent a revert, but it did trigger a more useful / relevant revert msg.

From [anvil logs](./anvil_latest.log):624

```log
[12:14:38.242]     Transaction: 0x32b6150a9f4f403e6084494f6dd3b8783a81d33ed338d738b06d9d227154a0fb
[12:14:38.242]     Gas used: 52001
[12:14:38.242]     Error: reverted with: revert: ResourceMetering: cannot buy more gas than available gas limit
```

This is a sort of progress. The revert makes sense (l2 has limit is much much less than this), and it's clear that the tx is hitting appropriate code & returning an [OP revert message](https://github.com/polymerdao/optimism-dev/blob/518341f3e2dc7bf88eb06513a740fc9ced1ccf39/packages/contracts-bedrock/src/L1/ResourceMetering.sol#L126), rather than something in Anvil's tx processing.
