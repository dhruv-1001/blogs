# Setting up Bitcoin node

We will start with setting up a bitcoin node(testnet). This will help up us interact with the testnet blockchain, look at blocks and transactions, build and broadcast tranasctions. We will also learn why the blockchain is designed the way it is designed in this chapter, and will go in detail on those topics in upcoming blogs.

Through this blog you will learn how to use commands from `bitcoind` `bitcoin-cli`

## Setting up `bitcoind`, `bitcoin-cli`

### Downloading bitcoin-core

You can download bitcoin-core through this link - https://bitcoincore.org/bin . Right now the latest version available is bitcoin-core-24.0.1. 
```
SHA256SUMS                                         11-Dec-2022 19:32                2898
SHA256SUMS.asc                                     11-Dec-2022 19:32                9003
SHA256SUMS.ots                                     11-Dec-2022 19:32                 503
bitcoin-24.0.1-aarch64-linux-gnu.tar.gz            11-Dec-2022 19:32            44427716
bitcoin-24.0.1-arm-linux-gnueabihf.tar.gz          11-Dec-2022 19:32            40505998
bitcoin-24.0.1-arm64-apple-darwin.dmg              11-Dec-2022 19:33            29886464
bitcoin-24.0.1-arm64-apple-darwin.tar.gz           11-Dec-2022 19:33            33110137
bitcoin-24.0.1-powerpc64-linux-gnu.tar.gz          11-Dec-2022 19:33            50700679
bitcoin-24.0.1-powerpc64le-linux-gnu.tar.gz        11-Dec-2022 19:34            49621229
bitcoin-24.0.1-riscv64-linux-gnu.tar.gz            11-Dec-2022 19:34            44331462
bitcoin-24.0.1-win64-setup.exe                     11-Dec-2022 19:34            30913184
bitcoin-24.0.1-win64.zip                           11-Dec-2022 19:35            44329918
bitcoin-24.0.1-x86_64-apple-darwin.dmg             11-Dec-2022 19:35            32458752
bitcoin-24.0.1-x86_64-apple-darwin.tar.gz          11-Dec-2022 19:35            36529733
bitcoin-24.0.1-x86_64-linux-gnu.tar.gz             11-Dec-2022 19:35            46384770
bitcoin-24.0.1.tar.gz                              11-Dec-2022 19:36            11412628
bitcoin-24.0.1.torrent                             11-Dec-2022 19:36               39325
```

Download here according to your system. In my case, i'll be downloading bitcoin-24.0.1-arm64-apple-darwin.tar.gz as i'm on M1 macbook.

Or, you can download it using curl
```bash
# This downloads the file to your current working directory
curl -O https://bitcoincore.org/bin/bitcoin-core-24.0.1/bitcoin-24.0.1-arm64-apple-darwin.tar.gz
```

In case you downloaded the file in Downloads directory, for there blogs let's create a directory named bitcoin at `$HOME` path, and move the downloaded file there and extract the files.
```bash
# Creates bitcoin directory at $HOME path
mkdir ~/bitcoin
# Moves the downloaded file to $HOME/bitcoin
mv ~/Downloads/bitcoin-24.0.1-arm64-apple-darwin.tar.gz ~/bitcoin
```

### Adding `bitcoind` `bitcoin-cli` to path

Let's see where the binaries are in the files extracted, and add them to the path so that you can use these executables from anywhere. 
```bash
# Change directory to $HOME/bitcoin
cd ~/bitcoin
# Extract the files
tar -xvf bitcoin-24.0.1-arm64-apple-darwin.tar.gz
# List files
ls
bitcoin-24.0.1                           bitcoin-24.0.1-arm64-apple-darwin.tar.gz
# remove downloaded file
rm bitcoin-24.0.1-arm64-apple-darwin.tar.gz
# Locate binaries
ls /bitcoin-24.0.1/bin
bitcoin-cli    bitcoin-qt     bitcoin-tx     bitcoin-util   bitcoin-wallet bitcoind       test_bitcoin
# Add $HOME/bitcoin/bitcoin-24.0.1/bin to path (use .bash_profile in case you are using bash)
echo "export PATH=$PATH:~/bitcoin/bitcoin-24.0.1/bin" >> ~/.zshrc
source ~/.zshrc # or .bash_profile in case of bash
# checking if the path was added successfully
which bitcoin-cli
/Users/dhruvbaliyan/bitcoin/bitcoin-24.0.1/bin/bitcoin-cli
```

