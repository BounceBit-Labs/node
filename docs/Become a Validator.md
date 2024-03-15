# Become a Validator

## Set up a full node first

Follow the instructions [here to run a full node](https://github.com/BounceBit-Labs/node/blob/main/docs/Running%20a%20Full%20Node.md).

## Create a validator account

> The `keyring-backend` supports multiple backends, some of which may not be available on all operating systems.  More options, please refer to run `bbcored keys --help` for details.
>
> If `--home` is not explicitly specified, the default path resolves to ~/.bouncebit.



**Make sure to store and backup keyring file in a safe place.**

```
bbcored keys add operator --algo eth_secp256k1
```

## Add funds to your validator account

Check your validator account balance.

```
bbcored query bank balances $(bbcored keys show operator -a)
```

Feel free to contact us and provide your validator's address. We'll send `1 BB` token to your validator's address. Additionally, you can claim BB tokens via  Bouncebit Testnet network  on [BounceBit Discord Server](https://discord.com/invite/bouncebit).

## Submit staking to become a validator

Here is an example of creating a validator on Bouncebit Testnet.

**Check your validator pubkey**

```
bbcored tendermint show-validator
```

```
bbcored tx staking create-validator \
  --amount="100000000000000000000000bit" \
  --pubkey=$(bbcored tendermint show-validator) \
  --moniker="<YOUR_MONIKER>" \
  --commission-rate="0.05" \
  --commission-max-rate="0.10" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="100000000000000000000000" \
  --gas="10000000" \
  --gas-prices="8bit" \
  --from=operator \
  --home=$WORKSPACE
```

> When specifying commission parameters, the `commission-max-change-rate` is used to measure % *point* change over the `commission-rate`, e.g. 1% to 2% is a 100% rate increase, but only 1 percentage point.

> `Min-self-delegation` is a positive integer representing the minimum self-delegated voting power required for your validator at all times.

**Check Transaction Status**

Please review the transaction hash log associated with your validator creation to ensure there are no errors. If any errors are present, it indicates that the creation of your validator has failed.

```
bbcored query tx <your transaction hash>
```

**Check Validator Status**

Please check the status of your validator. If it is in the `BOND_STATUS_UNBONDED` state, please stake a sufficient amount of tokens to become a part of the active validator set.

```
bbcored query staking validator $(bbcored keys show operator --bech val -a)
```

## Delegate additional tokens

To delegate extra tokens to your validator by executing the following command.

```
bbcored tx staking delegate $(bbcored keys show operator --bech val -a) 100000000000bit --from operator
```

Once your validator is bonded, you can check if it is in the validator set by executing the following command and it should return your validator address.

```
bbcored query tendermint-validator-set | grep $(bbcored tendermint show-address)
```
