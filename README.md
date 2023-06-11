# Solana NFT Collection Tutorial

![1](https://github.com/gecginserhat/Solana_NFT/assets/74310970/32ddb67f-6455-4d81-89f4-ba2b5174312c)

So Let's start
We will create a site where we can mint NFT on Solana. In this project, we will use Metaplex and Candy Machine. This way you can earn Millions of Dollars by developing from NFTs.

![2](https://github.com/gecginserhat/Solana_NFT/assets/74310970/c6d545a2-aae9-41d3-8cf8-e7d6f7ef8545)

## About Solana

Solana is a **super fast** blockchain with **super low** fees. We’ve already covered some of the benefits of this in our Solana Pay and Anchor tutorials. It’s great for NFT collections too because it costs pennies to mint an NFT and not much to launch the collection. None of that gas war nonsense here! And because it’s so fast we’ll be able to quickly show users exactly what they’ve minted :)

**What is NFT?**

Non-Fungible Token (NFT), in its shortest definition, is a **unique digital asset.** It represents many unique digital assets, from collector's items to virtual shoes, from virtual game content to digital properties.

## About Metaplex

Metaplex is the standard for creating NFTs on Solana. Candy Machine is one of the Solana programs written by them. We don’t need to write our own program to create NFTs on Solana, we just use theirs! They also provide a super handy CLI that lets us set up our NFT collection, and a TypeScript SDK to interact with our collection from a web app.

Requirements for the project

**Install Solana CLI**

-If you already have Solana CLI installed you can skip over this!

The Solana CLI lets us create a Solana account locally and interact with the blockchain. We won’t be using it a lot directly, but Metaplex uses it under the hood.

Head to https://docs.solana.com/cli/install-solana-cli-tools#use-solanas-install-tool. There are separate instructions for Windows and macOS/Linux. The command will include a version number, just use the one on the site to get the current recommended version.

Once it’s installed check the version:

```
$ solana --version
solana-cli 1.10.31 (src:77a40cd8; feat:4192065167)
```

Again just make sure your version is no older than mine. You won’t need to change versions during this tutorial, but you can always install the latest version using `solana-install update`

**Generate a Solana keypair**

-If you already have a Solana keypair locally then you can skip over this!

Now that we’ve got Solana CLI installed, we’re going to use it to generate a keypair. A keypair is just a public key and a private key. The public key can be given to other people or to apps, but only the private key allows making payments from the account. BTW account and wallet are other terms you’ll hear, you can just think of them as being a keypair. Every interaction with Solana is done by an account, so Anchor needs us to have one configured.

Generate it with `solana-keygen new` 

The BIP39 passphrase is optional. We’re only going to use this account locally so I didn’t use one, but you should for a real account!

There are two interesting parts of this output. Firstly it tells us where it’s written the keypair: `/Users/nfts-tutorial/.config/solana/id.json` in my case. There’s nothing really special here, it’s just a JSON file holding an array of small numbers, which represent bytes. A byte is just a number from 0-255. In Javascript land it’s a `Uint8Array`. That byte array stores both the private key and the public key.

All the Solana CLI commands will use this keypair by default now. Let’s take a quick look at the CLI we just installed.

First we can confirm it’s using the address we generated:

`$ solana address`

Now let’s check our SOL balance:

`$ solana balance`

Unsurprisingly our new account doesn’t have any. Let’s switch to devnet :

```
$ solana config set --url devnet
Config File: /Users/nfts-tutorial/.config/solana/cli/config.yml
RPC URL: https://api.devnet.solana.com
WebSocket URL: wss://api.devnet.solana.com/ (computed)
Keypair Path: /Users/nfts-tutorial/.config/solana/id.json
Commitment: confirmed

$ solana balance
0 SOL
```

Still no SOL! Let’s get some:

```
$ solana airdrop 2
Requesting airdrop of 2 SOL

Signature: 34gtEy4rxXMengtTUgCYCcMnTru6AYpnbFyStCFYFS5QVpcNZ4Y66Vk4mXcf5ALUKzU5FGuEiymSGL74SXdETVb8

```

Now we have some SOL.. on devnet. Unsurprisingly but sadly, that airdrop doesn’t work on mainnet! We’ll be working on devnet for this tutorial though.

**Install Metaplex Sugar CLI**

Sugar is the new candy machine CLI from Metaplex. It makes it super straightforward to create and deploy an NFT collection.

Head to https://docs.metaplex.com/developer-tools/sugar/overview/installation and follow the recommended installation instructions. For MacOS/Linux/WSL this should just be a command line instruction to run, on Windows there’s an installer. All the details are on that page though.

Once it’s installed check the version:

```
$ sugar --version
sugar-cli 1.1.0
```

We downloaded everything we need. Lets Build

**Starter code**

I’ve included some starter code for the frontend app. You can use this repo. 

git clone https://github.com/gecginserhat/Solana_NFT

cd solana-nft-tutorial-starter

If you browse to the `ui` directory you should see a pretty standard NextJS app. You can run it:

```
# install dependencies
$ npm install

# run the app
$ npm run dev
```

Browse to [localhost:3000](http://localhost:3000/) and you should see the app running with a pretty empty looking page:


![3](https://github.com/gecginserhat/Solana_NFT/assets/74310970/b828da53-4aa0-45f5-ab78-4e3e3c8c78cd)

**Creating the assets**

This is where we’re going to put the assets for our NFT collection. For each NFT in our collection we will need eg. an image `0.png` and metadata `0.json`. JPG and GIF file types are also supported for the image. Note that the naming must be numeric like this, and there can’t be any gaps starting from 0.

So our first job is to get images for our NFT collection. You can use the images you want on the internet by adding them yourself. Please select 3-5 pictures

Save your images in the `assets` directory, so you should have something like:

```
assets/
    0.jpg
    1.jpg
    2.jpg
    3.jpg
    4.jpg
```

Okay next let’s take a look at the metadata. This is where we’re going to put all the details about each of our NFTs: name, description, etc. You can also add custom attributes - so you can basically record anything you like about each NFT!

The standard for NFT metadata on Metaplex is here: https://docs.metaplex.com/programs/token-metadata/token-standard#the-non-fungible-standard - but it’s probably easiest to go through an example.

Here’s `0.json`:

```
{
  "name": "Dino 0",
  "description": "Massive dinosaur on water!",
  "image": "0.jpg",
  "symbol": "DINO",
  "attributes": [
    {
      "trait_type": "arms",
      "value": "short"
    }
  ],
  "properties": {
    "files": [
      {
        "uri": "0.jpg",
        "type": "image/jpg"
      }
    ]
  }
}
```

- Each NFT needs a name, this should be unique. I’m just including the index, which seems quite common. You can use anything though!
- Each NFT has a description. No restrictions here really either
- The image refers to a filename in our `assets` directory
- Symbol should be the same for each NFT in your collection
- Attributes are optional, but we’ll use one here. They consist of a `trait_type` which is the name of the attribute, and a `value` which is its value. So in this case the first dinosaur has short arms! You don’t have to use the same attributes across your collection, but marketplaces tend to let users filter on them and wallets will display them, so they’re a handy way to group your NFTs. We’ll look at how to display them in our app later too.
- Properties just lists the associated files, which in our case is just the image. Make sure you update `type` if you’re using a different image format.

Okay now you should have something like this:

```
assets/
    0.jpg
    0.json
    1.jpg
    1.json
    2.jpg
    2.json
    3.jpg
    3.json
    4.jpg
    4.json
```

Finally, we’re doing to add a `collection` image and JSON file. This will describe the NFT collection itself, and is actually stored as an NFT too on-chain. Metaplex will mark our NFTs as being part of this verified collection.

Select a nice image to represent your collection as a whole. Here’s mine:


![4](https://github.com/gecginserhat/Solana_NFT/assets/74310970/c8079138-bd2f-4897-8e99-cdf5f63f33f0)

Save it as `collection.jpg` (or other image type)

Our `collection.json` will look exactly the same as our others, except I’ll skip the attribute here. It just describes the collection as a whole:

```
{
  "name": "Dinos collection",
  "description": "A collection of dinosaurs living on the blockchain",
  "image": "collection.jpg",
  "symbol": "DINO",
  "attributes": [],
  "properties": {
    "files": [
      {
        "uri": "collection.jpg",
        "type": "image/jpg"
      }
    ]
  }
}
```

Now you should have something like this:

```
assets/
    0.jpg
    0.json
    1.jpg
    1.json
    2.jpg
    2.json
    3.jpg
    3.json
    4.jpg
    4.json
    collection.jpg
    collection.json
```

**Creating the candy machine**

Make sure you’re in the `nft-collection` directory that contains `assets`.

First we’ll run `sugar create-config` which will walk us through creating a configuration file describing the settings for our NFT collection. It’ll ask us a number of questions as it runs.
![5](https://github.com/gecginserhat/Solana_NFT/assets/74310970/7e36b30c-7657-4f22-aeb7-d6eee531ca9e)


Users will pay SOL to mint an NFT from our collection. You can choose any price, but use something quite small for this just to avoid having to use too much devnet SOL when testing. Decimals are fine, and you can also use 0 for a free mint.

```
? Found 5 file pairs in "assets". Is this how many NFTs you will have in your candy machine? (y/n) ›
```

It’ll read the file pairs from `assets`. I have 5 NFTs, and then also the collection - so 5 NFTs is correct. Type `y` to continue.

```
? Found symbol "DINO" in your metadata file. Is this value correct? (y/n) ›
```

It’s read the symbol from my `json` files. Again `y` to continue

```
? What is the seller fee basis points? ›
```

This is how we set royalties on secondary sales. So if someone sells our NFT on a marketplace, we can get a cut of that sale. This is in basis points, which basically means it’s in multiples of `0.01%`. You can enter 100 to get 1% royalties, or 250 to get 2.5% for example. We’re not going to be listing our NFTs on a marketplace in this tutorial, so you can enter whatever you like.

```
? What is your go live date? Many common formats are supported. If unsure, try YYYY-MM-DD HH:MM:SS [+/-]UTC-OFFSET or type 'now' for current time. For example 2022-05-02 18:00:00 +0000 for May 2, 2022 18:00:00 UTC. ›
```

We can deploy our candy machine with a future mint date, so that nobody can mint the NFTs until that time. For this tutorial just use `now` so that the candy machine will be live immediately.

```
? How many creator wallets do you have? (max limit of 4) ›
```

We can split the secondary sale royalties between up to 4 different creators automatically. For this tutorial just use `1` - you’re the only creator!

```
? Enter creator wallet address #1 ›
```

Put your Solana address here, the one you have configured locally. If you need it, open a new terminal and run `solana address`

```
? Enter royalty percentage share for creator #1 (e.g., 70). Total shares must add to 100. ›
```

If you have multiple creators you can decide how the royalties get divided up. In our case, we’re the only creator so we need to enter `100`

```
? Which extra features do you want to use? (use [SPACEBAR] to select options you want and hit [ENTER] when done) ›
✔ SPL Token Mint
✔ Gatekeeper
✔ Whitelist Mint
✔ End Settings
✔ Hidden Settings
```

Candy machine has a bunch of extra features that you can use, like minting with an SPL token, requiring a whitelist, or having an end date for the mint. You can enable and configure them in this step. For this tutorial, we don’t need any of them so I’d suggest leaving them for now. Just hit enter to continue.

```
? What is your SOL treasury address? ›
```

This is where the mint proceeds are paid. For this tutorial, we can just use our own Solana address again. You can just copy it from where we entered it a few questions earlier.

```
? What upload method do you want to use? ›
❯ Bundlr
  AWS
  NFT Storage
  SHDW
```

Because storing data on-chain is expensive, our actual image files and the JSON metadata file will be stored off-chain, with the on-chain data linking to them. Metaplex supports a number of ways to store this data. Bundlr is the simplest and will store our data on Arweave, which is a blockchain designed for permanent data storage. Hit enter to select it. Note that this won’t perform any uploads yet, we’re just creating our config for now.

```
? Do you want to retain update authority on your NFTs? We HIGHLY recommend you choose yes. (y/n) ›
```

You can use the update authority to update NFTs later. For some use cases this is required, like updating attributes in a game. Metaplex recommend retaining this update authority, so we’ll do that here. Hit `y` to answer yes and continue

```
? Do you want your NFTs to remain mutable? We HIGHLY recommend you choose yes. (y/n) ›
```

Similarly, if your NFTs are mutable then they can be updated in the future. Again Metaplex recommend we do this, so hit `y` to answer yes and continue


![6](https://github.com/gecginserhat/Solana_NFT/assets/74310970/3c1d4c94-7273-443a-8ddf-7dbe459497d1)

This will have created a new file `nft-collection/config.json` with all our responses. It’s a lot easier than writing that file ourselves! There’s also a `sugar.log` file that is just for debugging, you shouldn’t need to worry about that.

Now let’s upload our assets to Bundlr by running `sugar upload`:


![7](https://github.com/gecginserhat/Solana_NFT/assets/74310970/197a7d37-24e2-4631-a551-4ccab19987dc)

It uploads all 6 asset pairs, the 5 NFTs, and the collection one. We’ve paid for this with devnet SOL, it cost 0.00024467 to upload all our assets to Arweave.

This will have created a file called `cache.json` which has links to all your uploaded assets. Here’s a snippet from mine:

```
"-1": {
      "name": "Dinos collection",
      "image_hash": "2502df89e57c343deb1a75ea74c6d2e3a954c0eb42404bced16db5dbf2f409c7",
      "image_link": "https://arweave.net/v8ijxvx5Xht2JaUcLek2i3qcyKKa16n_I88Pis3hIq0",
      "metadata_hash": "8a42d488b401aa8b8757d2fb54819eae25c169ef9bda81b007172cd04184155e",
      "metadata_link": "https://arweave.net/e7OIORoO1guxRsloUC-xEIv3q0np6L5-rdnHqdcL5Yc",
      "onChain": false
    },
```

Those Arweave links have my uploaded `collection.jpg` and `collection.json`! These are the links that the candy machine will point the NFTs at.

Now let’s deploy the candy machine with `sugar deploy`


![8](https://github.com/gecginserhat/Solana_NFT/assets/74310970/98fe0228-b1cd-41cc-bbbc-e8218b880726)

Finally, let’s just check everything looks good with `sugar verify`


![9](https://github.com/gecginserhat/Solana_NFT/assets/74310970/90ff066a-1954-4098-be26-2cc9f43cfb61)


You can use that solaneyes link to view the NFTs in your collection and their metadata.

**Mint Page**

Now that you’ve deployed your NFT collection we’re going to want to make it available to mint. Metaplex provides a great SDK to get this working.

At a high level, we’re going to allow users to connect a Solana wallet to our app, and then we’re going to let them mint an NFT to that wallet.

## Phantom Wallet Setup

We connect a Solana wallet to web apps by using a browser wallet. This is how we connect to and interact with apps. If you already have a Solana wallet set up then feel free to speed through this!

The most popular browser wallet is called Phantom, and you can download it from their website here: https://phantom.app/download. They have browser extensions for Chrome, Brave, Firefox and Edge - so you’re probably covered! If you’d like to use a different wallet for any reason then feel free to, they’re all compatible.

When Phantom first launches you’ll have the option to create a new wallet or import an existing one:

![10](https://github.com/gecginserhat/Solana_NFT/assets/74310970/ff390cec-410a-462a-ad82-2b18867bf6b7)



Click “Create a new wallet” if this is your first time with a browser wallet. We won’t use the same account that we used to create the NFTs, we can use a different one to mint them. You’ll be asked to create a password, this is for the Phantom extension only.

You’ll then be presented with your Secret Recovery Phrase. We got this for the CLI wallet we set up earlier too. Your Secret Recovery Phrase will be 12 random words. This can be used to access all wallets you create in Phantom, for example, to import them into another wallet. Make sure to never share this with anyone, they’ll be able to access all your wallets. Save it somewhere secure, you’ll need it when you want to import your wallets elsewhere!

Once you’re done with the setup you’ll be able to see your wallet in Phantom:

Before we do anything else let’s switch to Devnet. In Phantom, go to Settings (gear icon in the bottom right), choose “Change Network” and select “Devnet”.


![11](https://github.com/gecginserhat/Solana_NFT/assets/74310970/01ce835d-2cbb-4bf7-a6d8-e3222a99124a)

**Get some Devnet SOL**


![13](https://github.com/gecginserhat/Solana_NFT/assets/74310970/b45ec8f0-6656-4d6a-9ae5-5192525880a1)

Now jump back into your terminal and airdrop to that address. Here’s how it looks:

```
$ solana airdrop 2 HHhBxpZiPMRGgwoWoCRQALGJw1FfDS4Sv98EennQ5idq
Requesting airdrop of 2 SOL

Signature: 3swmtvGTRkRQ9R6fXDNxEY26eprGEm7oaQfJaRbUp9XL7K5Dkw3asqzaQPkWvteJENGEwLLJ2okdc3hFwXdtSTFZ

2 SOL
```

**Connecting the wallet to our app**

Let’s jump back into the `ui` directory (it’s at the same level as `nft-collection`). This is where we’re going to build out a mint UI for our NFT collection.

If you haven’t already, install the dependencies (`npm install`) and run the app (`npm run dev`). It’ll be running on [localhost:3000](http://localhost:3000/)


![14](https://github.com/gecginserhat/Solana_NFT/assets/74310970/d5b937b6-0383-40db-be9b-56c5e02df278)

It’s a pretty straightforward app!

The first thing that we need to do is let users connect their Solana wallet to our app. They’ll use their connected wallet to mint our NFT.

Let’s start by installing some dependencies. These are maintained by Solana and cover connecting our app to the network and providing a nice interface to connect a user’s wallet

`npm install @solana/web3.js @solana/wallet-adapter-react @solana/wallet-adapter-react-ui @solana/wallet-adapter-base @solana/wallet-adapter-wallets`

Next, we need to add code to handle the Solana connection and connected wallet at the top of our component hierarchy, which in Next means in `pages/_app.tsx`. Update it to look like this:

```
import Head from 'next/head'
import { AppProps } from 'next/app'
import { useMemo } from 'react';
import Layout from '../components/Layout';
import { ConnectionProvider, WalletProvider } from '@solana/wallet-adapter-react';
import { WalletAdapterNetwork } from '@solana/wallet-adapter-base';
import { PhantomWalletAdapter, SolflareWalletAdapter } from '@solana/wallet-adapter-wallets';
import { WalletModalProvider } from '@solana/wallet-adapter-react-ui';
import { clusterApiUrl } from '@solana/web3.js';

// Default styles that can be overridden by your app
require('@solana/wallet-adapter-react-ui/styles.css');

import '../styles/index.css'

function MyApp({ Component, pageProps }: AppProps) {
  // The network can be set to 'devnet', 'testnet', or 'mainnet-beta'.
  const network = WalletAdapterNetwork.Devnet;

  // You can also provide a custom RPC endpoint.
  const endpoint = useMemo(() => clusterApiUrl(network), [network]);

  const wallets = useMemo(() => [
    new PhantomWalletAdapter(),
    new SolflareWalletAdapter(),
  ], []);

  return (
    <>
      <Head>
        <title>Dinosaurs 'r' Us</title>
        <meta name="viewport" content="initial-scale=1.0, width=device-width" />
      </Head>
      <ConnectionProvider endpoint={endpoint}>
        <WalletProvider wallets={wallets} autoConnect>
          <WalletModalProvider>
            <Layout>
              <Component {...pageProps} />
            </Layout>
          </WalletModalProvider>
        </WalletProvider>
      </ConnectionProvider>
    </>
  )
}

export default MyApp
```

There’s a lot new here! We’re creating a connection to the devnet Solana network. To change the network or endpoint we’ll just need to update it here. We’re also defining the wallets that we want to allow to connect to our app. Here I’ve used Phantom and Solflare, but there are adapters for loads of wallets so feel free to add any others you like! Once we’ve defined the endpoint and wallets, we wrap our app in some context providers so that we have access to the Solana connection and any connected wallet from every page in our app. This code is pretty much the same in any app using these Solana libraries.

Okay so now we can add a wallet connect button anywhere in our app. We’ll add it to the navbar. Update `components/Navbar.tsx`:

```
import { WalletMultiButton } from "@solana/wallet-adapter-react-ui";
import Link from "next/link";

export default function Navbar() {
  return (
    <nav className="flex w-full h-10 gap-4 px-4 pt-4 pb-20 font-sans text-white md:px-20 md:gap-10">
      <Link href="/">
        <a className="text-2xl no-underline hover:text-slate-300">
          Mint
        </a>
      </Link>
      <Link href="/holders">
        <a className="text-2xl no-underline text-grey-darkest hover:text-slate-300 grow">
          Holders only 😎
        </a>
      </Link>
      <WalletMultiButton />
    </nav>
  )
}
```

All we need to do is add `<WalletMultiButton />` from the `wallet-adapter-react-ui` library. The app should look like this now:


![15](https://github.com/gecginserhat/Solana_NFT/assets/74310970/ecefeacd-5ee1-4462-92db-0ed9264252c1)

And if you click that Select Wallet button you should get a modal to choose your wallet, and then connect:


![16](https://github.com/gecginserhat/Solana_NFT/assets/74310970/0da40e65-b47e-4dd7-af4e-d5248a35c6fd)

And now users can connect to our app!

## Fetching the candy machine

One more thing before we get to our mint function, we need to add code to fetch our candy machine. It’s a Solana account and holds a bunch of state, like the symbol (DINO in my case), and the number of NFTs minted and remaining.

We need to fetch it to pass to the mint function, but we’ll also use it to make our UI a bit more useful!

First, we have one more dependency, the Metaplex JS library:

```
npm install @metaplex-foundation/js
```

Next, let’s create a new directory `lib` and a file `lib/constants.ts`. We’re going to put our **Candy machine address** and our **Collection mint address** there.

These are the addresses that were output by `sugar deploy`:

```
$ sugar deploy
[1/3] 🍬 Creating candy machine
Candy machine ID: 6U2uTMVvv4jPXEwFVo2yCamLfKFwk4jcfHoe9PuyeEys

[2/3] 📦 Creating and setting the collection NFT for candy machine
Collection mint ID: GDuJigbNH878Yrpoqau9CT9MsH42eBhyXTtrPCAY6AJh
```

We only need the candy machine one for now, but let’s save both while we know where they are!

```
import { PublicKey } from "@solana/web3.js";

export const CANDY_MACHINE_ADDRESS = new PublicKey("paste-yours-here")
export const COLLECTION_MINT_ADDRESS = new PublicKey("paste-yours-here")
```

Make sure you use your own addresses!

Now let’s update our home page to fetch the candy machine on load. I’ll show the updated code and then talk through what’s new and what’s going on

```
import { CandyMachineV2, Metaplex } from "@metaplex-foundation/js";
import { useConnection } from "@solana/wallet-adapter-react";
import { useEffect, useState } from "react";
import PageHeading from "../components/PageHeading";
import { CANDY_MACHINE_ADDRESS } from "../lib/constants";

export default function Home() {
  const { connection } = useConnection();
  const [candyMachine, setCandyMachine] = useState<CandyMachineV2 | undefined>(undefined)

  const metaplex = Metaplex
    .make(connection)

  const candyMachines = metaplex.candyMachinesV2()

  async function fetchCandyMachine() {
    const fetched = await candyMachines
      .findByAddress({ address: CANDY_MACHINE_ADDRESS })

    console.log("Fetched candy machine!", fetched)
    setCandyMachine(fetched)
  }

  useEffect(() => {
    fetchCandyMachine()
  }, [])

  return (
    <main className="flex flex-col gap-8">
      <PageHeading>Dinosaurs 'r' Us</PageHeading>

      <div className="basis-1/4">
        <button
          type="button"
          className="inline-flex items-center px-4 py-2 text-sm font-medium text-indigo-700 bg-indigo-100 border border-transparent rounded-md cursor-pointer hover:bg-indigo-200 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed"
        >
          Mint a DINO! 🦖
        </button>
      </div>

      <hr />
    </main>
  )
}
```

Okay let’s step through it

```
const { connection } = useConnection();
```

First, we have a React hook to get the Solana connection

```
const [candyMachine, setCandyMachine] = useState<CandyMachineV2 | undefined>(undefined)
```

Just some React state to store the candy machine once we fetch it. It starts undefined. Note that our NFT collection is using Candy Machine V2, but `sugar` is taking care of that for us!

```
  const metaplex = Metaplex
    .make(connection)
```

This is the entry point to Metaplex on the client. We’re just creating a new instance and passing it the Solana connection.

```
  const candyMachines = metaplex.candyMachinesV2()
```

We’re only going to use the candy machines module of Metaplex on this page. There are also other modules, we’ll see one of them later.

```
  async function fetchCandyMachine() {
    const fetched = await candyMachines
      .findByAddress({ address: CANDY_MACHINE_ADDRESS })

    console.log("Fetched candy machine!", fetched)
    setCandyMachine(fetched)
  }
```

Here we’re using the `findByAddress` module of the candy machines module to fetch our candy machine. We pass our address in as input. When we’ve fetched it we log it and set that `candyMachine` state to it.

```
  useEffect(() => {
    fetchCandyMachine()
  }, [])
```

Just a React `useEffect` hook - when the page loads it’ll call that `fetchCandyMachine` function once.

Now if you refresh the page you should see the fetched candy machine logged:

<img width="1000" alt="17" src="https://github.com/gecginserhat/Solana_NFT/assets/74310970/a3e61b0a-a1b1-4fd8-9577-5f85a18e506f">


At this point, we have access to that whole object in our UI. There are a few fields that look like they’d be useful to display: the symbol, itemsMinted, and itemsAvailable for example.

Let’s update the `return` to include some information from the candy machine:

```
const canMint = candyMachine && candyMachine.itemsRemaining.toNumber() > 0

return (
  <main className="flex flex-col gap-8">
    <PageHeading>Dinosaurs 'r' Us</PageHeading>

    <div className="basis-1/4">
      <button
        type="button"
        className="inline-flex items-center px-4 py-2 text-sm font-medium text-indigo-700 bg-indigo-100 border border-transparent rounded-md cursor-pointer hover:bg-indigo-200 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed"
        disabled={!canMint}
      >
        Mint 1 {candyMachine ? candyMachine.symbol : "NFT"}! 🦖
      </button>
    </div>

    {candyMachine ? (
      <p className="text-white">
        {candyMachine.itemsMinted.toNumber()} / {candyMachine.itemsAvailable.toNumber()} minted!
      </p>
    ) : <p className="text-white">Loading...</p>
    }

    <hr />
  </main>
)
```

![18](https://github.com/gecginserhat/Solana_NFT/assets/74310970/166915b2-3df3-4636-93e9-35bb216a5955)


**Mint button**

The `findByAddress` function we called above is not creating a transaction, it’s just reading data from the blockchain. By contrast, our `mint` button will be creating a transaction since it’s updating data on the blockchain. The user is sending SOL and receiving an NFT. Ownership of NFTs is tracked on-chain.

This transaction will be paid for by the connected wallet. They will send the SOL and receive the NFT, and they will pay the (very small) transaction fee. So we need to update our Metaplex instance to add an **identity.** This identity will be the default payer, and the default to receive our NFTs. It’ll also be used to submit the transaction.

Specifically, we’re going to use the `walletAdapterIdentity` from Metaplex. This means that the connected wallet from `wallet-adapter` is used as the identity. Here’s how that looks:

```
import { CandyMachineV2, Metaplex, walletAdapterIdentity } from "@metaplex-foundation/js";
import { useConnection, useWallet } from "@solana/wallet-adapter-react";
import { useEffect, useState } from "react";
import PageHeading from "../components/PageHeading";
import { CANDY_MACHINE_ADDRESS } from "../lib/constants";

export default function Home() {
  const { connection } = useConnection();
  const wallet = useWallet();
  const [candyMachine, setCandyMachine] = useState<CandyMachineV2 | undefined>(undefined)

  const metaplex = Metaplex
    .make(connection)
    .use(walletAdapterIdentity(wallet))

  const candyMachines = metaplex.candyMachinesV2()

  ...
```

The `useWallet` hook gives us the state of the connected wallet. We can just pass this straight into the Metaplex SDK. This is a really nice abstraction - that connected wallet will now be used for everything in the Metaplex functions by default.

Next, let’s add our mint function:

```
import { CandyMachineV2, Metaplex, walletAdapterIdentity } from "@metaplex-foundation/js";
import { useConnection, useWallet } from "@solana/wallet-adapter-react";
import { useEffect, useState } from "react";
import PageHeading from "../components/PageHeading";
import { CANDY_MACHINE_ADDRESS } from "../lib/constants";

export default function Home() {
  const { connection } = useConnection();
  const wallet = useWallet();
  const [candyMachine, setCandyMachine] = useState<CandyMachineV2 | undefined>(undefined)
  const [isMinting, setIsMinting] = useState(false)

  const metaplex = Metaplex
    .make(connection)
    .use(walletAdapterIdentity(wallet))

  const candyMachines = metaplex.candyMachinesV2()

  async function fetchCandyMachine() {
    const fetched = await candyMachines
      .findByAddress({ address: CANDY_MACHINE_ADDRESS })

    console.log("Fetched candy machine!", fetched)
    setCandyMachine(fetched)
  }

  useEffect(() => {
    fetchCandyMachine()
  }, [])

  async function mintOne() {
    setIsMinting(true);

    const mintOutput = await candyMachines
      .mint({ candyMachine });

    setIsMinting(false);
    console.log("Minted one!", mintOutput)

    // Fetch the candy machine to update the counts
    await fetchCandyMachine()
  }

  const canMint =
    candyMachine &&
    candyMachine.itemsRemaining.toNumber() > 0 &&
    wallet.publicKey &&
    !isMinting

  return (
    <main className="flex flex-col gap-8">
      <PageHeading>Dinosaurs 'r' Us</PageHeading>

      <div className="basis-1/4">
        <button
          type="button"
          className="inline-flex items-center px-4 py-2 text-sm font-medium text-indigo-700 bg-indigo-100 border border-transparent rounded-md cursor-pointer hover:bg-indigo-200 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed"
          disabled={!canMint}
          onClick={mintOne}
        >
          Mint 1 {candyMachine ? candyMachine.symbol : "NFT"}! <span className={isMinting ? 'animate-spin' : 'animate-none'}>🦖</span>
        </button>
      </div>

      {candyMachine ? (
        <p className="text-white">
          {candyMachine.itemsMinted.toNumber()} / {candyMachine.itemsAvailable.toNumber()} minted!
        </p>
      ) : <p className="text-white">Loading...</p>
      }

      <hr />
    </main>
  )
}
```

Okay so if you refresh the page now you should be able to click on your mint button and see a wallet transaction:

![19](https://github.com/gecginserhat/Solana_NFT/assets/74310970/751978b8-b028-4c4a-8fd5-9fb68eb1e800)


If you approve it then the UI should quickly update to show an NFT has been minted:

![20](https://github.com/gecginserhat/Solana_NFT/assets/74310970/25ad956a-9de3-4711-9d7c-569e193fe67a)


And in the console you should see that minted NFT:

![21](https://github.com/gecginserhat/Solana_NFT/assets/74310970/b24c1fc9-abab-45c7-8403-84d705a178dc)


Just open your wallet and look for the NFTs, in Phantom it’s the second tab. You should be able to see the one you just minted!

![22](https://github.com/gecginserhat/Solana_NFT/assets/74310970/0ac9b8b0-bc3c-4130-98f1-b9271a374655)


Most wallets will also give a way to view the wallet in a blockchain explorer, which lets you see exactly what’s stored on-chain for that NFT. In Phantom, there’s an option to open in Solscan. If you change your preferred explorer in settings it’ll use that one instead - I like Solana Explorer best for NFTs because it shows a bit more information.

![23](https://github.com/gecginserhat/Solana_NFT/assets/74310970/fd5895f4-7edb-43a8-8820-e307155ab4c5)


**If you mint all your NFTs**

You might find you mint all the NFTs while testing stuff, and then you won’t be able to mint anymore. If this happens the easiest thing to do is just deploy a new collection

In `nft-collection` delete `cache.json` (don’t delete `config.json`!) and then run `sugar upload`, `sugar deploy` and `sugar verify` again. Or just run `sugar launch` which runs all of those sequentially.

Remember to update the candy machine and collection IDs in `ui/lib/constants.ts`

## Displaying the minted NFT

If you take a closer look at that console log message after an NFT is minted, you’ll see that it includes an `nft` field which has all the details about the specific NFT we minted:

<img width="1000" alt="24" src="https://github.com/gecginserhat/Solana_NFT/assets/74310970/7a094327-6506-43e5-8630-a5bafb02cff3">


It’d be pretty cool if our app showed them the NFT they minted right away, so they don’t need to go to their wallet to look at it. Let’s add that!

First, we’ll add a component to display an NFT. We’ll re-use this later on our holder page so it’s nice to make it its own component. Add a new file `components/NftDisplay.tsx` with this content:

```
import { JsonMetadata } from "@metaplex-foundation/js"

interface NftProps {
  json: JsonMetadata<string>
}

export default function NftDisplay({ json }: NftProps) {
  const arms = json.attributes?.find(a => a.trait_type === "arms").value

  return (
    <div className="flex flex-col gap-4">
      <h3 className="text-lg font-medium text-white">{json.name}</h3>
      <img src={json.image} alt={json.name} />
      <p className="text-sm text-white"><span className="font-bold">Arms: </span>{arms}</p>
    </div>
  )
}
```

This is a pretty simple component, it just shows the NFT’s name, its image, and a little caption including our `arms` attribute value. Feel free to change it up using any of the data available in `json`!

And then let’s display that on the page after minting:

```
import { CandyMachineV2, Metaplex, NftWithToken, walletAdapterIdentity } from "@metaplex-foundation/js";
import { useConnection, useWallet } from "@solana/wallet-adapter-react";
import { useEffect, useState } from "react";
import NftDisplay from "../components/NftDisplay";
import PageHeading from "../components/PageHeading";
import { CANDY_MACHINE_ADDRESS } from "../lib/constants";

export default function Home() {
  const { connection } = useConnection();
  const wallet = useWallet();
  const [candyMachine, setCandyMachine] = useState<CandyMachineV2 | undefined>(undefined)
  const [isMinting, setIsMinting] = useState(false)
  const [mintedNft, setMintedNft] = useState<NftWithToken | undefined>(undefined)

  const metaplex = Metaplex
    .make(connection)
    .use(walletAdapterIdentity(wallet))

  const candyMachines = metaplex.candyMachinesV2()

  async function fetchCandyMachine() {
    const fetched = await candyMachines
      .findByAddress({ address: CANDY_MACHINE_ADDRESS })

    console.log("Fetched candy machine!", fetched)
    setCandyMachine(fetched)
  }

  useEffect(() => {
    fetchCandyMachine()
  }, [])

  async function mintOne() {
    setIsMinting(true);

    const mintOutput = await candyMachines
      .mint({ candyMachine });

    setIsMinting(false);
    console.log("Minted one!", mintOutput)
    setMintedNft(mintOutput.nft)

    // Fetch the candy machine to update the counts
    await fetchCandyMachine()
  }

  const canMint =
    candyMachine &&
    candyMachine.itemsRemaining.toNumber() > 0 &&
    wallet.publicKey &&
    !isMinting

  return (
    <main className="flex flex-col gap-8">
      <PageHeading>Dinosaurs 'r' Us</PageHeading>

      <div className="basis-1/4">
        <button
          type="button"
          className="inline-flex items-center px-4 py-2 text-sm font-medium text-indigo-700 bg-indigo-100 border border-transparent rounded-md cursor-pointer hover:bg-indigo-200 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed"
          disabled={!canMint}
          onClick={mintOne}
        >
          Mint 1 {candyMachine ? candyMachine.symbol : "NFT"}! <span className={isMinting ? 'animate-spin' : 'animate-none'}>🦖</span>
        </button>
      </div>

      {candyMachine ? (
        <p className="text-white">
          {candyMachine.itemsMinted.toNumber()} / {candyMachine.itemsAvailable.toNumber()} minted!
        </p>
      ) : <p className="text-white">Loading...</p>
      }

      {mintedNft ?
        <div className="w-96">
          <h2 className="text-xl font-medium text-white">You minted a {candyMachine.symbol}! 🎉</h2>
          <NftDisplay json={mintedNft.json} />
        </div> : null
      }

      <hr />
    </main>
  )
}
```

Hopefully, this is quite straightforward! We have new state `mintedNft`, and after minting we set it to `mintOutput.nft`, which is of type `NftWithToken`. When it’s set we just display the NFT using that `NftDisplay` component. Here’s how it looks:


![25](https://github.com/gecginserhat/Solana_NFT/assets/74310970/5a918b89-d3bc-4126-aa93-886b74368fe6)

And of course you can double check it’s the same one in your wallet.

**Fetching NFTs the user minted**

Let’s update `pages/holders.tsx` to fetch NFTs for the connected user:

```
import { Metaplex } from "@metaplex-foundation/js"
import { useConnection, useWallet } from "@solana/wallet-adapter-react"
import { useEffect } from "react"
import { COLLECTION_MINT_ADDRESS } from "../lib/constants"

export default function Holders() {
  const { connection } = useConnection()
  const wallet = useWallet()

  const metaplex = Metaplex
    .make(connection)

  const nfts = metaplex.nfts()

  async function getUserNfts() {
    if (!wallet.publicKey) return

    // Fetch all the user's NFTs
    const userNfts = await nfts
      .findAllByOwner({ owner: wallet.publicKey })

    // Filter to our collection
    const ourCollectionNfts = userNfts.filter(
      metadata =>
        metadata.collection !== null &&
        metadata.collection.verified &&
        metadata.collection.address.toBase58() === COLLECTION_MINT_ADDRESS.toBase58()
    )

    console.log("Got their NFTs!", ourCollectionNfts)
  }

  useEffect(() => {
    getUserNfts()
  }, [wallet.publicKey])

  return (
    <p className="text-lg text-white">Holders only! ☠️</p>
  )
}
```

Let's step through this

```
  const { connection } = useConnection()
  const wallet = useWallet()

  const metaplex = Metaplex
    .make(connection)

  const nfts = metaplex.nfts()
```

This should look familiar from our mint page. This time we’re using the nfts module of Metaplex. Also we don’t need a Metaplex identity here, we’re just reading blockchain data.

```
  async function getUserNfts() {
    if (!wallet.publicKey) return

    // Fetch all the user's NFTs
    const userNfts = await nfts
      .findAllByOwner({ owner: wallet.publicKey })

    // Filter to our collection
    const ourCollectionNfts = userNfts.filter(
      metadata =>
        metadata.collection !== null &&
        metadata.collection.verified &&
        metadata.collection.address.toBase58() === COLLECTION_MINT_ADDRESS.toBase58()
    )

    console.log("Got their NFTs!", ourCollectionNfts)
  }
```

We’re using the `findAllByOwner` function, which takes the owner public key as input and returns a list of objects describing the metadata of owned NFTs. Unfortunately the filtering here is quite limited: we’re going to get all their owned NFTs from all collections. Remember this stuff is all on chain, we don’t have a database specific to our collection.

So to filter to our collection we can use the **collection** field. Remember how we added that `collection.jpg` and `collection.json` when we were creating our assets? That’s a verified collection that all our NFTs are going to be linked to. We can see this on the explorer page for any of our NFTs:

<img width="1000" alt="26" src="https://github.com/gecginserhat/Solana_NFT/assets/74310970/9be02022-7640-4c9d-b1e7-6a5b3219e6e6">


This is our collection image as an NFT! The **Verified Collection Address** field shows that the NFT in our wallet is part of our collection. Also note that this Address should match the **collection mint ID** that sugar output for you. Super important: not the candy machine ID! We put this in `lib/constants.ts` earlier. Now we can compare it to the collection stored for each NFT to see if they’re a part of our collection.

That’s what we’re doing here:

```
    // Filter to our collection
    const ourCollectionNfts = userNfts.filter(
      metadata =>
        metadata.collection !== null &&
        metadata.collection.verified &&
        metadata.collection.address.toBase58() === COLLECTION_MINT_ADDRESS.toBase58()
    )
```

And once we’ve fetched and filtered the NFTs we just log them for now.

```
  useEffect(() => {
    getUserNfts()
  }, [wallet.publicKey])
```

This is just a `useEffect` hook to get the user NFTs. It’ll re-run whenever the connected wallet changes to make sure the page always stays up to date.

Okay so now if we load our `/holders` page and connect a wallet that has minted some of our NFTs we should see some data logged:

<img width="1000" alt="27" src="https://github.com/gecginserhat/Solana_NFT/assets/74310970/45a70a4b-fe6d-47bc-939c-0b6c36f206db">


It looks a little bit like the response we got back from our mint, with one quite big difference: `json` is null, and `jsonLoaded` is false. Remember that we’re using that `json` field to display NFTs, without it we don’t have the attributes or the image!

Now if you check out the `uri` field it should be an arweave URL like https://arweave.net/JkIux3AM6k0xMjZ_qySYu4lJoq97g5qIB8LBySdmNew. If you visit that it should have JSON like:

```
{
  "name": "Dino 4",
  "symbol": "DINO",
  "description": "The gang of swimmers",
  "image": "https://arweave.net/Qd-5yMVOqgOptq_FrQL0fNVoubMVwwoPtdKC3RRs9sA",
  "attributes": [
    {
      "trait_type": "arms",
      "value": "more like fins"
    }
  ],
  "properties": {
    "files": [
      {
        "uri": "https://arweave.net/Qd-5yMVOqgOptq_FrQL0fNVoubMVwwoPtdKC3RRs9sA",
        "type": "image/jpg"
      }
    ]
  }
}
```

Okay, so that’s the JSON we want! So what’s going on here is that the URL is stored on-chain, but the JSON itself isn’t. That’s nice because it means we can have any number of attributes etc. without worrying about storing all that data on chain.

The Metaplex `findAllByOwner` doesn’t automatically go and fetch this JSON for us, while the mint function did. That’s because `findAllByOwner` could return loads of NFTs, remember this is before filtering by our collection. So someone could have hundreds of NFTs, we don’t want to go and fetch all their JSON data just to throw out 99% of them that aren’t in our collection.

Fortunately we don’t need to go and fetch the JSON data ourselves, Metaplex has a function to do it for us. Let’s update our function to load the NFTs after we’ve filtered to our collection:

```
import { Metadata, Metaplex } from "@metaplex-foundation/js"
import { useConnection, useWallet } from "@solana/wallet-adapter-react"
import { useEffect } from "react"import { Metadata, Metaplex } from "@metaplex-foundation/js"
import { useConnection, useWallet } from "@solana/wallet-adapter-react"
import { useEffect } from "react"
import { COLLECTION_MINT_ADDRESS } from "../lib/constants"

export default function Holders() {
  const { connection } = useConnection()
  const wallet = useWallet()

  const metaplex = Metaplex
    .make(connection)

  const nfts = metaplex.nfts()

  async function getUserNfts() {
    if (!wallet.publicKey) return

    // Fetch all the user's NFTs
    const userNfts = await nfts
      .findAllByOwner({ owner: wallet.publicKey })

    // Filter to our collection
    const ourCollectionNfts = userNfts.filter(
      metadata =>
        metadata.collection !== null &&
        metadata.collection.verified &&
        metadata.collection.address.toBase58() === COLLECTION_MINT_ADDRESS.toBase58()
    )

    // Load the JSON for each NFT
    const loadedNfts = await Promise.all(ourCollectionNfts
      .map(metadata => {
        return nfts
          .load({ metadata: metadata as Metadata })
      })
    )

    console.log("Got their NFTs!", loadedNfts)
  }

  useEffect(() => {
    getUserNfts()
  }, [wallet.publicKey])

  return (
    <p className="text-lg text-white">Holders only! ☠️</p>
  )
}

We can pass a single NFT to nfts.load() to have its JSON loaded. Each of these is an independent task, so we use Promise.all to load all of them. Now if we refresh the page we should see the JSON loaded:
import { COLLECTION_MINT_ADDRESS } from "../lib/constants"

export default function Holders() {
  const { connection } = useConnection()
  const wallet = useWallet()

  const metaplex = Metaplex
    .make(connection)

  const nfts = metaplex.nfts()

  async function getUserNfts() {
    if (!wallet.publicKey) return

    // Fetch all the user's NFTs
    const userNfts = await nfts
      .findAllByOwner({ owner: wallet.publicKey })

    // Filter to our collection
    const ourCollectionNfts = userNfts.filter(
      metadata =>
        metadata.collection !== null &&
        metadata.collection.verified &&
        metadata.collection.address.toBase58() === COLLECTION_MINT_ADDRESS.toBase58()
    )

    // Load the JSON for each NFT
    const loadedNfts = await Promise.all(ourCollectionNfts
      .map(metadata => {
        return nfts
          .load({ metadata: metadata as Metadata })
      })
    )

    console.log("Got their NFTs!", loadedNfts)
  }

  useEffect(() => {
    getUserNfts()
  }, [wallet.publicKey])

  return (
    <p className="text-lg text-white">Holders only! ☠️</p>
  )
}
```

We can pass a single NFT to `nfts.load()` to have its JSON loaded. Each of these is an independent task, so we use `Promise.all` to load all of them. Now if we refresh the page we should see the JSON loaded:

<img width="1000" alt="28" src="https://github.com/gecginserhat/Solana_NFT/assets/74310970/0094805e-df94-4768-857e-cdf2c58c5b14">


## Displaying their NFTs

Let’s make our holder page a bit more useful - we’ll display the user’s NFTs that we’ve loaded.

First let’s add some state to hold the fetched NFTs:

```
import { Metadata, Metaplex, Nft } from "@metaplex-foundation/js"
import { useConnection, useWallet } from "@solana/wallet-adapter-react"
import { useEffect, useState } from "react"
import { COLLECTION_MINT_ADDRESS } from "../lib/constants"

export default function Holders() {
  const { connection } = useConnection()
  const wallet = useWallet()
  const [userNfts, setUserNfts] = useState<Nft[]>([])

  const metaplex = Metaplex
    .make(connection)

  const nfts = metaplex.nfts()

  async function getUserNfts() {
    if (!wallet.publicKey) {
      setUserNfts([])
      return
    }

    // Fetch all the user's NFTs
    const userNfts = await nfts
      .findAllByOwner({ owner: wallet.publicKey })

    // Filter to our collection
    const ourCollectionNfts = userNfts.filter(
      metadata =>
        metadata.collection !== null &&
        metadata.collection.verified &&
        metadata.collection.address.toBase58() === COLLECTION_MINT_ADDRESS.toBase58()
    )

    // Load the JSON for each NFT
    const loadedNfts = await Promise.all(ourCollectionNfts
      .map(metadata => {
        return nfts
          .load({ metadata: metadata as Metadata })
      })
    )

    console.log("Got their NFTs!", loadedNfts)
    setUserNfts(loadedNfts as Nft[])
  }

  useEffect(() => {
    getUserNfts()
  }, [wallet.publicKey])

  return (
    <p className="text-lg text-white">Holders only! ☠️</p>
  )
}
```

When the `wallet.publicKey` is not set we don’t have a connected wallet, so we set the `userNfts` to be empty. Otherwise we fetch and load the NFTs and store the result.

And finally we can update our return to render the NFTs using our `NftDisplay` component!

```
import { Metadata, Metaplex, Nft } from "@metaplex-foundation/js"
import { useConnection, useWallet } from "@solana/wallet-adapter-react"
import { useEffect, useState } from "react"
import NftDisplay from "../components/NftDisplay"
import PageHeading from "../components/PageHeading"
import { COLLECTION_MINT_ADDRESS } from "../lib/constants"

export default function Holders() {
  const { connection } = useConnection()
  const wallet = useWallet()
  const [userNfts, setUserNfts] = useState<Nft[]>([])

  const metaplex = Metaplex
    .make(connection)

  const nfts = metaplex.nfts()

  async function getUserNfts() {
    if (!wallet.publicKey) {
      setUserNfts([])
      return
    }

    // Fetch all the user's NFTs
    const userNfts = await nfts
      .findAllByOwner({ owner: wallet.publicKey })

    // Filter to our collection
    const ourCollectionNfts = userNfts.filter(
      metadata =>
        metadata.collection !== null &&
        metadata.collection.verified &&
        metadata.collection.address.toBase58() === COLLECTION_MINT_ADDRESS.toBase58()
    )

    // Load the JSON for each NFT
    const loadedNfts = await Promise.all(ourCollectionNfts
      .map(metadata => {
        return nfts
          .load({ metadata: metadata as Metadata })
      })
    )

    console.log("Got their NFTs!", loadedNfts)
    setUserNfts(loadedNfts as Nft[])
  }

  useEffect(() => {
    getUserNfts()
  }, [wallet.publicKey])

  if (userNfts.length === 0) {
    return (
      <PageHeading>Holders only! ☠️</PageHeading>
    )
  }

  return (
    <main className="flex flex-col gap-8">
      <PageHeading>Hello awesome holder!</PageHeading>

      <p className="text-lg text-white">Here is the secret information! ✅</p>

      <div className="grid grid-cols-2 gap-4">
        {userNfts.map((userNft, i) => (
          <NftDisplay key={i} json={userNft.json} />
        ))}
      </div>
    </main>
  )
}
```

Pretty straightforward - we’re just checking whether or not there are any userNfts. If there are then we can show the holders page, including some secret content we only want to show holders and a nice little grid of their NFTs.

So now our page looks like this when I’m logged in:


![29](https://github.com/gecginserhat/Solana_NFT/assets/74310970/6ca7dd26-7a47-4316-88ee-44f20d4f8295)

And when I log out…

![30](https://github.com/gecginserhat/Solana_NFT/assets/74310970/b4db2c07-d602-4e84-9bef-f4fd2d22551d)


We’re done! Users can come to our site, mint our NFTs, and view their minted NFTs - and whatever else we want to show. Congrats!

We created a nice example using the Metaplex SDK. I hope it was useful for you. Solana is awesome for NFTs. 

See you next time
