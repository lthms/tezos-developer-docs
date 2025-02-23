---
title: "Part 2: Accessing wallets and minting NFTs"
authors: 'Yuxin Li'
last_update:
  date: 15 November 2023
---

Accessing the user's wallet is essential before your application can engage with the Tezos blockchain. It enables your app to view the tokens within the wallet and request the user to initiate transactions. However, it's important to note that accessing the wallet doesn't grant your app direct control over it.

## Connecting to the user's wallet

In this section, you add code to connect to the user's wallet with the `TezosToolkit` and `BeaconWallet` objects.

IMPORTANT: however you design your app, it is essential to use a single instance of the `BeaconWallet` object.
It is also highly recommended use a single instance of the `TezosToolkit` object.
Creating multiple instances can cause problems in your app and with Taquito in general.

1. Open the file named `src/App.svelte` and add the following code in the `<script lang="ts">` section after we initialize and set up the Ghostnet endpoint:

   ```javascript
   <script lang="ts">

   let wallet;
   let address;
   let balance;
   let statusMessage = "Mint an NFT";
   let buttonActive = false;

   const connectWallet = async () => {
     try {
       const newWallet = new BeaconWallet({
         name: "Simple NFT app tutorial",
         network: {
          type: NetworkType.GHOSTNET,
        },
       });
       await newWallet.requestPermissions();
       address = await newWallet.getPKH();
       const balanceMutez = await Tezos.tz.getBalance(address);
       balance = balanceMutez.div(1000000).toFormat(2);
       buttonActive = true;
       wallet = newWallet;
     } catch (error) {
       console.error("Error connecting wallet:", error);
     }
   };
   </script>
   ```

   The `connectWallet` function creates a `BeaconWallet` object that represents the user's wallet.
   It provides a name for the app, which appears in the wallet UI when it asks the user to allow the connection.
   It also includes the network to use, such as the Tezos main network or test network.

1. Add the following code in `<script lang="ts">` to disconnect the wallet:

   ```javascript
   const disconnectWallet = () => {
     wallet.client.clearActiveAccount();
     wallet = undefined;
     buttonActive = false;
   };
   ```

   The `disconnectWallet` function runs these steps to disconnect the wallet and reset the state of the app:

   1. It closes the connection to the Beacon SDK with the `wallet.client.clearActiveAccount()` command.
   1. It nullifies the wallet reference by setting `wallet` to `undefined`.
   1. It deactivates the associated button or functionality by setting the `buttonActive` flag to false.

1. Add this code inside `<main>`, which creates a button that the user can click to connect or disconnect their wallet:

   ```html
   <main>
     <h1>Simple NFT dApp</h1>

     <div class="card">
       {#if wallet}
         <p>The address of the connected wallet is {address}.</p>
         <p>Its balance in tez is {balance}.</p>
         <button on:click={disconnectWallet}> Disconnect wallet </button>
         <button on:click={requestNFT}>{statusMessage}</button>
       {:else}
         <button on:click={connectWallet}> Connect wallet </button>
       {/if}
     </div>
   </main>
   ```

   Now you have functions that allow your application to connect and disconnect wallets.

## Design considerations

Interacting with a wallet in a decentralized application is a new paradigm for many developers and users.
Follow these practices to make the process easier for users:

- Let users manually connect their wallets instead of prompting users to connect their wallet immediately when the app loads.
Getting a wallet pop-up window before the user can see the page is annoying.
Also, users may hesitate to connect a wallet before they have had time to look at and trust the application, even though connecting the wallet is harmless.

- Provide a prominent button to connect or disconnect wallets.

- Put the button in a predictable position, typically at the top right or left corner of the interface.

- Use **Connect** as the label for the button.
Avoid words like "sync" because they can have different meanings in dApps.

- Display the status of the wallet clearly in the UI.
You can also add information about the wallet, including token balances and the connected network for the user's convenience, as this tutorial application does.
Showing information about the tokens and updating it after transactions allows the user to verify that the application is working properly.

- Enable and disable functions of the application based on the status of the wallet connection.
For example, if the wallet is not connected, disable buttons for transactions that require a wallet connection.

## Minting NFTs

1. Still in `App.svelte`, add the following code to the` <script lang="ts">` section to initialize a deployed contract address. This contract has multiple entrypoints that allow us to interact with the Tezos blockchain, such as `mint`.

   ```javascript
   const contractAddress = "KT1W8FrDRM28BGy1VVKXfN9L61jW1dgAjHQi"
   ```
1. Add this code to build the function structure
   ```javascript
   const requestNFT = async () => {
    }
    ```
1. Add the following code inside the `requestNFT` function to set up button state:

   ```javascript
    if (!buttonActive) {
      return;
    }
    buttonActive = false;
    statusMessage = "Minting NFT...";
   ```
  This asynchronous function, requestNFT, checks if a button (likely associated with minting an NFT) is active, and if so, it deactivates the button and sets a status message indicating that the NFT minting process has begun.

