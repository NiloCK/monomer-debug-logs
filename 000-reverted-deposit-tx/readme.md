In this run, the [external transactor utility](https://github.com/nilock/gethtx) succeeds in having anvil process a deposit transaction.

... but the transaction reverts. See anvil log line 604-606.

```log
[10:28:44.036]     Transaction: 0x1d6cac25c03ab9e0c009163d3e15424b0441dae1a73999d16ffeee138124eebf
[10:28:44.036]     Gas used: 984832
[10:28:44.036]     Error: reverted with: EvmError: Revert
```

This revert is a problem to be fixed.

Notable and surprising side effect is that monomer breaks down after this reverted deposit tx.

See monomer.log:120 and onward:

```log
    stack_test.go:68: WARN [05-30|10:28:44.038] node: Derivation process temporary error attempts=1 err="temp: failed to fetch receipts of L1 block 0xe6bdcd60bd7ffd696699d1571675688e3f71a52f3bd12c37fda36114609053a1:22 (parent: 0xc7d063da1855d11093989dbb2bf0efe673613f211db795e0ff3986d01e59b1b2:21) for L1 sysCfg update: querying block: invalid transaction v, r, s values"

    stack_test.go:68: DEBUG[05-30|10:28:44.038] node: scheduling re-attempt with delay   attempts=1 delay=2.14524489s

    stack_test.go:68: INFO [05-30|10:28:44.038] node: creating new block                 parent=bf3f17..7540b3:1 l1Origin=e6bdcd..9053a1:22

    stack_test.go:68: ERROR[05-30|10:28:44.039] node: sequencer temporarily failed to start building new block err="temp: failed to fetch L1 block info and receipts: querying block: invalid transaction v, r, s values"

    stack_test.go:68: INFO [05-30|10:28:44.039] node: creating new block                 parent=bf3f17..7540b3:1 l1Origin=e6bdcd..9053a1:22

    stack_test.go:68: ERROR[05-30|10:28:44.040] node: sequencer temporarily failed to start building new block err="temp: failed to fetch L1 block info and receipts: querying block: invalid transaction v, r, s values"

    ... (forever)
```

The invalid tx message is likely to be at least partly incorrect: I had previously been receiving it from anvil, and replaced it with a different error by sending a tx with a different nonce.

Some documentation here: https://github.com/foundry-rs/foundry/issues/7898

This is a bad failure mode for monomer. It should ignore the ineffectual deposit tx attempt and continue to operate.
