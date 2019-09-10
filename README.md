# sovrin-write-txns
Test of preparing transactions to write to sovrin network

## Using a local ledger (Issuer) and BCovrin Dev ledger (Endorser)

Local ledger is started as per sdk instructions https://github.com/hyperledger/indy-sdk#1-starting-the-test-pool-on-localhost

BCovrin Dev is http://dev.bcovrin.vonx.io/

1. Manually create any necessary DID’s on BCovrin Dev
   (use VON ledger browser)

   Note that DID's are created manually via the ledger browser, see http://dev.bcovrin.vonx.io/browse/domain?page=89

2. Create local wallet with DID’s (for issuer local ledger)

```
indy-cli local-ledger/issuer-1-create-wallet.txt
```

3. Create local wallet with DID’s (for endorser BCovrin Dev ledger)

```
indy-cli bcovrin-ledger/endorser-1-create-wallet.txt
```

4. Manually edit schema to add endorser `,"endorser":"AUPCKiiq1ema4fbkXYP2Kg"`

```
vi /tmp/dev_schema_1.txt
```

5. Run script to sign schema txn (issuer local ledger)

```
indy-cli issuer-2-sign-schema.txt
```

6. Run script to write schema to ledger (endorser BCovrin Dev ledger)

```
indy-cli endorser-2-write-schema.txt
```

7. Make a note of the schema version

8. TODO how to generate cred def public key

9. Run script to create cred def transaction (issuer local ledger)

```
indy-cli issuer-3-create-cred-def.txt
```

10. Run script to write cred def to ledger (endorser BCovrin Dev ledger)

```
indy-cli endorser-3-write-cred-def.txt
```

## Both Issuer and Endorser using BCovrin Dev ledger

BCovrin Dev is http://dev.bcovrin.vonx.io/

1. Manually create any necessary DID’s on BCovrin Dev
   (use VON ledger browser)

   Note that DID's are created manually via the ledger browser, see http://dev.bcovrin.vonx.io/browse/domain?page=89

2. Run script to create Issuer wallet (on BCovrin Dev ledger)

```
indy-cli bcovrin-ledger/issuer-1-create-wallet.txt
```

3. Etc ... just use the same scripts from the `local` section

   i.e. `local_pool` is now pointing to `bcovin_pool`


