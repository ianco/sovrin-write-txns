# Deprecated

This whole process has been refined and integrated into [von-network](https://github.com/bcgov/von-network) via [von-network PR76](https://github.com/bcgov/von-network/pull/76).


# sovrin-write-txns
Test of preparing transactions to write to sovrin network


## Starting von-agent-template on the Sovrin Staging network with un-privileged DID's

You can see this process in action here:  https://zoom.us/recording/play/D8WVfMhYakL19PYXoxTBQgBfYMH8bQO372fC75zTJplAycBSj7sTE-fmQy--RFU4?continueMode=true

Indy-cli scripts are all in https://github.com/ianco/sovrin-write-txns

Note that all indy-cli scripts leave you logged into the CLI when they are done (in case you want to do some more queries).  To get back to a command prompt enter `exit`.

Note also you have to manually enter wallet passwords at every step (create, open, export and import).


1. Run OrgBook with Sovrin connection parameters (note no local von-network requried):

```
GENESIS_URL=https://raw.githubusercontent.com/sovrin-foundation/sovrin/master/sovrin/pool_transactions_sandbox_genesis AUTO_REGISTER_DID=false LEDGER_URL=https://foo.com ./manage start seed=0000000000000o_anon_research_inc
```


2. Run von-agent-template (assume you have already done the `. init.sh` and `./manage build` to initialize):

```
./manage start
```

Note that this requires a DID on the Sovrin network

Note that this will fail because the DID doesn't have privileges to write to the Sovrin network


3. Export the agent's wallet - it will contain a VON-compatible DID with metadata

```
indy-cli ~/Projects/sovrin-write-txns/sovrin-staging-ledger/issuer-0-export-wallet.txt
```

Note there will be nothing else in the wallet

Note leave the OrgBook and VON Agent services running, we will be using the wallet database


4. Run a set of local Indy nodes (this is in the Indy SDK):

```
docker run -itd -p 9701-9708:9701-9708 indy_pool
```


5. Initialize a temporary (local) wallet for building the new cred def

```
indy-cli ~/Projects/sovrin-write-txns/sovrin-staging-ledger/issuer-1-create-wallet.txt
```


6. Initialize an endorser wallet

```
indy-cli ~/Projects/sovrin-write-txns/sovrin-staging-ledger/endorser-1-create-wallet.txt
```


7. Edit the saved transaction in `/tmp/dev_schema_1.txt` to add `,"endorser":"DFuDqCYpeDNXLuc3MKooX3"`


8. Sign the transaction with your issuer:

```
indy-cli ~/Projects/sovrin-write-txns/sovrin-staging-ledger/issuer-2-sign-schema.txt
```


9. Sign the transaction with your endorser and write to the ledger:

```
indy-cli ~/Projects/sovrin-write-txns/sovrin-staging-ledger/endorser-2-write-schema.txt
```

Make a note of the `Sequence Number`, you will need it later

```
Following Schema has been received.
Metadata:
+------------------------+-----------------+---------------------+---------------------+
| Identifier             | Sequence Number | Request ID          | Transaction time    |
+------------------------+-----------------+---------------------+---------------------+
| DFuDqCYpeDNXLuc3MKooX3 | 70371           | 1568249112025424000 | 2019-09-12 00:45:09 |
+------------------------+-----------------+---------------------+---------------------+
Data:
+-------------------+---------+---------------------------------------------------------------------------------------------------------+
| Name              | Version | Attributes                                                                                              |
+-------------------+---------+---------------------------------------------------------------------------------------------------------+
| ian-permit.ian-co | 1.0.7   | "permit_id","permit_type","permit_issued_date","permit_status","effective_date","legal_name","corp_num" |
+-------------------+---------+---------------------------------------------------------------------------------------------------------+
```


10. Edit scripts to include the above Sequence Number = `cred_def.py` and `issuer-3-create-cred-def.txt`


11. Create the cred def in our local wallet:

```
python3 cred_def.py
```

Make a note of the `primary` produced by this script, copy and paste it into an editor, and edit to remove all spaces, line feeds etc


12. Edit `issuer-3-create-cred-def.txt` to include the primary from the above step


13. Now run it - signs the transaction and sends to the endorser

```
indy-cli ~/Projects/sovrin-write-txns/sovrin-staging-ledger/issuer-3-create-cred-def.txt
```


13a. As above, edit the saved transaction in `/tmp/dev_cred_def_1.txt` to add `,"endorser":"DFuDqCYpeDNXLuc3MKooX3"`


13b. Run the following to sign the transaction:

```
indy-cli ~/Projects/sovrin-write-txns/sovrin-staging-ledger/issuer-3-sign-cred-def.txt
```


14. Endorser signs and writes to the ledger

```
indy-cli ~/Projects/sovrin-write-txns/sovrin-staging-ledger/endorser-3-write-cred-def.txt
```


15. Export our "temporary wallet"

```
indy-cli ~/Projects/sovrin-write-txns/sovrin-staging-ledger/issuer-4-export-wallet.txt
```


16. Kill all the OrgBook and VON Agent processes (`./manage rm`),a nd also kill the local ledger (`docker ps -a -q | xargs docker rm -f`)


17. Startup OrgBook again:

```
GENESIS_URL=https://raw.githubusercontent.com/sovrin-foundation/sovrin/master/sovrin/pool_transactions_sandbox_genesis AUTO_REGISTER_DID=false LEDGER_URL=https://foo.com ./manage start seed=0000000000000o_anon_research_inc
```


18. Startup *only* the Agent Wallet Database (so we can import our temporary wallet):

```
./manage start myorg-wallet-db
```


19. Now run the script to import our database:

```
indy-cli ~/Projects/sovrin-write-txns/sovrin-staging-ledger/issuer-5-import-wallet.txt
```


20. Last step!  Stop the wallet database and start all Agent processes:

```
./manage stop
./manage start
```

(make sure you ./manage `stop` and *NOT* `rm`)

God willing, you will see output something like this:

```
myorg-agent_1      | 2019-09-12 01:03:24,236 INFO [von_anchor.wallet]: Created wallet bctob-Verifier-Wallet
myorg-agent_1      | 2019-09-12 01:03:24,633 INFO [von_anchor.wallet]: Opened wallet bctob-Verifier-Wallet on handle 6
myorg-agent_1      | 2019-09-12 01:03:24,641 INFO [von_anchor.wallet]: Wallet bctob-Verifier-Wallet set seed hash metadata for DID VePGZfzvcgmT3GTdYgpDiT
myorg-agent_1      | 2019-09-12 01:03:25,052 INFO [von_anchor.wallet]: Opened wallet myorg_issuer on handle 7
myorg-agent_1      | 2019-09-12 01:03:25,059 INFO [von_anchor.wallet]: Wallet myorg_issuer got verkey GcWd3dYwmptFdGBf4DFMi5jUxmcgBNVoDWRV2ritLh4r for existing DID VePGZfzvcgmT3GTdYgpDiT
myorg-agent_1      | 2019-09-12 01:03:26,168 INFO [von_anchor.anchor.base]: myorg_issuer endpoint already set as http://192.168.65.3:5001
myorg-agent_1      | 2019-09-12 01:03:26,168 INFO [vonx.indy.service]: messages.Endpoint stored: http://192.168.65.3:5001
myorg-agent_1      | 2019-09-12 01:03:26,168 INFO [vonx.indy.service]: Checking for schema: ian-permit.ian-co (1.0.7)
myorg-agent_1      | 2019-09-12 01:03:26,333 INFO [von_anchor.anchor.base]: BaseAnchor.get_schema: got schema SchemaKey(origin_did='VePGZfzvcgmT3GTdYgpDiT', name='ian-permit.ian-co', version='1.0.7') from ledger
myorg-agent_1      | 2019-09-12 01:03:26,335 INFO [vonx.indy.service]: Checking for credential def: ian-permit.ian-co (1.0.7)
myorg-agent_1      | 2019-09-12 01:03:26,654 INFO [von_anchor.anchor.base]: BaseAnchor.get_cred_def: got cred def VePGZfzvcgmT3GTdYgpDiT:3:CL:70371:tag from ledger
myorg-agent_1      | 2019-09-12 01:03:26,654 INFO [vonx.indy.service]: Indy agent synced: ian-co
myorg-agent_1      | 2019-09-12 01:03:27,049 INFO [von_anchor.wallet]: Opened wallet bctob-Verifier-Wallet on handle 13
myorg-agent_1      | 2019-09-12 01:03:27,051 INFO [von_anchor.wallet]: Wallet bctob-Verifier-Wallet got verkey GcWd3dYwmptFdGBf4DFMi5jUxmcgBNVoDWRV2ritLh4r for existing DID VePGZfzvcgmT3GTdYgpDiT
myorg-agent_1      | 2019-09-12 01:03:27,217 INFO [von_anchor.anchor.base]: BaseAnchor.get_endpoint: got endpoint for VePGZfzvcgmT3GTdYgpDiT from cache
myorg-agent_1      | 2019-09-12 01:03:28,367 INFO [vonx.indy.service]: messages.Endpoint stored: None
myorg-agent_1      | 2019-09-12 01:03:28,367 INFO [vonx.indy.service]: Indy agent synced: bctob
myorg-agent_1      | 2019-09-12 01:03:28,368 WARNING [vonx.indy.tob]: No file found at logo path: ../config/../assets/img/ian-co-logo.jpg
myorg-agent_1      | 2019-09-12 01:03:30,154 INFO [vonx.common.service]: Starting sync: indy
```


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


## Both Issuer and Endorser using local Indy ledger

As above, but use the `create-wallet` scripts in the `local-ledger` folder


## Cleanup local environment

In between runs of the above:

```
rm /tmp/dev*
rm -rf ~/.indy*
```

... and also restart local Indy ledger

