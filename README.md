# channel_demo_tg_msg_sender

It's a tg bot service, which simply means you set up a payment channel with the server, and then exchange ckb for my [UDT](https://github.com/ZhichunLu-11/ckb-gpc-contract/blob/c8ad9ef42c6dd9e334c5099fa9510cef2997557d/main.c). Then you can have the tg bot pin the message you want to pin to the group. The idea of GPC is from [A Generic Payment Channel Construction and Its Composability](https://talk.nervos.org/t/a-generic-payment-channel-construction-and-its-composability/4697). 
Also, you can view the contract codes for [GPC](https://github.com/ZhichunLu-11/ckb-gpc-contract/blob/c8ad9ef42c6dd9e334c5099fa9510cef2997557d/main.c) and [UDT](https://github.com/ZhichunLu-11/ckb-gpc-contract/blob/c8ad9ef42c6dd9e334c5099fa9510cef2997557d/c/simple_UDT.c).

## Prerequisites

* [ckb testnet](https://github.com/nervosnetwork/ckb) with rpc listen port 8114.
* [ckb indexer](https://github.com/nervosnetwork/ckb-indexer) with listen port 8116.
* [mongodb](https://github.com/mongodb/mongo)

Please make sure that all of the above services are running and synced to the latest blocks. Then you need to have the following ruby module, and also, and I suggest you use ruby 2.6.

* ckb-ruby-sdk
* Thor
* Mongo

And then execute:

``` 

$ bundle install
```

## Usage

First you should run

``` 

cd client
ruby GPC.rb
```

to see all the command. I'm only going to cover the commands you need to run under normal cases here. 

### Init client

``` 

ruby GPC.rb init <Your private key>
```

For example, 

``` 

ruby GPC.rb init 0xd986d7bf901e50368cbe565f239c224934cd554805357338abcef177efadc08d
```

Please don't forget to prefix it with 0x.

### Monitor the chain

``` 

ruby GPC.rb monitor
```

This is to prevent the other party from cheating and to automatically send settlement transactions and so on. 

### Create channel

Then you need to reopen a shell and run 

``` 

ruby GPC.rb send_establishment_request --funding <quantity>
```

For example, 

``` 

ruby GPC.rb send_establishment_request --funding ckb:200 UDT:0
```

Please follow the format of funding, even if the amount of UDT you want to put in is zero. Then, if the channel is established successfully, you should see 

``` 

channel is established, please wait the transaction on chain.
```

Once the transaction on chain, you can see in the shell that runs the monitor

``` 

f119905665a0dea6a3eb6cbdc9bcf75b's fund tx on chain at block number 610468, the tx hash is 0x9f31a56c578047d6bc6714f984b52a9619367eb7acb74e3e7a4403e540d41e57.
```

### List channel

``` 

ruby GPC.rb list_channel
```

This allows you to see the status of all your channels. For example, 

``` 

channel f119905665a0dea6a3eb6cbdc9bcf75b, with 4 payments and stage is 3
 local's ckb: 80 ckbytes, local's UDT 88.
 remote's ckb: 120 ckbytes, remote's UDT 1999912.
```

* 'f119905665a0dea6a3eb6cbdc9bcf75b' is the channel id.

### Exchange UDT

As you can see, you only put in CKB when you build the channel (If you've previously closed the channel while you held UDT, you can also invest in UDT while you're establishing the channel). to make the process more fun, I made the bot recognize UDT as the only currency. So you first need to exchange for UDT. 10 CKBytes can exchange 1 UDT.

``` 

ruby GPC.rb make_exchange_ckb_to_UDT --quantity 20
```

You can replace '20' with any number you want, as long as you have enough ckbytes. Note that one CKB can only be exchanged for one UDT.
Also, you can trade UDT for CKB, just use make_exchange_UDT_to_ckb.

### Inquiry msg id.

If you want to pin a msg already in this chat, you need to know the id of it firstly. 

``` 

ruby GPC.rb inquiry_msg --text <text>
```

For example

``` 

ruby GPC.rb inquiry_msg --text '123'
'123' sent by Zhichun in group_test at 2020-11-12T16:14:49+08:00, the id of this msg is 146.
Timed out. If you fail to receive the data, you should try again.
```

### Pin msg

After knowing the id, you can pin a msg with specific id. 

``` 

ruby GPC.rb pin_msg --id <msg_id> --duration <seconds> --price <price_per_seconds>
```

Duration denotes how long you want to pin this msg, price means the UDT per second you are willing to pay. Since the UDT amount can only be an integer, the total price you enter ends up being rounded upwards. For example, if you input duration with 10 seconds and 0.01 UDT per second. The amount you want to pay is 0.1, but I will round it upward. So the actually amount is 1. In this case, the server will treat your bid as 0.1 UDT per second.

### Send and pin tg msg

Of course, you can let the bot pin msg with any content you want, which you let the bot send a msg and pin it immediately.

``` 

ruby GPC.rb pin_msg --text <text> --duration <seconds> --price <price_per_seconds>
```

### Close the channel

You have the following two ways to close the channel.

1. Bilateral closing

``` 

ruby GPC.rb send_closing_request
```

Closing in this way closes the channel immediately and you just need to wait for a settlement transaction to be on chain.

2. Unilateral closing

``` 

ruby GPC.rb close_channel
```

The advantage of this method is that you don't need to interact with the other party. However, please note that you are committing a closing transaction instead of a settlement transaction, and the Monitor will commit the corresponding settlement transaction after a hundred blocks of the closing transaction on chain.

## FAQ

### This application clearly doesn't require a payment channel to implement, so why are you forcing the inclusion of a payment channel.

This is actually a demo of a payment channel, so I actually just forced it in because I couldn't find a better idea. If you think of a better way to use it, please let me know.

### Can I treat this bot as an application that sends messages anonymously?

I was going to call it an anonymous bot, but I realized that the server has access to your IP (Although I don't actually record your IP.), and your public key on the ckb test network. So I just call it msg_sender. of course. But if you trust me, you can think of it as a bot that helps you send messages anonymously.

###  Why is it so un-robust? If I have a problem on the way to an interaction(Like disconnecting the Internet.), it doesn't even work!

You make a good point, it is indeed very un-robust and I apologize for that, at the moment I haven't done some implementation that adds the robustness function (retransmission mechanism etc). So when you find that it doesn't work, one of the best ways to close that channel is by unilaterally closing it and then reopening it later.
