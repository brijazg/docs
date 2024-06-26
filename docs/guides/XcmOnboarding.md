# ⛓ Establish XCM Communication

## Why Open HRMP Channel With Calamari

A parachain might want to open a HRMP (Horizontal Relay-chain Message Passing) channel with Calamari for various reasons, such as:

1. Data sharing and collaboration: By opening an HRMP channel, parachains can exchange data with one another in real-time, facilitating collaboration and enabling them to share and utilize each other's resources.

2. Interoperability: HRMP channels allow parachains to interoperate with one another. This interoperability allows parachains to leverage the unique features of each other's chains, such as the zk-enabled privacy features of Calamari, and build more complex and advanced applications.

3. Scalability: HRMP channels enable parachains to scale horizontally by allowing them to communicate and coordinate with other chains in the same network. This communication and coordination can help manage the load on the network and distribute it more evenly across the system, resulting in increased scalability and throughput.

Overall, opening an HRMP channel with Calamari can bring a range of benefits and opportunities for both chains, enabling us to collaborate, interoperate, and build more novel decentralized applications.

## Process Overview:

* Test XCM between Calamari and your runtime locally with polkadot-launch.

* Become a parachain on Manta Network's internal Kusama relay chain.

* Calculate and fund your parachain's sovereign account on the Kusama relay chain.

* Open HRMP channels with Calamari.

* Assets registrations.

* Complete XCM tests between parachains.

* Next steps discussion.

* Manta Team Contact:
    - Georgi (XCM Eng): @Ghz (Telegram)
    - Shumo (Co-founder, Tech.): @xstec (Telegram)

## Local XCM Integration

- First you can test the integration locally. For that you can download the latest manta binary from the Releases page.
- Then use polkadot-launch to launch a `calamari-local` or `calamari-dev` network for testing.
- You will also need to launch a `rococo-local` relay chain using the latest release of Polkadot.
- Here's a reference polkadot-launch config for [calamari-dev](XcmOnboarding#example-polkadot-launch-config).
- Please let us know if there's a specific branch of your codebase that we should test with.

## XCM Integration on Manta Network's Internal Testnet

- The next step of the Calamari integration is to integrate with our parachain (Dolphin), which runs on an internal Kusama-based relay chain. As part of this integration, you’ll need to register an HRMP channel with Dolphin.

### Dolphin Ecosystem Data

