This uses [this version](https://github.com/NiloCK/gethtx/tree/73a855916ad71c945dc1b2bcf8ba24fedd133b2f) of the transactor.

The change again is to the l2 `gasLimit` - now set to 25_000_000/100, one hundredth of the l2 gas limit (a reasonable number).

```
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
		25_000_000/100,     // from outputs on ruConfig - gasLimit: 25_000_000
		false,
		[]byte{},
	)
```

This tx _does not revert_, and the deposit-tx event _is emitted_.

```log
// transactor
[13:53:53.834] deposit tx sent: 0xa29d25e15f43bff72a976979b748e89c2c39c6850e809f7f26ca607e26fa68c9
```

```log
// anvil
[13:53:54.319]     Transaction: 0xa29d25e15f43bff72a976979b748e89c2c39c6850e809f7f26ca607e26fa68c9
[13:53:54.319]     Gas used: 278559
```

```log
// monomer
stack_test.go:196: deposit event: &{From:0x4408a2504736a48781D977db750D1E3fF49f86e1 To:0x4408a2504736a48781D977db750D1E3fF49f86e1 Version:+0 OpaqueData:[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 13 224 182 179 167 100 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 6 240 91 89 211 178 0 0 0 0 0 0 0 3 208 144 0] Raw:{Address:0xEE915F299A6d1eFf68c6EA22E5f93cFD551936F3 Topics:[0xb3813568d9991fc951961fcb4c784893574240a28925604d09fc577c55bb7c32 0x0000000000000000000000004408a2504736a48781d977db750d1e3ff49f86e1 0x0000000000000000000000004408a2504736a48781d977db750d1e3ff49f86e1 0x0000000000000000000000000000000000000000000000000000000000000000] Data:[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 32 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 73 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 13 224 182 179 167 100 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 6 240 91 89 211 178 0 0 0 0 0 0 0 3 208 144 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] BlockNumber:28 TxHash:0xa29d25e15f43bff72a976979b748e89c2c39c6850e809f7f26ca607e26fa68c9 TxIndex:0 BlockHash:0x6a3599fcadbbda8823d45a5e28244e03eb5f4aafe9524fd139df63ce3da71689 Index:0 Removed:false}}
    stack_test.go:67: ERROR[05-30|13:53:54.327] node: sequencer temporarily failed to start building new block err="temp: failed to fetch L1 block info and receipts: querying block: invalid transaction v, r, s values"

    stack_test.go:67: INFO [05-30|13:53:54.327] node: creating new block                 parent=03ac44..04e5d5:1 l1Origin=6a3599..a71689:28

    stack_test.go:67: ERROR[05-30|13:53:54.327] node: sequencer temporarily failed to start building new block err="temp: failed to fetch L1 block info and receipts: querying block: invalid transaction v, r, s values"

    stack_test.go:67: INFO [05-30|13:53:54.328] node: creating new block                 parent=03ac44..04e5d5:1 l1Origin=6a3599..a71689:28

    stack_test.go:67: ERROR[05-30|13:53:54.328] node: sequencer temporarily failed to start building new block err="temp: failed to fetch L1 block info and receipts: querying block: invalid transaction v, r, s values"
```

But the op-node / monomer fail to move forward and enter the same loop as with the revered tx.

Next:

- retry with branch of anvil that addresses this `v, r, s` error: https://github.com/foundry-rs/foundry/pull/7918
