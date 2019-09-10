# sovrin-write-txns
Test of preparing transactions to write to sovrin network

## Using a local ledger and BCovrin Dev ledger

Scripts are in local-ledger sub-dir

1. Create local wallet with DID’s

```
indy-cli local-ledger/local-1-create-wallet.txt
```

2. Manually create any necessary DID’s on BCovrin Dev
   (use VON ledger browser)

3. Create local wallet with DID’s

```
indy-cli local-ledger/bcovrin-1-create-wallet.txt
```

   Manually edit schema to add endorser

```
vi /tmp/dev_schema_1.txt
```

4. Run script to sign schema txn

```
indy-cli local-ledger/local-2-sign-schema.txt
```

5. Run script to write schema to ledger

```
indy-cli local-ledger/bcovrin-2-write-schema.txt
```

6. Make a note of the schema version

7. TODO how to generate cred def public key

8. Run script to create cred def transaction

```
indy-cli local-ledger/local-3-create-cred-def.txt
```

9. Run script to write cred def to ledger

```
indy-cli local-ledger/bcovrin-3-write-cred-def.txt
```