- [Kusama endpoint](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fws.internal.kusama.systems#/explorer)
- [Dolphin endpoint](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fws.calamari.seabird.systems#/explorer)
- [Dolphin faucet](https://discord.com/channels/795390654628102165/1055864933692219453/1058294885427466291)

### Sync Node & Request A Slot

- To sync your node, you can use the following [relay chain spec](https://manta-ops.s3.amazonaws.com/kusma-internal/kusama-internal.json) (note: relay chain is Kusama based, and will probably take a few hours to sync)
- Register your parachain on Kusama. For that you will need to contact someone on our team and provide state/wasm file to be registered.

## Calculate and Fund your Parachain's Sovereign Account

- To calculate your Parachain’s Sovereign account, you can use the following tool: [https://github.com/Manta-Network/Dev-Tools/tree/main/caclulate-sovereign-account](https://github.com/Manta-Network/Dev-Tools/tree/main/caclulate-sovereign-account)

- Make sure you run the command by providing the parachain ID (flag  –paraid NUMBER) that you’ve selected on the relay chain. For example, Dolphin’s Sovereign account for both the relay chain and other parachains can be obtained with:

```
ts-node calculateSovereignAddress.ts --paraid 2084
```
- The result will be:
```
Sovereign Account Address on Relay: 0x7061726124080000000000000000000000000000000000000000000000000000
Sovereign Account Address on other Parachains (Generic): 0x7369626c24080000000000000000000000000000000000000000000000000000
Sovereign Account Address on Dolphin: 0x7369626c24080000000000000000000000000000
```

- Once you’ve got your `Sovereign Account`’s address, please ask someone on our team to fund it.

## Create HRMP Channel with Dolphin
### Get the Relay Encoded Call Data to Open HRMP Channel.

- Once your parachain is onboard, you need to create the HRMP channel between your Parachain and Dolphin.
- The first step is to get an encoded call data from the relay chain. The extrinsic contains the target parachain ID, max number of messages, and max message size, described in the next bullet.
- In PolkadotJS app, switch to the relay chain network. Go to Developer -> [Javascript section](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fws.internal.kusama.systems#/js). Run the following code, note to replace the demo recipient para id with your own:

```
const tx = api.tx.hrmp.hrmpInitOpenChannel(2084, 8, 1024);
console.log(tx.toHex());
```

- The result will be like:

`0x3c040x1700240800000800000000040000`, remove the leading hex `3c04`, and so the encoded result is `0x1700240800000800000000040000`.

### Send XCM to Relay Chain

- The next step is to build and send an XCM message to the relay chain that will request a channel to be opened through the relay chain. This XCM message needs to be sent from the root account (either SUDO or via governance). The message can be broken down in the following elements:
    1. Withdraw asset: take funds out of the Sovereign Account of the origin parachain (in the relay chain) to a holding state
    2. Buy execution: buys execution time from the relay chain, to execute the XCM message
    3. Transact: provides the call data to be executed
    4. Deposit asset (optional): refunds the leftover funds after the execution. If this is not provided, no refunds will be carried out
- Therefore, to build/send this XCM, you need to:
    1. Polkadot.js Apps in your parachain -> extrinsics
    2. Set the following parameters: polkadotXcm -> send
    3. The destination has to be the relay chain, for dest (V1) set:
    `{ parents:1, interior: Here }`
    4. For the message (V2), you’ll be adding 4 items (described before):
        1. WithdrawAsset { id: Concrete { parents: 0, interior: Here}, Fungible: 1000000000000 }
        2. BuyExecution { id: Concrete: {parents: 0, interior: Here}, Fungible: 1000000000000, weightLimit: Unlimited }
        3. Transact { originType: Native, requireWeightAtMost: 1000000000, call: XcmDoubleEncoded: { encoded: RelayEncodedCallData } }
            Note: you need to provide the encoded call data obtained before
        4. RefundSurplus
        5. DepositAsset: { assets: Wild { Wild: All }, maxAssets: 1, beneficiary: { parents: 0, interior: X1 { X1: AccountId32 { network: Any, id: SovereignAccountonRelay } } } }
    
    **Note:** The values used above are for reference to be used in this testing environment, do not use these values in production!
    
- Once this message is sent, the relay chain should execute the content and the request to open the channel with Dolphin
- **Please let us know once you’ve requested opening the channel because the request needs to be accepted by Dolphin.**

Here's an example of the fully formed extrinsic:

![https://i.imgur.com/GUN8qJd.png](https://i.imgur.com/GUN8qJd.png)

![https://i.imgur.com/3ONY21d.png](https://i.imgur.com/3ONY21d.png)

## Accepting HRMP Channel with Calamari

- Channels are one way. This means that if you open a channel with Dolphin, it will allow you only to send tokens from your parachain to Dolphin. There needs to be a channel that Dolphin will request to send back tokens, and you need to accept.
- The process of accepting the channel is similar to the one for opening, meaning that you have to construct an encoded call data in the relay chain, and then get it executed via an XCM from your parachain.

### Get the Relay Encoded Call Data to Accept HRMP Channel

- To get an encoded call data from the relay chain, to accept a channel request with a target parachain, take the following steps:
- In PolkadotJS app, switch to the live Polkadot/Kusama network. Go to Developer -> [Javascript section](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fws.internal.kusama.systems#/js). Run the following code, note to replace the demo recipient para id with your own:
```
const tx = api.tx.hrmp.hrmpAcceptOpenChannel(2084);
console.log(tx.toHex());
```

- The result will be like:

`0x1c04170124080000`, remove the leading hex `1c04`, and the encoded result is `0x170124080000`.

### Send XCM to Relay Chain

- The steps are the same as before (when making the request to open a channel). The main difference is in the `Transact` item, where you need to provide the encoded call data calculated above. This XCM message needs to be sent from the root account (either SUDO or via governance):
```
Transact { originType: Native, requireWeightAtMost: 1000000000, call: XcmDoubleEncoded: { encoded: RelayEncodedCallData } }
```
    
**Note**: The values used above are for reference to be used in this testing environment, do not use these values in production!
    
## Assets Registrations

### Registering your Asset on Dolphin

- Once the channel is opened, we need to register the asset that will be transferred to Dolphin. For that, we need the following information:
    1. `MultiLocation` of your asset (as seen by Dolphin). Please indicate parachain ID and the interior (if you use any pallet index, general index, etc)
    2. `Asset Name`
    3. `Asset symbol`
    4. `Number of decimals`
- Please write this information as a comment in this issue.
- We will confirm once the asset is registered. We will also set an arbitrary UnitsPerSecond, which is the number of tokens charged per second of execution of the XCM message.
- After the asset is successfully registered, you can try transferring tokens from your parachain to Dolphin.
- For testing, please also provide your Parachain WS Endpoint so we can connect to it. Lastly, we would need some funds to the following account:
    
    `5CacAW3K4gq3Ufv2dAqUFYWKoqJcQaFu346ahesmt4sua7Xx`
    
- If you need KMA tokens (the native token for Dolphin) to use your parachain's asset, you can get some from our Discord Bot - We can also provide you with some if you give us your address

### Registering Calamari’s Token on your Parachain

- To register our KMA token on your parachain, you can use the following MultiLocation:

`{ "parents": 1, "interior": {"X1": { "Parachain": 2084 }}`

- And the following metadata:

```
Name: Calamari
Symbol: KMA
Decimals: 12
```

- Note: Calamari MultiLocation is different!

### Example Polkadot Launch Config

```
{
	"relaychain": {
		"bin": "./polkadot",
		"chain": "rococo-local",
		"nodes": [
			{
				"name": "alice",
				"wsPort": 9944,
				"port": 30444,
				"flags": [
					"--rpc-cors=all",
					"--execution=wasm",
					"--wasm-execution=compiled",
				]
			},
			{
				"name": "bob",
				"wsPort": 9955,
				"port": 30555,
				"flags": [
					"--rpc-cors=all",
					"--execution=wasm",
					"--wasm-execution=compiled",
				]
			},
			{
				"name": "charlie",
				"wsPort": 9966,
				"port": 30666,
				"flags": [
					"--rpc-cors=all",
					"--execution=wasm",
					"--wasm-execution=compiled",
				]
			}
		],
		"genesis": {
			"runtime": {
				"runtime_genesis_config": {
					"configuration": {
						"config": {
							"validation_upgrade_frequency": 10,
							"validation_upgrade_delay": 10
						}
					}
				}
			}
		}
	},
	"parachains": [
		{
			"bin": "./manta",
			"chain": "calamari-dev",
			"nodes": [
				{
					"wsPort": 9801,
					"port": 31201,
					"name": "alice",
					"flags": [
						"--rpc-cors=all",
						"--rpc-port=9971",
						"--execution=wasm",
						"--wasm-execution=compiled"
					]
				}
			]
		}
	],
	"hrmpChannels": [
	],
	"types": {},
	"finalization": false
}
```

## Next Steps - Calamari & Manta

* The following items must have been completed and fully tested with Dolphin before proceeding with an XCM integration on Calamari (and Manta in the future):

    1. Bi-directional HRMP channels between Dolphin and your parachain
    2. Bi-directional asset registration (KMA token and the token of your parachain)
    3. Both teams must have successfully tested asset transfers through Polkadot.js Apps
* Once everything is successful we can plan governance proposals to open HRMP channels and asset registrations on the Kusama parachains, as well as additional marketing initiatives if relevant.
