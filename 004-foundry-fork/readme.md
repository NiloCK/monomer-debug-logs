Updated from 003.

Installing foundry tools via

`foundryup --repo ngotchac/foundry --branch ngotchac/fix-signature-v`. See https://github.com/foundry-rs/foundry/pull/7918/.

Prior signature errors are resolved, but op-node / monomer still failing to progress and falling into the same style of fast error loop.

```log
// from 003 - vrs signature errors processing deposit tx
stack_test.go:196: deposit event: &{From:0x4408a2504736a48781D977db750D1E3fF49f86e1 To:0x4408a2504736a48781D977db750D1E3fF49f86e1 Version:+0 OpaqueData:[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 13 224 182 179 167 100 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 6 240 91 89 211 178 0 0 0 0 0 0 0 3 208 144 0] Raw:{Address:0xEE915F299A6d1eFf68c6EA22E5f93cFD551936F3 Topics:[0xb3813568d9991fc951961fcb4c784893574240a28925604d09fc577c55bb7c32 0x0000000000000000000000004408a2504736a48781d977db750d1e3ff49f86e1 0x0000000000000000000000004408a2504736a48781d977db750d1e3ff49f86e1 0x0000000000000000000000000000000000000000000000000000000000000000] Data:[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 32 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 73 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 13 224 182 179 167 100 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 6 240 91 89 211 178 0 0 0 0 0 0 0 3 208 144 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] BlockNumber:28 TxHash:0xa29d25e15f43bff72a976979b748e89c2c39c6850e809f7f26ca607e26fa68c9 TxIndex:0 BlockHash:0x6a3599fcadbbda8823d45a5e28244e03eb5f4aafe9524fd139df63ce3da71689 Index:0 Removed:false}}
    stack_test.go:67: ERROR[05-30|13:53:54.327] node: sequencer temporarily failed to start building new block err="temp: failed to fetch L1 block info and receipts: querying block: invalid transaction v, r, s values"

    stack_test.go:67: INFO [05-30|13:53:54.327] node: creating new block                 parent=03ac44..04e5d5:1 l1Origin=6a3599..a71689:28

    stack_test.go:67: ERROR[05-30|13:53:54.327] node: sequencer temporarily failed to start building new block err="temp: failed to fetch L1 block info and receipts: querying block: invalid transaction v, r, s values"

    stack_test.go:67: INFO [05-30|13:53:54.328] node: creating new block                 parent=03ac44..04e5d5:1 l1Origin=6a3599..a71689:28

    stack_test.go:67: ERROR[05-30|13:53:54.328] node: sequencer temporarily failed to start building new block err="temp: failed to fetch L1 block info and receipts: querying block: invalid transaction v, r, s values"

    ...
```

```log
// from this run - a new error!
stack_test.go:196: deposit event: &{From:0xFa42202126BD6C02d90113195e6F6c00703c9077 To:0xFa42202126BD6C02d90113195e6F6c00703c9077 Version:+0 OpaqueData:[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 13 224 182 179 167 100 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 6 240 91 89 211 178 0 0 0 0 0 0 0 3 208 144 0] Raw:{Address:0xEE915F299A6d1eFf68c6EA22E5f93cFD551936F3 Topics:[0xb3813568d9991fc951961fcb4c784893574240a28925604d09fc577c55bb7c32 0x000000000000000000000000fa42202126bd6c02d90113195e6f6c00703c9077 0x000000000000000000000000fa42202126bd6c02d90113195e6f6c00703c9077 0x0000000000000000000000000000000000000000000000000000000000000000] Data:[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 32 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 73 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 13 224 182 179 167 100 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 6 240 91 89 211 178 0 0 0 0 0 0 0 3 208 144 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] BlockNumber:20 TxHash:0x8d3d04d34419a466561cdb96c36a692c53b7d29b7940e4b9340071e9c3e0dabd TxIndex:0 BlockHash:0x47b0ffeaef6e019e78197a32cff9eb3f87df71cbb551d155a9296d8b6690fd17 Index:0 Removed:false}}
    stack_test.go:67: WARN [05-31|11:42:31.150] node: Derivation process temporary error attempts=1 err="temp: failed to fetch receipts of L1 block 0x47b0ffeaef6e019e78197a32cff9eb3f87df71cbb551d155a9296d8b6690fd17:20 (parent: 0xcb6b46f29628b747857b79b431eb6b810de5bf200e9d40edf5eb5fabedb602ca:19) for L1 sysCfg update: invalid receipts: failed to fetch list of receipts: expected receipt root 0x46cdda11315362816acd220fa323fa3145cc336a36ec1b7fc664f618624aec80 but computed 0x34d529ef90878c6356396d2523a76db9f89f7f6d2efeff00838213010d07832f from retrieved receipts"

    stack_test.go:67: DEBUG[05-31|11:42:31.150] node: scheduling re-attempt with delay   attempts=1 delay=2.179433329s

    stack_test.go:67: INFO [05-31|11:42:31.151] node: creating new block                 parent=77cddb..2a42f3:1 l1Origin=47b0ff..90fd17:20

    stack_test.go:67: ERROR[05-31|11:42:31.151] node: sequencer temporarily failed to start building new block err="temp: failed to fetch L1 block info and receipts: invalid receipts: failed to fetch list of receipts: expected receipt root 0x46cdda11315362816acd220fa323fa3145cc336a36ec1b7fc664f618624aec80 but computed 0x34d529ef90878c6356396d2523a76db9f89f7f6d2efeff00838213010d07832f from retrieved receipts"

    stack_test.go:67: INFO [05-31|11:42:31.152] node: creating new block                 parent=77cddb..2a42f3:1 l1Origin=47b0ff..90fd17:20

    stack_test.go:67: ERROR[05-31|11:42:31.152] node: sequencer temporarily failed to start building new block err="temp: failed to fetch L1 block info and receipts: invalid receipts: failed to fetch list of receipts: expected receipt root 0x46cdda11315362816acd220fa323fa3145cc336a36ec1b7fc664f618624aec80 but computed 0x34d529ef90878c6356396d2523a76db9f89f7f6d2efeff00838213010d07832f from retrieved receipts"

    stack_test.go:67: INFO [05-31|11:42:31.153] node: creating new block                 parent=77cddb..2a42f3:1 l1Origin=47b0ff..90fd17:20

    stack_test.go:67: ERROR[05-31|11:42:31.153] node: sequencer temporarily failed to start building new block err="temp: failed to fetch L1 block info and receipts: invalid receipts: failed to fetch list of receipts: expected receipt root 0x46cdda11315362816acd220fa323fa3145cc336a36ec1b7fc664f618624aec80 but computed 0x34d529ef90878c6356396d2523a76db9f89f7f6d2efeff00838213010d07832f from retrieved receipts"

    stack_test.go:67: INFO [05-31|11:42:31.153] node: creating new block                 parent=77cddb..2a42f3:1 l1Origin=47b0ff..90fd17:20

    stack_test.go:67: ERROR[05-31|11:42:31.154] node: sequencer temporarily failed to start building new block err="temp: failed to fetch L1 block info and receipts: invalid receipts: failed to fetch list of receipts: expected receipt root 0x46cdda11315362816acd220fa323fa3145cc336a36ec1b7fc664f618624aec80 but computed 0x34d529ef90878c6356396d2523a76db9f89f7f6d2efeff00838213010d07832f from retrieved receipts"
```