Now we can use these executables from any directory.

## Setting up testnet blockchain
For the blog purpose, i'm using testnet (which is what developers use to test some features, and it's small in size compared to mainnet which is arount 450 GB (2nd Feb 2023), while the testnet is around 28 GB)

### Setting up bitcoin.conf file

Let's start `bitcoind`, and download testnet blockchain. Run this command
```bash
bitcoind -chain=test
```
For now, let's stop using `command/ctrl + c`. You will see bunch of output, and let's focus on these lines
```bash
...
...
2023-02-02T17:27:24Z Default data directory /Users/dhruvbaliyan/Library/Application Support/Bitcoin
2023-02-02T17:27:24Z Using data directory /Users/dhruvbaliyan/Library/Application Support/Bitcoin/testnet3
2023-02-02T17:27:24Z Config file: /Users/dhruvbaliyan/Library/Application Support/Bitcoin/bitcoin.conf (not found, skipping)
...
...
```

These tell us what are some default directory for storing blockchain on where `bitcoind` is looking for the `bitcoin.conf` file. If you want to store the blockchain to some other location you would require to add `-datadir=<dir>` argument to `bitcoind` binary, or if you want to change the port, you would have to pass in the argument `-port=<port>`. <br>
You would be required to add these arguments every time you start `bitcoind`, so let's add `bitcoin.conf` file to at it's default location, which is available in the output above. <br>
For macOS it is `$HOME/Library/Application Support/Bitcoin/bitcoin.conf` and for linux it is `$HOME/.bitcoin/bitcoin.conf`. <br>

For now, let's create `bitcoin.conf` file with argument `-chain=test`, so that we don't have to pass in this argument everytime we run `bitcoind`.
``` bash
# for macOS
echo "chain=test" >> $HOME/Library/Application\ Support/Bitcoin/bitcoin.conf
# for linux
echo "chain=test" >> $HOME/.bitcoin/bitcoin.conf
```
If you want to further modify the config file, you can go to this site - https://jlopp.github.io/bitcoin-core-config-generator. You can check out what other arguments are there using `bitcoind --help`

Now let's run the node
```bash
bitcoind
```

This process will connect to different nodes, and start downloading block headers, and then the blocks with all transactions.
``` bash
...
...
2023-02-04T00:39:05Z Pre-synchronizing blockheaders, height: 2000 (~0.35%)
2023-02-04T00:39:05Z Pre-synchronizing blockheaders, height: 6000 (~1.06%)
...
...
2023-02-04T00:42:58Z Synchronizing blockheaders, height: 76041 (~12.94%)
2023-02-04T00:42:58Z Synchronizing blockheaders, height: 78041 (~13.24%)
...
...
```
What are blockheaders, transactions? We will discuss all that, in upcoming blogs. Let's look inside what a block is made up of.

### Playing with bitcoin-cli
We can intereact with the blockchain using bitcoin-cli. There are other ways we can interact with the blockchain like different languages using RPC if your node is on different system.

