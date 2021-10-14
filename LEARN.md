# Netflix clone to pay subscription by the second, not by the month
If you’ve used Netflix or any other online software service, you’ve probably paid the service every month. But why? You’ve been paying every month because someone needs to sit down and do the balance sheets once to analyse the profits and losses. But with web3 and blockchain transactions, every transaction is automatically recorded. Monthly payments are so outdated. In this quest we will see how we can setup subscription services that allow users to be able to pay per second.

This quest requires that you have understanding of how crypto currencies work. It also requires you understand UI elements of Web3 (building a dapp). If not do checkout the cryptocurrencies path on [https://questbook.app](https://questbook.app) . You also need to know react to build the UI.
## Firing multiple transactions very quickly
The simplest way to do a per second payment is to actually fire up a transaction every second. 

Let’s do that.

We’ll be doing this using ethers.js to do this.

Create a html file called 100transactions.html

In the head, include the ethers.js 

```

	

```

We’ll now write a function that will make 100 transactions. We’ll mimic the usage that the user is paying per second. You can also try to add a sleep to delay it such that you’re paying per second.

In this function, we’ll first initialize the ethers object.

```

const provider = new ethers.providers.Web3Provider(window.ethereum, "any");

const signer = provider.getSigner();

```

`window.ethereum` is usually not available in javascript. However it becomes available when the user has installed metamask on their browser. 

The signer is who will sign the transactions off. Recollect that whenever you want to send money from your wallet it needs to be signed off by the account - representing which account the money should be debited from. The `signer ` object represents the account on the metamask of the user. 

We’ll send transactions in a for-loop. In the transaction object we need to send money `to` a particular eth account and the `value` depicts how much.

```

	

```

Fire off this html file on your browser.

Metamask can operate only on `http://` or `https://` websites.

So you can run the following command in your terminal in the directory where your html file resides : 

```

python3 -m http.server

```

This is a super handy way to setup an http server so that your html file can be accessed from the browser using `[http://localhost:8000/100transactions.html](http://localhost:8000/100transactions.html)`

When you open the site on your browser, you’ll be prompted to connect the metamask first. 

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/0a4875d8-b55e-4e8d-aaa6-4cc18f7438c9.jpg)

This is basically how Metamask will evaluate which account is going to be the `signer` on this webpage.

Once connected refresh the page.

And you will see a barrage of requests on your metamask. 

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/cec2a541-e1ad-458e-95d5-5afa9b0425d1.jpg)

100 transactions got created. If you want to approve all of them you will have to click confirm on each of them one after another. This is clearly not a good way to have micro-payments made.

Here’s the html file : 

[https://gist.github.com/madhavanmalolan/9453a7cd311fe075941892937216c1c9](https://gist.github.com/madhavanmalolan/9453a7cd311fe075941892937216c1c9)
## It costs money, each of the 100 times
If you notice in the code, we were sending a value of `0.00000001` ETH. But in the metamask pop-up the amount is `0.000032` ETH for each transaction. Why is that?

This is called gas. You need to pay a transaction fees for submitting any transaction on Ethereum. This is the money that is collected by the people running computers to support the Ethereum World Computer. Sending money is not free. For each transaction, if we are to pay a transaction fees, we’ll get drained! As you’d see, the number of ETH we’re sending is actually less than the number ETH we’re having to spend to make the transaction. 

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/cbe7530d-d12e-401d-bf8b-2f2ac023041a.jpg)

So, we’ll look at a new technology called money streams. We’ll use a protocol called SuperFluid.
## Creating a react app
We’ll create a react app where you can start creating your subscription service.

```

npx create-react-app my-app

```

Once you’ve created the react-app, let us add a video. This video will act as the video on our streaming service. We’ll use a youtube video for this example.

```

//app.js 



      

    

```

If you run `npm start`, you’ll see the video embedded. But we want to show the video only if there is a valid subscription that is paying for the services.
## Setting up SuperFluid
Let’s start using Superfluid that will be our subscription provider for this quest. 

```

yarn add @superfluid-finance/js-sdk @ethersproject/contracts @ethersproject/providers ethers

```

This will install the superfluid sdks so that we can start building a subscription payment system where the users are charged per second.

Now let’s start using this in the project. 

```

//app.js

const SuperfluidSdk = require('@superfluid-finance/js-sdk');

const {Web3Provider} = require('@ethersproject/providers');

const { ethers } = require("ethers");

```

We also need to import Web3Provider so that we interact with Metamask.

Let’s initialize Superfluid

```

const init = async function() {

    const provider = new Web3Provider(window.ethereum);

    const sf = new SuperfluidSdk.Framework({ ethers: provider});

    await sf.initialize();  

}

useEffect(() => init());

```

Here we’re using the Superfluid Sdk using the ethers Framework. There are multiple libraries available for interacting with Metamask from the browser. Ethers.js is one of them. We’re telling Superfluid to use ethers.js and not web3.js. Web3.js is another library for interacting with metamask.
## Checking for an existing subscription
Let’s first initialize the user on superfluid. We need to tell Superfluid the address of the user and the token in which the payments is expected.

