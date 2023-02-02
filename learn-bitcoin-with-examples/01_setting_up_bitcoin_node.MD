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
For macOS it is `$HOME/Library/Application Support/Bitcoin/bitcoin.conf` and for linux it is `$HOME/.bitcoin/bitcoin.conf`.

