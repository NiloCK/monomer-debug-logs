This uses [this version](https://github.com/NiloCK/gethtx/tree/73a855916ad71c945dc1b2bcf8ba24fedd133b2f) of the gethtx util.

The pertinent change is the

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
		25_000_000/2,       // from outputs on ruConfig - gasLimit: 25_000_000
		false,
		[]byte{},
	)
```

This "recovers" the non-informative revert message.

```log
[13:31:49.621]     Transaction: 0xb9218ffe4139676e83e77bba296327652f666ae54a3eef4ee58cec21587710a9
[13:31:49.621]     Gas used: 984832
[13:31:49.621]     Error: reverted with: EvmError: Revert
```

Known at this point:

- the tx is being run
- the tx is passing the `metered` modifier checks

Guesses:

- This value may still be too high and causing a revert.
- revert could be to do with sufficient gas on the L1 tx to cover this
  - but I don't fully understand the gas / gas burn mechanics here
