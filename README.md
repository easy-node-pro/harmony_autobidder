# harmony_autobidder

## About

This is a version of Harmony's Autobidder with updates by Patrick @ EasyNode.pro

## Setup
### Clone Repo
After logging into the system you'll run autobidder on, clone the repo and set it to our production branch:
```
git clone https://github.com/easy-node-pro/harmony_autobidder.git
cd harmony_autobidder
git checkout easy-prod
```

### Install requirements
```
$ pip3 install -r requirements.txt
```

### Download the hmy client if you're not running autobidder on a validator
```
$ cd ~/
$ mkdir ~/harmony
$ cd ~/harmony
$ curl -LO https://harmony.one/hmycli && mv hmycli hmy && chmod +x hmy
```
You don't have to use `~/harmony` but that's how the service is currently configured.
See `config.py` if you'd like to modify the path settings. Also make sure `$HOME/harmony` is 
in your system path.

### Create your passphrase.txt file
Autobidder needs the wallet password on this server to change keys.

```
$ echo 'passphrase' > ~/harmony/passphrase.txt
$ chmod og-rw ~/harmony/passphrase.txt
```

### Copy BLS .key and .pass files into .hmy/allkeys
Copy all of your keys for all shards into the allkeys folder:

```
$ mkdir ~/harmony/.hmy/allkeys
$ cp ~/harmony/.hmy/blskeys/*.* ~/harmony/.hmy/allkeys
$ chmod og-rw ~/harmony/.hmy/allkeys/*.*
$ # Copy any other bls .key and .pass files from other nodes to ~/harmony/.hmy/allkeys as well
```

### Setup config.py 
1. If you are running a node on shard 0 on the same machine you can set
`USE_REMOTE_NODE = False` otherwise you need to set it to `True` since a lot of RPCs we use only work on shard 0.
2. Set `VALIDATOR_ADDR` to your validator's address
4. Configure `TARGET_SLOT` to your liking. The bot will try to adjust the keys in a way that puts you just at or above `TARGET_SLOT`.
5. If `PREVENT_INEFFICIENT_BID` is set to True the bot will try it's hardest to never overbid while keeping you elected.   
6. `MAX_VALIDATORS_PAGES` is the maximum number of validator pages the client can scan for downloading validator info
7. You still need to make sure your nodes are setup to run with all the BLS keys assigned to the validator between epoch changes

### Update bls_keys.py
1. Update `BLS_KEYS` to contain all your bls key string in order of highest priority. The keys will be added and removed as if this list was a stack.
```
BLS_KEYS = (
    "f82d30adadabaaaeba00406a5d607134343888dccf4fc45bdc22f02ad10df3ddeed1656a2a253262dae92095297e3f84", 
    "8be0121e5282071287886b07af678eefb67737f12d306c370f588d2612bf3020f9dc97fce63ee86c0ee3cd2153e43f90", 
    "2bcee590cb191a7bff14641fb0afdd6e07ee83b45e7247fce9dcf0fc10d8c7c3560dad54891cdf8bffff36e4a2c24f04", 
    "1c1d7ca4562ddda2cb456a766f86f487ff30bf2d3cbb692f359a29438b04cb6b79daf9241c44b514893076a14bea9984",  
)
```
2. The bls_keys.py file was separated out in order to avoid merge conflicts. If you want to do development 
you can run the following command so git will not track changes to the file.
```
git update-index --skip-worktree bls_keys.py
```

### vStatsBot Integration By Fortune Validator
Add your vStatsBot token acquired by running /token command on the bot to the autobidder config.py file. 
Set the two variables to True or False (VSTATS_ALERT_REMOVE_KEY, VSTATS_ALERT_OUT_OF_ELECTION)

### Start program with target.py
You can copy target.py to your home directory or launch it anywhere with:

```
python3 ~/harmony_autobidder/target.py 500
```

The script will launch your autobidder in a tmux session named autobidder. It will check for one already running on startup, stop it, and start a new one with your new slot.  

To exit a running tmux session and leave it active, send the commands `ctrl+b d` to exit back to your normal terminal. Verify your autobidder is still running with `tmux ls` and check for the autobidder session.  

To reconnect to a running session, send the command `tmux attach -t autobidder`  

### Monitoring the autobidding tmux session
```
''''
384: kaparnos (1019000)
385: FNHarmonyOS (1000000)
386: BBIT (545000)
387: FANIN5050-000001 (500452)
388: Total Harmony (488966)
389-404: InfStones (444438)
405: HarmonyFans - 1% FEE 🚀 (350337)
406: CONTABO-1#I´m sorry to all my Delegators,please undelegate ASAP.I´m closing this Validator#With 99.76% & fee 0.1%  :-((( # (281725)
407-410: Omen OS (DMun) (252500)
411: Coff33Blak (243647)
Name: RoboValidator.com
Current slots: 301-302
Current epoch uptime: 98.86814%
Current bid: 5506999
If we lower the bid by adding key the slots will be: 366-368
If we increase the bid by removing a key the slots will be: 28
....
```
The bot will poll for validator information every several seconds and output the state again if anything changes.

### termius startup snipped
This snippet can be used in termius or on it's own to ask for a slot and start/restart the autobidder session with your new target:
```
echo Enter your target slot number: && read slot_number && cd ~/harmony_autobidder/ && python3 target.py $slot_number
```

## Script options
```
$ python3 autobid.py --help
usage: autobid.py [-h] [-o] [-l] [-j] [-e]

Autobidding service for harmony validators. python autobid.py

optional arguments:
  -h, --help          show this help message and exit
  -o, --once          Whether or not to run once
  -l, --html          Whether or not to print html
  -j, --json          Whether or not to output json
  -e, --raise_errors  Whether or not to let errors stop execution
  -d, --disable_bidding  If set then disable bidding and just output information
```