```

    const user = sf.user({

      address: await provider.getSigner().getAddress(),

      token: '0x6fC99F5591b51583ba15A8C2572408257A1D2797'

    });

```

`provider.getSigner().getAddress()` gets the address from the connected metamask wallet. The token tells superfluid that we’re going to do all subscriptions related activities using the said ERC20 token. 

Superfluid converts every token into a token in it’s own format. ETH becomes ETHx, DAI becomes DAIx and so on. You can get the value address of the token from here : [https://docs.superfluid.finance/superfluid/networks/networks](https://docs.superfluid.finance/superfluid/networks/networks) 

The snippet above is that of ETHx on the Ropsten Network. We will be accepting payments only in ETH via superfluid and we’ll be doing so on the Ropsten network.

Subscriptions on Superfluid are called flows - as in a flow of money, also called constant flow agreements (CFA). 

Let’s check if there’s a flow already.

```

    const details = await user.details();

    if(details.cfa && details.cfa.flows.outFlows){

      // subscriptions exist

    }

    else {

      setSubscribed(false);

    }

```

We’ll check if this user already has an outflow. If they don’t have one, they ofcourse don’t have a subscription. We’ll set the state variable `subscribed` to `false` and show a subscribe button.

```

  const \[subscribed, setSubscribed\] = useState();

  if(!subscribed)

    return (

      

         subscribed()}>Subscribe

      

    )

```
## refilling Superfluid wallet
To be able to create a superfluid subscription or flow, we need to add money (ETHx) to superfluid. 

To do this, go to [https://app.superfluid.finance/](https://app.superfluid.finance/) 

Add some eth to the wallet by selecting the deposit button. Make sure you’re on the Ropsten Network.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/3b46583c-72a2-4285-9188-3d93c2f68678.jpg)

Once it is successful, you’ll see something like this :

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/10227271-d45e-4439-a098-faad097fd177.jpg)

If you deposited $3000 worth of ETH, you’ll have the exact same amount of ETHx in the wallet. 

Now that we’ve refilled the account, let’s create a subscription in our code.
## Starting a new subscription
Once the ETH has been converted into ETHx, we’re ready to start paying a continuous subscription on superfluid.

```

    const flow = await user.flow({

      recipient: '0x89Ce0f71D7387a580c6C07032f74f393a65d77F4',

      flowRate: parseInt(0.01 \* 1e18 / ( 60\* 60\* 24\* 30)).toString() // 0.01 wei per month - paid every second

    });

```

Here’s the entire subscribe function with initializations : [https://gist.github.com/madhavanmalolan/d6e01a7e43887899bc39dc32a8576a56](https://gist.github.com/madhavanmalolan/d6e01a7e43887899bc39dc32a8576a56) 

The recipient is to whom the amount will be paid. Remember, the recipient and the payer must be different addresses, else the subscription will fail. We also need to mention what is the `flowrate`. Flow rate is how much money will be sent per second to the recipient. This is calculated in wei. In this case we set the price of the service as 0.01ETH (or 0.01 x 10^18 wei) per month. So we convert that into the amount that must be paid per second. 

Change your metamask wallet address to an account other than the recipient mentioned above & refresh the page. 

When you hit subscribe and call the `subscribe()` function, a new flow is created and metamask asks you for an approval.

Once you approve, you can go to the superfluid dashboard [https://app.superfluid.finance](https://app.superfluid.finance) and see that the money has started flowing from your account to the recipient account. The money is deducted every second!
## Providing the service
Now that you have started accepting money over a subscription flow, it’s time to actually show the videos the user has come here for.

Let’s revisit our check in the `init()` function.

If the flow exists, the following condition is true : 

```

if(details.cfa && details.cfa.flows.outFlows){

}

```

We’ll quickly check if there exists a flow “to” the recipient we want.

```

    if(details.cfa && details.cfa.flows.outFlows){

      for(let i = 0; i &lt; details.cfa.flows.outFlows.length; i\+= 1){

        const outFlow = details.cfa.flows.outFlows\[i\];

        if(outFlow.receiver === "0x89Ce0f71D7387a580c6C07032f74f393a65d77F4"){

          setSubscribed(true);

        }

      }

    }

```

And with that you should be able to refresh the page to see the subscribed video.  


The user has access to the page as long as they keep paying. If you go and stop the flow from the superfluid dashboard, you won’t be able to access the video any more!

  
Here’s the full code we’ve written :

[https://gist.github.com/madhavanmalolan/5a97374b79eea22dedf9e0392739cf48](https://gist.github.com/madhavanmalolan/5a97374b79eea22dedf9e0392739cf48)

#
## What next
Now that you have created a simple subscription. Can you try to modify this to include a pause subscription button directly from the same page?

There’s a prize to be won! The first 5 developers to create this feature satisfactorily will receive upto $200 in prizes from Superfluid, and an additional NFT from Questbook! Hurry, send us a loom video with what you’ve created!