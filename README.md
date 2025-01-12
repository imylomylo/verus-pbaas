# PBaaS Chain Generation on Verus

This repository is intended to document and inform about the steps required to create a PBaaS chain on Verus (testnet, v0.9.2-3), with the goal of providing a reproducible workflow for future use.

**Contents:**

- [Compiling from source](#compile)
- [Generating an ID for use on Verus Testnet](#idgen)
- [Generating a PBaaS chain](#chaingen)
  - [Conceptual description of chosen `definecurrency` options](#chaingen-concept)
    - [Background on CHIPS](#background)
    - [Itemized options for chipstensec chain](#options-chipstensec)
    - [Itemized options for gateway converter](#options-gateway)
  - [Funding identity](#chaingen-fund)
  - [Defining a currency](#chaingen-define)
  - [Verifying currency generation](#chaingen-verify)
- [Launching a PBaaS Chain](#launch)

---

<h2 id="compile">Compiling from Source</h2>

<h3>Step 1: Clone, build dependencies:</h3>

```
cd ~
git clone https://github.com/VerusCoin/VerusCoin.git verus
cd ~/verus/depends
make -j4
```

Wait for `depends` to finish building.

<h3>Step 2: Configure build enviroment, and build from source:</h3>

```
cd ~/verus
./autogen.sh
CONFIG_SITE=$PWD/depends/x86_64-unknown-linux-gnu/share/config.site ./configure
make -j4
```

If all steps completed successfully, you should have resulting binaries located in `~/verus/src`.

<h3>Step 3: Launch Verus daemon</h3>

```
# Create a new tmux session
tmux new -s vrsctest

# Launch vrsctest chain
cd ~/verus/src
./verusd -chain=vrsctest &

# Detaching from tmux session can be done with Ctrl+B, then D

```

If you wish to mine/stake, you can launch with following options:

```
./verusd -chain=vrsctest -gen -genproclimit=2 -mineraddress=RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph -mint &
```

Fill in equivalent values for your wallet, desired mining process limits, etc.  When you need to re-attach to your `tmux` session, you can do so with: `tmux a -t vrsctest`.

---

<h2 id="idgen">Generate an ID for use on Verus Testnet</h2>

*You must have at least 100 VRSCTEST in your wallet to generate a primary ID on the Verus testnet*

<h3>Step 1: Register name commitment</h3> 

```
./verus -chain=vrsctest registernamecommitment chipstensec RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph
```

You may optionally add other fields such as a referral id.  Help text should be consulted for optional parameters. They will not be covered here.

If successful, output should show similar to the following:

```
{
  "txid": "a5e1cc0dd3ec5774c35087adb2bb7aff7bd74836c6a4c0b75c280aa6419d04a1",
  "namereservation": {
    "version": 1,
    "name": "chipstensec",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "6b2ff5b181e2788b5108592742d9214b0e9a33569e5e1ebad48e4000e9aad986",
    "referral": "",
    "nameid": "iKRtCZT87icurxb1ECUHKBq4jcuCTUV4ty"
  }
}
```

<h3>Step 2: Register identity</h3>

Copy and paste the entire resulting JSON object into a `registeridentity` command, issued to `verus` client, and append desired identity specifications:

```
./verus -chain=vrsctest registeridentity '{
  "txid": "a5e1cc0dd3ec5774c35087adb2bb7aff7bd74836c6a4c0b75c280aa6419d04a1",
  "namereservation": {
    "version": 1,
    "name": "chipstensec",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "salt": "6b2ff5b181e2788b5108592742d9214b0e9a33569e5e1ebad48e4000e9aad986",
    "referral": "",
    "nameid": "iKRtCZT87icurxb1ECUHKBq4jcuCTUV4ty"
  },
    "identity":{
        "name":"chipstensec", 
        "primaryaddresses":["RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph"], 
        "minimumsignatures":1, 
        "privateaddress": ""
    }
}'
```
You should modify relevant identity fields to match desired id, control address, minsigs, and optionally a private address.

If successful, output should display a txid:

```
b4e78320eb5ecc9d2ff2f3efd2f6fcea9182393658dc589d023825d0ae83b680
```

Wait for this transaction to be confirmed.

<h3>Step 3: Verify identity has been created</h3>

Identities can be located by name, or by i-address:

```
# By name
./verus -chain=vrsctest getidentity "chipstensec@"

# By i-address
./verus -chain=vrsctest getidentity iHjThi6GKjyp6QC7wjewHN7LaQ1DQxXe1i
```

If successful, output should show similar to the following:

```
{
  "identity": {
    "version": 3,
    "flags": 0,
    "primaryaddresses": [
      "RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph"
    ],
    "minimumsignatures": 1,
    "name": "chipstensec",
    "identityaddress": "iKRtCZT87icurxb1ECUHKBq4jcuCTUV4ty",
    "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "systemid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
    "contentmap": {
    },
    "revocationauthority": "iKRtCZT87icurxb1ECUHKBq4jcuCTUV4ty",
    "recoveryauthority": "iKRtCZT87icurxb1ECUHKBq4jcuCTUV4ty",
    "timelock": 0
  },
  "status": "active",
  "canspendfor": true,
  "cansignfor": true,
  "blockheight": 73188,
  "txid": "b4e78320eb5ecc9d2ff2f3efd2f6fcea9182393658dc589d023825d0ae83b680",
  "vout": 0
}
```

**Before generating PBaaS chain, repeat Steps 2 & 3 to generate verus ids for all desired notaries in `definecurrency` command.** 

---

<h2 id="chaingen">Generating a PBaaS Chain</h2>

<h3 id="chaingen-concept">Conceptual description of chosen `definecurrency` options</h3>

In [step 2, we will issue a `definecurrency` command to vrsctest daemon](#chaingen-define), which includes selected options tailored to CHIPS unique case and blockchain history.  It is important to understand both why these options were chosen and what they do.

<h4 id="background">Background on CHIPS blockchain</h4>

CHIPS is a coin that has already reached maximum total supply.  In its present form, mining is largely altruistic since block rewards have reduced to zero, and transactions are quite infrequent (miners not being paid by fees).  Price discovery has not occurred, and we want to ensure it takes place using Verus's PBaaS tooling.  We want existing holders of CHIPS to retain their coins while migrating to a new PBaaS chain, and to provide some backing in basket currencies for price discovery via a gateway converter.

Verus's fee market presents a unique opportunity for a coin that has already reached its max supply and will emit no new coins.  Actions taken by users via id generation, cross-chain conversions, and more, will provide greater block rewards to miners.  Combined with merge mining capabilities on Verus, this is a big improvement in theoretical security model for CHIPS.

The options below will generate a `chipstensec` blockchain, as well as a gateway converter `bridge.chipstensec`.

<h4 id="options-chipstensec">`definecurrency` options for `chipstensec` and justifications</h4>

- `"name":"CHIPSTENSEC"`

Name for our generated chain

- `"options": 264`

Defines our chain as a PBaaS chain, with `OPTION_PBAAS: 0x100 = 256`, and `OPTION_ID_REFERRALS: 8`

- `"currencies":["vrsctest"]`

Defines our chain as being based on existing `vrsctest` currency

- `"maxpreconversion":[0]`

Prevents users of `vrsctest` blockchain from preconverting `vrsctest` to `chipstensec` in during pre-launch period.  If we wanted to accept pre-launch conversions, we could set `minpreconversion` and `maxpreconversion` to specify required preconversion amounts for launch to proceed.

- `"conversions":[1]`

Specifies the desired conversion ratio for `vrsctest` to `chipstest`

- `"eras"`

Describes reward schedule on new `chipstensec` chain.  We want block subsidies to be zero, so that all emission comes from gateway converter.  Any additional issuance beyond that will be supply-neutral.

- `"notaries":["biz@","BizNotary@","BizNotary2@"]`

These notaries will be responsible for notarizing from `chipstensec` to `vrsctest` and vice versa.  These are required for cross-chain interaction.  Specified notaries here are all owned by Biz's control address.

- `"nodes":[{...},{...}]`

Seed nodes for new PBaaS chain

- `"preallocations":[{"biz@":20850000},{"BizNotary@":25000},{"BizNotary2@":25000}]`

Pre-allocated amount from supply of new chain.  In this example, we are pre-allocating 20,900,000 `chipstensec` coins to identities `biz@`, `BizNotary@`, and `BizNotary2@`.  These can then be re-allocated based on snapshot from legacy CHIPS codebase.

<h4 id="options-gateway">`definecurrency` options for gateway converter</h4>

- `"gatewayconvertername":"bridge"`

Defines a separate blockchain for gateway conversions named `bridge.chipstensec`

- `"gatewayconverterissuance":100000`

Specifies that our final 100,000 `chipstensec` supply will be issued through the gateway converter, for a max supply of 21 million coins.

- `"currencies":["vrsctest","kmd","chipstensec"]`

Specifies a basket of currencies backing our gateway issuance.

- `"initialcontributions":[1000,90,0]`

Specifies amounts of backing currencies to contribute on launch of gateway from our `chipstensec@` identity address.

- `"initialsupply":3000`

1:1 backing against `vrsctest` amount of 1000 in a basket of 3 currencies


<h3 id="chaingen-fund">Step 1: Send VRSCTEST, and Basket currencies to identity address</h3>

To generate a PBaaS chain, you will need *at least* a 10,200 VRSCTEST balance in the address registered to your newly generated identity.  Once again, we can send to either the identity's canonical name, or the i-address associated with it.

10,200 VRSCTEST is a base cost (covering 200 VRSCTEST `currencyregistrationfee`, and 10k VRSCTEST `pbaassystemregistrationfee`).  Additional registration costs may be necessary, depending on currency definition specifics.

For this chain, we will be generating both a PBaaS chain (`chipstensec@`), and a gatewayconverter chain (`bridge.chipstensec`).  We will be initially contributing a basket of currencies to the gateway converter.  In example below, we are sending `15000 VRSCTEST` and `100 KMD` to fund the basket.

```
# send vrsctest
./verus -chain=vrsctest sendtoaddress chipstensec@ 15000
374c9021a3f026f62dfadba4516ab23d7377749517528bf5459c8571d7b75322

# send kmd
./verus -chain=VRSCTEST sendcurrency RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph '[{"address":"chipstensec@","currency":"kmd","amount":120}]'
opid-90c1e936-830f-416b-8532-dd75fc14ad85
```

We also want to verify that the resulting operation id for 'kmd` transfer succeeded:

```
./verus -chain=vrsctest z_getoperationresult '["opid-90c1e936-830f-416b-8532-dd75fc14ad85"]'
[
  {
    "id": "opid-e96fa394-7a8d-4782-a05b-af9ae3852fce",
    "status": "success",
    "creation_time": 1659457274,
    "result": {
      "txid": "34f81f6ad827f42788f1683a57895d8e071a6c04758e40870f005e6efa418f11"
    },
    "execution_secs": 0.305115012,
    "method": "sendcurrency",
    "params": [
      {
        "address": "chipstensec@",
        "currency": "kmd",
        "amount": 100
      }
    ]
  }
]
```

<h3 id="chaingen-define">Step 2: Define a currency with desired parameters</h3>

```
./verus -chain=vrsctest definecurrency '{"name":"chipstensec","options":264,"currencies":["vrsctest"],"maxpreconversion":[0], "conversions":[1],"eras":[{"reward":0,"decay":0,"halving":0,"eraend":0}],"notaries":["biz@","BizNotary@","BizNotary2@"],"minnotariesconfirm":2,"nodes":[{"networkaddress":"51.222.159.244:12121"},{"networkaddress":"149.56.13.160:12121"}],"preallocations":[{"biz@":20850000},{"BizNotary@":25000},{"BizNotary2@":25000}], "gatewayconvertername":"bridge", "gatewayconverterissuance":100000}' '{"currencies":["vrsctest","kmd","chipstensec"],"initialcontributions":[1000,90,0],"initialsupply":3000}'
```

If successful, you should see a very large JSON output in your terminal.  This command does not finish the process of defining a currency.  It simply constructs a transaction, and does not send it to network.


<h3>Step 3: Send resulting raw transaction to network</h3>

Copy and paste the `"hex"` value from JSON output into `sendrawtransaction`

```
./verus -chain=vrsctest sendrawtransaction 0400008085202f890380b683aed02538029d58dc5836398291eafcf6d2eff3f22f9dcc5eeb2083e7b400000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc9940ddc170bdc42ea32f1b0864bf8cf38a06a998ebd4f052a894aed1f72cd9f7a8b03c9cb2715a4973b10b4f70d91d64188bcbf572876a9298b6a382d0f344fa5c1dffffffff118f41fa6e5e000f87408e75046c1a078e5d89573a68f18827f427d86a1ff83400000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc99400b1f019dfbd6275bec933070831064dd5caf38666e901b761abc9bc8a86565424eb8b012b88a02839c35a18f01d18b27a539b000ab1b1b2821b594d931977961ffffffff2253b7d771859c45f58b5217957477733db26a51a4dbfa2df626f0a321904c3701000000694c67010101012102d6cffd65b1e6209af2266742f5e048614658e48d023c1e2c8e7e5ec4454bfc9940a0e10dab3f62c6fdc6d5ef40ab5e0eee3628f75b4f71449f9c8f443d5df12d690ae167a7f39cf83df26e57df0bdef84f44554f1da62493f14d6de7b65f5ac419ffffffff0e0000000000000000fd26014704030001031504af02199c2345ac75f3eb20cb6d71edf2dce533771504af02199c2345ac75f3eb20cb6d71edf2dce533771504af02199c2345ac75f3eb20cb6d71edf2dce53377cc4cda04030e01011504af02199c2345ac75f3eb20cb6d71edf2dce533774c8503000000010000000114fdb1570a0d7ff7ec497ae03fef470b30ab75b1e501000000a6ef9ea235635e328124ff3429db9f9e91b64e2d0b636869707374656e7365630000af02199c2345ac75f3eb20cb6d71edf2dce53377af02199c2345ac75f3eb20cb6d71edf2dce5337700a6ef9ea235635e328124ff3429db9f9e91b64e2d000000001b04030f01011504af02199c2345ac75f3eb20cb6d71edf2dce533771b04031001011504af02199c2345ac75f3eb20cb6d71edf2dce53377750000000000000000fdf3012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4dc60104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54d9c010100000008010000a6ef9ea235635e328124ff3429db9f9e91b64e2d0b636869707374656e736563a6ef9ea235635e328124ff3429db9f9e91b64e2daf02199c2345ac75f3eb20cb6d71edf2dce5337701000000010000000000000000000000000000000000000000000000000083bb15000000000000000000038e6e5ae05ebf7dd1d74b17f45ca4791e68bf639e00505be44b6807006df4164224d7eb36df23a3fccf2cd6ae7b5827dc00a89c134602000012fe13ad935c9be94c10ff064b30d126b3baa15300a89c134602000000a0724e1809000001a6ef9ea235635e328124ff3429db9f9e91b64e2d000100e1f50500000000000100000000000000000100000000000000000100000000000000000000000000038e6e5ae05ebf7dd1d74b17f45ca4791e68bf639e6df4164224d7eb36df23a3fccf2cd6ae7b5827dc12fe13ad935c9be94c10ff064b30d126b3baa15302a49faec70003aed6c100c9bfde8f009c8ca4939f00a49faec700bc8340bc8340066272696467650fff001e01000000000000000001000000000000000001000000000100000000750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a01000100af02199c2345ac75f3eb20cb6d71edf2dce5337701000000af02199c2345ac75f3eb20cb6d71edf2dce533770000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004000000750000000000000000fdb9012704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4d8c0104030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34d62010180030000af02199c2345ac75f3eb20cb6d71edf2dce5337701000200af02199c2345ac75f3eb20cb6d71edf2dce5337701a6ef9ea235635e328124ff3429db9f9e91b64e2d01000000000100e1f50500000000000082dcbd84cf9bff0000000000000000000000000000000000000000000000000000000000000000000100000000000000000100000000000000000100000000000000000100e1f505000000000100000000000000000100000000000000000100000000010000000000000000011e01000000000000000000000000000000000000000000000000000000000000000000ffffffff0000000000000000000000000000000000000000000000000000000000000000000000000000021435312e3232322e3135392e3234343a31323132310000000000000000000000000000000000000000133134392e35362e31332e3136303a31323132310000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d0000000000000000000000000000000000000000000000000000000000000000af02199c2345ac75f3eb20cb6d71edf2dce53377af02199c2345ac75f3eb20cb6d71edf2dce533770083bb010000000001a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a7400000001a6ef9ea235635e328124ff3429db9f9e91b64e2d0088526a74000000000000ffffffff00750000000000000000fd7d012704030001012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef5cc4d500104030201012102a0de91740d3d5a3a4a7990ae22315133d02f33716b339ebce88662d012224ef54d26010100000021020000af02199c2345ac75f3eb20cb6d71edf2dce5337706627269646765a6ef9ea235635e328124ff3429db9f9e91b64e2daf02199c2345ac75f3eb20cb6d71edf2dce5337701000000010000000000af02199c2345ac75f3eb20cb6d71edf2dce5337783bb150000b864d9450000000000a0724e1809000003a6ef9ea235635e328124ff3429db9f9e91b64e2d3c28dd25a7127ce1b1e9a8dd9fa550a1b805dc10af02199c2345ac75f3eb20cb6d71edf2dce533770356a0fc0155a0fc0155a0fc010300000000000000000000000000000000000000000000000000000300e8764817000000001a71180200000000000000000000000300e8764817000000001a711802000000000000000000000000000000000000a49faec70003aed6c100750000000000000000b51a0403000101146e4ae35cca122eb65e73abd4c956940ef25a3eabcc4c9604030d0101146e4ae35cca122eb65e73abd4c956940ef25a3eab4c7a01000100af02199c2345ac75f3eb20cb6d71edf2dce5337701000000c26b4433c2d037372b98022ddb3ecf69318b308d00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000ffffffff750000000000000000fd23022704030001012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c3cc4df60104030501012102d85f078815b7a52faa92639c3691d2a640e26c4e06de54dd1490f0e93bcc11c34dcc010180030000c26b4433c2d037372b98022ddb3ecf69318b308d01000300c26b4433c2d037372b98022ddb3ecf69318b308d03a6ef9ea235635e328124ff3429db9f9e91b64e2d3c28dd25a7127ce1b1e9a8dd9fa550a1b805dc10af02199c2345ac75f3eb20cb6d71edf2dce533770356a0fc0155a0fc0155a0fc01030000000000000000000000000000000000a0724e1809000087dcca91ef000087dcca91ef000000000000000000000000000000000000000000000000000000000000000000030000000000000000000000000000000000a0724e180900000300000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000039f86010000000000a086010000000000a08601000000000003000000000000000000000000000000000000000000000000030000000000000000000000000000000000000000000000000300000000000000000000000003000000000000000000000000000000000000000000000000011e01000000000000000000000000000000000000000000000000000000000000000000ffffffff000000000000000000000000000000000000000000000000000000000000000000000000000000750000000000000000fdff002704030001012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4acc4cd304030c01012102cbfe54fb371cfc89d35b46cafcad6ac3b7dc9b40546b0f30b2b29a4865ed3b4a4caa01004100a6ef9ea235635e328124ff3429db9f9e91b64e2d0000000000000000000000000000000000000000000000000000000000000000af02199c2345ac75f3eb20cb6d71edf2dce53377c26b4433c2d037372b98022ddb3ecf69318b308d0083bb010000000001a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b540200000001a6ef9ea235635e328124ff3429db9f9e91b64e2d00e40b5402000000000000ffffffff0075cbc6f44917000000981a040300010114cb8a0f7f651b484a81e2312c3438deb601e27368cc4c79040308010114cb8a0f7f651b484a81e2312c3438deb601e273684c5d01a6ef9ea235635e328124ff3429db9f9e91b64e2d81f3ced0f02b05a6ef9ea235635e328124ff3429db9f9e91b64e2d809b200414c26b4433c2d037372b98022ddb3ecf69318b308dc26b4433c2d037372b98022ddb3ecf69318b308d75204e000000000000971a040300010114cb8a0f7f651b484a81e2312c3438deb601e27368cc4c78040308010114cb8a0f7f651b484a81e2312c3438deb601e273684c5c013c28dd25a7127ce1b1e9a8dd9fa550a1b805dc10a0c3cce14205a6ef9ea235635e328124ff3429db9f9e91b64e2d809b200414c26b4433c2d037372b98022ddb3ecf69318b308dc26b4433c2d037372b98022ddb3ecf69318b308d750088526a74000000832704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5704030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502f01a6ef9ea235635e328124ff3429db9f9e91b64e2d8dc5d1c98f00af02199c2345ac75f3eb20cb6d71edf2dce533777500e40b5402000000822704030001012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c050cc4c5604030b01012103b99d7cb946c5b1f8a54cde49b8d7e0a2a15a22639feb798009f82b519526c0502e01a6ef9ea235635e328124ff3429db9f9e91b64e2da49faec700af02199c2345ac75f3eb20cb6d71edf2dce533777515ab457858000000551b04030001011504af02199c2345ac75f3eb20cb6d71edf2dce53377cc3604030901011504af02199c2345ac75f3eb20cb6d71edf2dce533771a013c28dd25a7127ce1b1e9a8dd9fa550a1b805dc1082dae0e43e7500000000161e01000000000000000000000000

# resulting txid
567cc5eb56134553ac1eb7da69e74b67fa503d390f128ef0fd569ddb8dd5e897
```

<h3 id="chaingen-verify">Step 4: Verify currency generation was succesful</h3>

```
./verus -chain=vrsctest getcurrency chipstensec
```

If successful, output should show similar to the following:

```
{
  "version": 1,
  "options": 264,
  "name": "chipstensec",
  "currencyid": "iKRtCZT87icurxb1ECUHKBq4jcuCTUV4ty",
  "parent": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
  "systemid": "iKRtCZT87icurxb1ECUHKBq4jcuCTUV4ty",
  "notarizationprotocol": 1,
  "proofprotocol": 1,
  "launchsystemid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
  "startblock": 73237,
  "endblock": 0,
  "currencies": [
    "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq"
  ],
  "conversions": [
    1.00000000
  ],
  "maxpreconversion": [
    0.00000000
  ],
  "preallocations": [
    {
      "iGTdav42siHgX96ybWn3pRTFxQgiUpZ9K9": 20850000.00000000
    },
    {
      "iDVuVSjzYatSYq19YkPJdCH6SuA59WBWvC": 25000.00000000
    },
    {
      "i5Cwy6kMHCrLDWw31REbWxAnnZ2mHFUk5D": 25000.00000000
    }
  ],
  "initialcontributions": [
    0.00000000
  ],
  "gatewayconverterissuance": 100000.00000000,
  "idregistrationfees": 100.00000000,
  "idreferrallevels": 3,
  "idimportfees": 1.00000000,
  "notaries": [
    "iGTdav42siHgX96ybWn3pRTFxQgiUpZ9K9",
    "iDVuVSjzYatSYq19YkPJdCH6SuA59WBWvC",
    "i5Cwy6kMHCrLDWw31REbWxAnnZ2mHFUk5D"
  ],
  "minnotariesconfirm": 2,
  "currencyregistrationfee": 200.00000000,
  "pbaassystemregistrationfee": 10000.00000000,
  "currencyimportfee": 100.00000000,
  "transactionimportfee": 0.01000000,
  "transactionexportfee": 0.01000000,
  "gatewayconverterid": "iMCX2HwtnpXynPpXfnTATwPF5hz92bYCre",
  "gatewayconvertername": "bridge",
  "initialtarget": "000000ff0f000000000000000000000000000000000000000000000000000000",
  "eras": [
    {
      "reward": 0,
      "decay": 0,
      "halving": 0,
      "eraend": 0
    }
  ],
  "currencyidhex": "7733e5dcf2ed716dcb20ebf375ac45239c1902af",
  "fullyqualifiedname": "chipstensec",
  "currencynames": {
    "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq": "VRSCTEST"
  },
  "definitiontxid": "567cc5eb56134553ac1eb7da69e74b67fa503d390f128ef0fd569ddb8dd5e897",
  "definitiontxout": 1,
  "bestheight": 73219,
  "lastconfirmedheight": 73219,
  "bestcurrencystate": {
    "flags": 2,
    "version": 1,
    "currencyid": "iKRtCZT87icurxb1ECUHKBq4jcuCTUV4ty",
    "launchcurrencies": [
      {
        "currencyid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
        "weight": 0.00000000,
        "reserves": 1.00000000,
        "priceinreserve": 1.00000000
      }
    ],
    "initialsupply": 0.00000000,
    "emitted": 0.00000000,
    "supply": 21000000.00000000,
    "currencies": {
      "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq": {
        "reservein": 0.00000000,
        "primarycurrencyin": 0.00000000,
        "reserveout": 0.00000000,
        "lastconversionprice": 1.00000000,
        "viaconversionprice": 0.00000000,
        "fees": 0.00000000,
        "conversionfees": 0.00000000,
        "priorweights": 0.00000000
      }
    },
    "primarycurrencyfees": 0.00000000,
    "primarycurrencyconversionfees": 0.00000000,
    "primarycurrencyout": 0.00000000,
    "preconvertedout": 0.00000000
  },
  "lastconfirmedcurrencystate": {
    "flags": 2,
    "version": 1,
    "currencyid": "iKRtCZT87icurxb1ECUHKBq4jcuCTUV4ty",
    "launchcurrencies": [
      {
        "currencyid": "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq",
        "weight": 0.00000000,
        "reserves": 1.00000000,
        "priceinreserve": 1.00000000
      }
    ],
    "initialsupply": 0.00000000,
    "emitted": 0.00000000,
    "supply": 21000000.00000000,
    "currencies": {
      "iJhCezBExJHvtyH3fGhNnt2NhU4Ztkf2yq": {
        "reservein": 0.00000000,
        "primarycurrencyin": 0.00000000,
        "reserveout": 0.00000000,
        "lastconversionprice": 1.00000000,
        "viaconversionprice": 0.00000000,
        "fees": 0.00000000,
        "conversionfees": 0.00000000,
        "priorweights": 0.00000000
      }
    },
    "primarycurrencyfees": 0.00000000,
    "primarycurrencyconversionfees": 0.00000000,
    "primarycurrencyout": 0.00000000,
    "preconvertedout": 0.00000000
  }
}
```

**Cross-reference `startblock` value with current blockchain height.  Do not begin mining/staking until after this height!**


---

<h2 id="launch">Launching a PBaaS chain</h4>

<h3 id="launch-firstblock">Merge mine the first block on generated chain</h3>

Now that we have defined our chain's currency, generated and committed the launch parameters...  we need to merge mine at least the first block in conjunction with the launch system chain.

Only the first block must be merge mined.  Afterward, we can choose to merge mine or mine the newly launched chain independently.

To do this, we must be already mining on `VRSCTEST` in our example.  This can be performed with the following startup options (for solo mining):

```
./verusd -chain=vrsctest -gen -genproclimit=2 -mineraddress=RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph -mint &
```

Next, we must spin up our `chipstensec` daemon, and start mining.  We can either: a.) fetch the address generated on wallet initialization, or b.) import our WIF private key from `vrsctest` chain.

```
# Start chipstensec daemon
./verusd -chain=chipstensec &

# option a.) Fetch address generated on init
./verus -chain=chipstensec getaddressesbyaccount ""

# option b.) import WIF
./verus -chain=chipstensec importprivkey <WIF here>
```

Once we've fetched our new address, or imported a known address, we can restart our `chipstensec` daemon to begin merge mining with parent `vrsctest`:

```
# Stop chipstensec daemon
./verus -chain=chipstensec stop

# Restart with mining options
./verusd -chain=chipstensec -gen -genproclimit=2 -mineraddress=RYQbUr9WtRRAnMjuddZGryrNEpFEV1h8ph -mint &
```

If successful, you should see output in your terminal:

```
Merge mining chipstensec with vrsctest as the hashing chain
Found block 2885
staking reward 0.00010000 chipstensec!
POS hash: 00000000007fca5a2624d9276d2ddd7074f68fd32b9f6c81904d96b4a12f6301
target:   0000000002e3ae00000000000000000000000000000000000000000000000000

Block 2885 added to chain
```

Similar messages should be seen in `vrsctest` daemon output.  Once the first block is mined, running `vrsctest` daemon at the same time is optional.