```bash
bitcoin-cli help
```
This lists all the available commands for bitcoin-cli, what we need right now is to look what a block is made up of.
```bash
bitcoin-cli getblockchaininfo
{
  "chain": "test",
  "blocks": 2105123,
  "headers": 2418834,
  "bestblockhash": "000000003fe9bdb5db4a164f29a8a2817ccf7a1c0564d8317db70e227cf41890",
  "difficulty": 1,
  "time": 1637681739,
  "mediantime": 1637676975,
  "verificationprogress": 0.9465394904211927,
  "initialblockdownload": true,
  "chainwork": "000000000000000000000000000000000000000000000611c539875485f42505",
  "size_on_disk": 30110762396,
  "pruned": false,
  "warnings": ""
}
```
You can see basic information about the blockchain like the chain type, blocks, headers. If you run this command again, you will see different block number as it's being downloaded and the chain is syncing up. <br>
Let's get the blockhash for the latest available block, which in my case is 2105123, and use that to get the block and see what's inside.
``` bash
bitcoin-cli getblockhash 2105123
000000003fe9bdb5db4a164f29a8a2817ccf7a1c0564d8317db70e227cf41890
bitcoin-cli getblock
{
  "hash": "000000003fe9bdb5db4a164f29a8a2817ccf7a1c0564d8317db70e227cf41890",
  "confirmations": 32011,
  "height": 2105123,
  "version": 536870912,
  "versionHex": "20000000",
  "merkleroot": "f9ac3035e2a4ec0ff1f47a646d158cd8034f070758d4b26fca4980b0bfc8afbc",
  "time": 1637681739,
  "mediantime": 1637676975,
  "nonce": 413158366,
  "bits": "1d00ffff",
  "difficulty": 1,
  "chainwork": "000000000000000000000000000000000000000000000611c539875485f42505",
  "nTx": 98,
  "previousblockhash": "0000000083c8ae0df0da76d8fd3bad466548c4a09b8270effa2f071e72f38dc4",
  "nextblockhash": "000000001b147d25c55e12c707e9859190aea3bf60ec5b4f32633d42fcf3affb",
  "strippedsize": 19456,
  "size": 36277,
  "weight": 94645,
  "tx": [
    "1b19cf9481607987e702511859063ef5e308af682034d2b7c95a309cf1695156",
    "3c1595083c0953b1dee6f3933e6247d577cb3acd0076c490456ad02d18e611e6",
    "aad6e89ed964ca983276687ef17f02ab675ed79be05dabc0ec7a7d38c47e43c3",
    "a26acb4a0f0d739d1783d5d06c443c8ac344d2d5e068406597b12c57d6efa8a6",
    "d4e5866b34a67ac215fa40770751bed6829ad72c2c2fa54c584361d715ee99e2",
    "79cbe9fc8d58bbb2242dbfd6d1378410a8512bcbe227679ba6731f71ff85c8d0",
    "aa940e6a094d7886244748982bab7619cbeb0aa48fc9f215be7bd3951d7af4a8",
    "06ce195f6cfb5b986c4492f04d255500180cccf88a9ca247252dd46e020bb789",
    "0f6cc4bfc537d6f15e7a5e6adf6621755f7ca41d0cf4ada45d16139e1047e29a",
    "829f95b19f6d5a48b222114c993f2d5954377e6951204aabb6440dafe0744532",
    "23958618165956ca16eeb2513e46378ddf4baecdba4661f269acac0e32dfb017",
    "bc6fe4d8b6b5bf15728ba803b96db596a4c4278b55d5e80da161fbbba93d4696",
    "8a0b087a1f8008bd5953a26acb5e8905ae8a971c665a92cf7c00012171fa3ebe",
    "46712015db48636f750053812c2de550d4a08983cf99a5f5960470069d59547c",
    "947e93f4de60ec652db009e7bf596a8c17244e1e3f27c0e9e57858950ac09558",
    "78d8c4da90660c868745a4bd2b2f7e51d7f5ea5a418ebcd80bfabf6bc1bb291f",
    "cf2b1e57f9882375e481f7b753139ee0c05329ab8f0adb2228b24cb6f160dd40",
    "6cac9b8be5b1669dc3d2de8daec8cc1b42997f53d980fe6d3df00d91bd088908",
    "ceccf4311c0a27b30140519974505f67b43cd70525608e59a597d2f412956370",
    "37ee3b737ade73b539cd6b0029a8fda9a35b98f52c0599542dcceee64605a791",
    "1ebe69f8e2f9958dbbff7253d79d0d1c2a96d0b160cf44b61efd52b635b3c277",
    "136b11139f494f2c28f2922cdb991cc475103dce4fff1787d32d111c1dd34c59",
    "bdc721bd4c534babc5b6f8405bc8eb8a1a43472f9593f54fe3dd0d1fecd3f0e4",
    "6bdb751cc687af29d302c0b4115483c076c3f786b568648782954dd12857e084",
    "e50e6773c4e18d116cc3851b12a281549ce3d5d9ebc00eb53bbfdb84e8ede6bc",
    "6007ca207042a83e1ea357670d4d37b0f04b2e524499aeac436143932a557d1a",
    "ba2c06b11da3b49e25126b535d27359b20e49b1757a2e2a89904fbfc69b2e335",
    "7b4d70f1543e0fa216c1a413717447181f92b16fc8c45c16015fcdbbb31a53a2",
    "06d0d17db274f5af2fa8ecd3d434808f98024f87e7b69ca28cce4d6d8aa39cbd",
    "a3eb628ec0f1bf7af0452ca2956a8805ebe6edcbd4643776b29dd5d4c4f01f1e",
    "a46c093059dda5e2cb2bc77d9bd6cc2ade6b09653c29bec04d6e39dbacb05c08",
    "e855a876fdddda3b56040b77b766bafb8e016f57e5eaa784ecbd6e24711fefdf",
    "aa937defceb162daf9dfca553db1d8e170facc93ea3cbcb5e95256e072b902c8",
    "4a1e1e2297f8d9369eadad385ad4db3344c9116c7064efc53ff272e2c4f5e592",
    "9c55da9eb9ae7dc85e1525e64e51953babbd27b4f086ead792b9315f3764d7b1",
    "eddde3f0522683475bd74bf6c524bb3e4ce7faa32acb164cd3c09bc14144c391",
    "c8193eec43b71df2757953888caa931b03036fa09659340c6a5b81a49439cd5e",
    "8a599a1339fe33cb232c041d2ac8f98d53d9cef03ee8097c221d8a782acb9f7c",
    "e05cbf55cf6425e8cdb51f1fa0ff439675a573b3b7bb27e41af550c373008ce3",
    "5ca49785f50a04e2b1e87262ddc3384caab05c67b7ecb313c79a968750aecda4",
    "94a0c4417959b8259e81082850b0e9e0df8d29b4e6dd68dc8163fb92dba770b3",
    "29c551da88e156a801240d0c475f41ee41e59d007003dd70322689a214a84b23",
    "f6bf764a98731c7cc4ee37d9be85599107155fdf458a028565dc022f116d35b5",
    "21f056577a30f9a7c5c429b34c02bd71e962be59bcde0c30cfb8e5f6ccbe322f",
    "cd1840daf94ee679d1dd1cef6c2aadf580d55d24b1a160475af24908f6154423",
    "aae35e9d7d85c75f9ce06e147be8b1c0499a2bf7b0f484728847772a38c5b49d",
    "03542e7694601f569f3b1727d2486e1b4de95db443c5e4ccfaae76a55b012df9",
    "542c3a02cb4905051716c4a6f888350a9575d6382b2dc1a828ce2ae5f5657920",
    "5708237e058c0de42b30ef58b595ed2a05eda41cf9fb10a5fb34a877dc96de70",
    "6211ec65713245083845549518e0fc333d85dae1ded756aa9eaf7ab38c3d5ba7",
    "cbb52a490011ef563775eb6c49344bb45852d6f12941cd1728467605057a9ff7",
    "34046ac0a5ffaa1bb7aa996245e62c2f9f37f0545dd63d2f75315f797861b4bb",
    "22400a1c45b57dcde7cb9e911eac1a4a165596906dd7264da6d7239ea8bb776e",
    "b59ba2c5010bb2ad887140b085f976b5822d612ee5f46ea677d57108e34ca4e5",
    "620714a97694346854b883a83b4fe64951fdbc96329311c6873a5a3edc270b56",
    "88f721b68d4ea86aea16217a7b63e6ffe83245c21b779a72728edf982d3fecc9",
    "30c603924a9cc8a0e3ae3e36ff9c2e3d442a11102288e5608b1004857b4f8d70",
    "0d7a1e8656e168f45372f26f657bed4cd93e80aca68d6dbb2f246a710a6c30cb",
    "a315f58e14e2d4944e5a49d852fa4564956eed59bc22bddc8e806a06164d7dcf",
    "22644bcab188194177009ba510d3d2d6f134d70732517316fda4822c168f1c4f",
    "4f48b790a1829cdbf84955595ee3f2ff18c5cb554d168ab16b42e98f078939fa",
    "ad1fdac196f62286445dbaeac2e65771814425898341e4f191e4f65c3a3ea553",
    "8a58d9002d8c838fdadb5ed1a9196d012fcea7ed1f85f3f78a4ecdc22734efd2",
    "46bd0475620346d49288c5a5d29cc8004ba5da01179150d8822eb42c566d61c3",
    "3e29bd2192b9db2427f8ca7dd1b6b0239b967ea679b86f113d592d111df008ab",
    "32bc266b0b03dd63b230cd56c7015cb929261043e1d7367dccdc2edc474f6141",
    "bd4c91f1e177d2544daa6d32179a166f352753e4cff07dd2a6b7f97d438fc170",
    "09c893b5def086d1a16077e772b300893030ae23c5fe51a0fcf133ca801d3d02",
    "265ce00d42a382aa15fd741abcedac10ec089be7c92b97b0976513215b65981b",
    "3c201eaa9121829d3348ada2fbd921cd2dc4738211f10fdb7be0d6f83218147e",
    "941dec9df8ac3a319f7a741b759ab5320c7bbea46bbc50e683ce0a92ed034c74",
    "4b2d3db8f73fcd0454e17a23c5abc29e1de60ec26cf24a30b5cdc0d812e8349e",
    "86ffb5ec8c23e284feadf685ac227c5cb94e58823a07524099a155d5e05b0b24",
    "e249878d8f666a11094970e39d5e0206a0b4ee929271bf7c9ec649e594c36878",
    "6a2150635243b19fe7af76ef51ab440818b440e58adc7a318ce308e60543ad4b",
    "56781d33e8b3858211b09f836c02063c2800db407c932ab0d62081e4f1c2b1ef",
    "dc04afdccdf187a069b293e8c5b9ae167833b351620e67733ecc73bd970da526",
    "4e6c9fc1b25740364063ec189d21f6c166a62d920f6fe60a87dba32d26dcca37",
    "a22d21df28175ce4ac8e66ba6b5a2729c0802ca8e21dc64c7f1c5258619e043a",
    "aa2e1fb62401f8c76c05839a2bd236a14b7bd9ceaa1e481c7102cc89acffce3d",
    "5e8065203018acc514bed239915e26faacc51996f30a1040409e63ba0e1d624a",
    "1482b4191ad69e2eacedc807b53037d05951acda6bef5afb8228329a54546e56",
    "e391375fcf2efcad1e11cf6c4e99fb657865d68b2c012af9c39902903be48a5c",
    "55fe9d259ba117a36384c9787f991aee1579f6e6fadcda75887272519ea3505e",
    "a8ec3454f323b2f38a2bcd6245054fdc6ad4fb5feaf9d8c35eb1609d3e4ca06c",
    "ae17683d555ab93dd802c247f3d91d0d045898ab21ca2cd5fc49dfacb597ebad",
    "631618fe426fa6465877056328be7b5fce5832c5a648a32b8ce181ba47d646da",
    "7b0d58b53576e4f12bb06c92a3b1f46b666929434968c1517e1b54c4c4101971",
    "00ec1035899993769339f692f48d63e5967eda12c3d3daff641c8d08661a9f81",
    "aa7f87ce90165e5af9221f734fdd2915bd6ddd655b6890fcd223a25d55e5b98b",
    "2fb5f3654d32de53d928dcbe2706a99bca9d6f4a56cba37c084952a7a8df1c8c",
    "c2b681bf62e5e40c1b04fffc2e23e4398b2f9e58e70e76d7611dd1e60b35d693",
    "59f09c9e5be84150927b11daac8c89fa43ab5adea554340746bd4cfcc06c8497",
    "dcf466d5518b45be53e84fd45fc32961a35c76abc6b6f6e599ff40b948d4cba8",
    "c7eb561f33315e249569e3f722c06c05893eaa8326f4da89e6b0c58de0ea33e0",
    "fd6aa21e66ce753d054274b3707468625c7ab3dab0f64da51ef450e5863b68f6",
    "358d729e4ae97560305dc725f6764919e9676398551981f0cee60fc665308ff9",
    "bca0694df982ed3706e077520e52b9c182b9a70073d859cfb33b27941deacaf9"
  ]
}
```
You get to see the inside of a block, we will learn what all this data mean. The transactions are not included here, just there txid is provided to us. Let's also see what's inside this last transaction.
``` bash
# bitcoin-cli getrawtransaction <txid> true <blockhash>
bitcoin-cli getrawtransaction bca0694df982ed3706e077520e52b9c182b9a70073d859cfb33b27941deacaf9 true 000000003fe9bdb5db4a164f29a8a2817ccf7a1c0564d8317db70e227cf41890
{
  "in_active_chain": true,
  "txid": "bca0694df982ed3706e077520e52b9c182b9a70073d859cfb33b27941deacaf9",
  "hash": "7dbe2cdc291c9c21f2d007fb23e0667d976ef4528e968a03c055cb094f658f74",
  "version": 2,
  "size": 224,
  "vsize": 143,
  "weight": 569,
  "locktime": 2105122,
  "vin": [
    {
      "txid": "d8831b03bb35428f063413cc8e68914237ce5608240ad01a406dca15fe6f8092",
      "vout": 1,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "txinwitness": [
        "3044022059c227aa740aac005a6d2ed7d96f5e1875107fa925f4743834a0beb08ca9e7730220445999fee0218ce6a3514cf6421f8cd9627f084d771fd51241c82f47717a1e2201",
        "03abcedff3947e415c9fc415c376885206f88e6bd6cbd14f5ac5849ec59fa99ed7"
      ],
      "sequence": 4294967294
    }
  ],
  "vout": [
    {
      "value": 0.01335435,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_HASH160 508a858fb18dbb69c658fb11f95db2076aa4d09b OP_EQUAL",
        "desc": "addr(2Mzb5zHgjVaovc46GHG8sgeusBAR6BopWki)#m9h54cga",
        "hex": "a914508a858fb18dbb69c658fb11f95db2076aa4d09b87",
        "address": "2Mzb5zHgjVaovc46GHG8sgeusBAR6BopWki",
        "type": "scripthash"
      }
    },
    {
      "value": 0.00100000,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_HASH160 df2ef3b876016eb85b2d6d296ab9be95a67033d9 OP_EQUAL",
        "desc": "addr(2NDbJvM4F6H43quQLbJahgRbd1Q1e3yzCVd)#f74hga7v",
        "hex": "a914df2ef3b876016eb85b2d6d296ab9be95a67033d987",
        "address": "2NDbJvM4F6H43quQLbJahgRbd1Q1e3yzCVd",
        "type": "scripthash"
      }
    }
  ],
  "hex": "0200000000010192806ffe15ca6d401ad00a240856ce374291688ecc1334068f4235bb031b83d80100000000feffffff028b6014000000000017a914508a858fb18dbb69c658fb11f95db2076aa4d09b87a08601000000000017a914df2ef3b876016eb85b2d6d296ab9be95a67033d98702473044022059c227aa740aac005a6d2ed7d96f5e1875107fa925f4743834a0beb08ca9e7730220445999fee0218ce6a3514cf6421f8cd9627f084d771fd51241c82f47717a1e22012103abcedff3947e415c9fc415c376885206f88e6bd6cbd14f5ac5849ec59fa99ed7221f2000",
  "blockhash": "000000003fe9bdb5db4a164f29a8a2817ccf7a1c0564d8317db70e227cf41890",
  "confirmations": 313712,
  "time": 1637681739,
  "blocktime": 1637681739
}
```

Great! Now you know, how to read data from the blockchain. Now what all these fields means, we will learn that in upcoming blogs.