1. Add the following code inside the `requestNFT` function to define the metadata for a specific NFT.

   ```javascript
    const metadata = "7b226172746966616374557269223a22697066733a2f2f516d57476342434c516132387955505634355172444e47545a6e6a56784b6434416546565a4233666166486f5950222c2261747472696275746573223a5b5d2c2263726561746f7273223a5b22747a3157584b32795776416861466458456a73587548656e6667664472396d7157694277225d2c2264617465223a22323032332d30342d31335431373a32343a35312e3035353930335a222c22646563696d616c73223a302c226465736372697074696f6e223a22547a6f7563616e206973206120736d616c6c20636f6c6c656374696f6e206f662066756e6e7920746f7563616e73206f6e207468652054657a6f7320426c6f636b636861696e2e222c22646973706c6179557269223a22697066733a2f2f516d634d484a514c475444646a363873554d4b42687a3669464538326f36504144386e61755a6b4b4b43426d7173222c22666f726d617473223a5b7b2264696d656e73696f6e73223a7b22756e6974223a227078222c2276616c7565223a2235313278363430227d2c2266696c654e616d65223a2261727469666163742e706e67222c2266696c6553697a65223a3232383532332c226d696d6554797065223a22696d6167652f706e67222c22757269223a22697066733a2f2f516d57476342434c516132387955505634355172444e47545a6e6a56784b6434416546565a4233666166486f5950227d2c7b2264696d656e73696f6e73223a7b22756e6974223a227078222c2276616c7565223a2235313278363430227d2c2266696c654e616d65223a22646973706c61792e706e67222c2266696c6553697a65223a3232373534322c226d696d6554797065223a22696d6167652f706e67222c22757269223a22697066733a2f2f516d634d484a514c475444646a363873554d4b42687a3669464538326f36504144386e61755a6b4b4b43426d7173227d2c7b2264696d656e73696f6e73223a7b22756e6974223a227078222c2276616c7565223a2233323078343030227d2c2266696c654e616d65223a227468756d626e61696c2e706e67222c2266696c6553697a65223a3131323837382c226d696d6554797065223a22696d6167652f706e67222c22757269223a22697066733a2f2f516d593334616a6872757a66784a3979387358704b4365625a4a524638435a69443270487931534254375a4b5475227d5d2c22696d616765223a22697066733a2f2f516d634d484a514c475444646a363873554d4b42687a3669464538326f36504144386e61755a6b4b4b43426d7173222c226d696e746572223a224b543141713477576d56616e70516871345454666a5a584235416a467078313569514d4d222c226d696e74696e67546f6f6c223a2268747470733a2f2f6f626a6b742e636f6d2f6d696e745632222c226e616d65223a22547a6f7563616e20233438222c22726967687473223a224e6f204c6963656e7365202f20416c6c20526967687473205265736572766564222c22726f79616c74696573223a7b22646563696d616c73223a342c22736861726573223a7b22747a3157584b32795776416861466458456a73587548656e6667664472396d7157694277223a3530307d7d2c2273796d626f6c223a224f424a4b54434f4d222c2274616773223a5b22746f7563616e222c2262697264222c226d6f6465726e222c22617274225d2c227468756d626e61696c557269223a22697066733a2f2f516d593334616a6872757a66784a3979387358704b4365625a4a524638435a69443270487931534254375a4b5475227d"

    const metadatamap = new MichelsonMap()
    metadatamap.set('',metadata)
    ```
    Metadata for NFTs provides detailed information about the token, describing its unique attributes. This context helps in distinguishing each NFT.

    In the provided code, a new MichelsonMap instance is initialized to handle Tezos's native map data type, a structure used to store key-value pairs where each key is unique. The MichelsonMap utility allows for easier interaction with Tezos's smart contract language, Michelson. By setting the metadata with an empty string as its key, the data is prepared for either storage or use within the Tezos contract.

  1. Add the following code inside the `requestNFT` function to access the wallet and mint an NFT.

    ```javascript
    try {
      console.log("setting the wallet");
      Tezos.setWalletProvider(wallet);

      console.log("getting contract");
      const contract = await Tezos.wallet.at(contractAddress);
      console.log("minting");
      const op = await contract.methods.mint(metadatamap,address).send();

      console.log(`Waiting for ${op.opHash} to be confirmed...`);
      const hash = await op.confirmation(3).then(() => op.opHash);
      console.log(`Operation injected: https://ghost.tzstats.com/${hash}`);
    } catch (error) {
      console.error("Error minting NFT:", error);
    } finally {
      statusMessage = "Mint another NFT";
      buttonActive = true;
    }
   ```
   Taquito can fetch the user's tez balance from the connected wallet.
   To get the tez balances, the app uses the [Beacon SDK](../../../dApps/wallets).
   The `mint` function takes two parameters: a metadata for the NFT and the user's wallet address.


You'll start the app and mint NFTs with your dApp in the next section!
