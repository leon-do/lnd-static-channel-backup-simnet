works when cd to this directory
$ cd static-chan-backup

install btcd https://github.com/btcsuite/btcd
btcd$ btcd --txindex --simnet --rpcuser=kek --rpcpass=kek

start alice's lnd
alice$ ./lnd-debug --rpclisten=localhost:10001 --listen=localhost:10011 --restlisten=localhost:8001 --datadir=./alice --bitcoin.simnet --bitcoin.active --bitcoin.node=btcd --btcd.rpcuser=kek --btcd.rpcpass=kek --no-macaroons

create alice's wallet
alice$ ./lncli-debug --rpcserver=localhost:10001 --no-macaroons create

---------------BEGIN LND CIPHER SEED---------------
 1. absorb   2. balcony   3. live      4. stuff
 5. sugar    6. large     7. south     8. floor
 9. plunge  10. turn     11. ostrich  12. lounge
13. pear    14. clump    15. sun      16. list
17. add     18. upgrade  19. soldier  20. food
21. pretty  22. advice   23. amazing  24. sock
---------------END LND CIPHER SEED-----------------

generate address for alice
alice$ ./lncli-debug --rpcserver=localhost:10001 --no-macaroons newaddress np2wkh
{
    "address": "rssZ8WH5N9jJvRdCHjkDwVfTDKo5GZdvK3"
}

restart btcd to mine to alice's address
btcd$ btcd --simnet --txindex --rpcuser=kek --rpcpass=kek --miningaddr=sb1qgnrsmwmwczpxagat27rkhrd996kvdnu4jhcy6f

mine bitcoind
alice$ btcctl --simnet --rpcuser=kek --rpcpass=kek generate 400

check alice's balance
alice$ ./lncli-debug --rpcserver=localhost:10001 --no-macaroons walletbalance
{
    "total_balance": "1505000000000",
    "confirmed_balance": "1505000000000",
    "unconfirmed_balance": "0"
}

start bob's lnd
bob$ ./lnd-debug --rpclisten=localhost:10002 --listen=localhost:10012 --restlisten=localhost:8002 --datadir=./bob --bitcoin.simnet --bitcoin.active --bitcoin.node=btcd --btcd.rpcuser=kek --btcd.rpcpass=kek --no-macaroons

create bob's wallet
bob$ ./lncli-debug --rpcserver=localhost:10002 --no-macaroons create

---------------BEGIN LND CIPHER SEED---------------
 1. above     2. bomb      3. sadness   4. brass
 5. gallery   6. chalk     7. airport   8. fame
 9. float    10. auction  11. guess    12. net
13. lava     14. skill    15. maple    16. ankle
17. month    18. gain     19. trip     20. unique
21. invite   22. cactus   23. lift     24. inherit
---------------END LND CIPHER SEED-----------------

get bob's pubkey
bob$ ./lncli-debug --rpcserver=localhost:10002 -no-macaroons getinfo 
{
	"version": "0.5.2-99-beta commit=",
	"identity_pubkey": "02ec2d48da0f83bb44368aa11dbf304a1ba5abd68c4fb9eaa9f61e6f0dca0b72b1",
  ...
}

connect alice to bob
alice$ ./lncli-debug --rpcserver=localhost:10001 -no-macaroons connect 02ec2d48da0f83bb44368aa11dbf304a1ba5abd68c4fb9eaa9f61e6f0dca0b72b1@localhost:10012

open channel with bob
alice$ ./lncli-debug --rpcserver=localhost:10001 -no-macaroons openchannel --node_key=02ec2d48da0f83bb44368aa11dbf304a1ba5abd68c4fb9eaa9f61e6f0dca0b72b1 --local_amt=123456

mine blocks so channel is valid
alice$ btcctl --simnet --rpcuser=kek --rpcpass=kek generate 6

bob creates invoice
bob$ ./lncli-debug --rpcserver=localhost:10002 -no-macaroons addinvoice --amt=123
{
	"r_hash": "ca95415ed8fbe235c33f8a4c72317a1315264c5d46582a57961ebea67286d9c8",
	"pay_req": "lnsb1230n1pwg42cdpp5e225zhkcl03rtsel3fx8yvt6zv2jvnzagevz54ukr6l2vu5xm8yqdqqcqzys69gx8mhugukpsfzc5z0h7uc2ykypznu777eu54punrxcld00atc4yss6tc4endy9z8rpnq8zpcuk0nc9mzmqmld24paw6r29vaepclsqjwka6h",
	"add_index": 1
}

alice pays bob's invoice
./lncli-debug --rpcserver=localhost:10001 -no-macaroons sendpayment --pay_req=lnsb1230n1pwg42cdpp5e225zhkcl03rtsel3fx8yvt6zv2jvnzagevz54ukr6l2vu5xm8yqdqqcqzys69gx8mhugukpsfzc5z0h7uc2ykypznu777eu54punrxcld00atc4yss6tc4endy9z8rpnq8zpcuk0nc9mzmqmld24paw6r29vaepclsqjwka6h

verify alice sent payment
$alice ./lncli-debug --rpcserver=localhost:10001 -no-macaroons listchannels
{
    "channels": [
        {
            "active": true,
            "remote_pubkey": "02ec2d48da0f83bb44368aa11dbf304a1ba5abd68c4fb9eaa9f61e6f0dca0b72b1",
            "channel_point": "9bd26e335655ef70668a34f15dbd0a11b75bba849b2c1943f6dcba6fcdd68cd2:0",
            "chan_id": "880708813914112",
            "capacity": "123456",
            "local_balance": "114283",
            "remote_balance": "123",
            "commit_fee": "9173",
            "commit_weight": "600",
            "fee_per_kw": "12500",
            "unsettled_balance": "0",
            "total_satoshis_sent": "123",
            "total_satoshis_received": "0",
            "num_updates": "2",
            "pending_htlcs": [
            ],
            "csv_delay": 144,
            "private": false,
            "initiator": true,
            "chan_status_flags": "ChanStatusDefault"
        }
    ]
}

verify bob recieved payment
$bob ./lncli-debug --rpcserver=localhost:10002 -no-macaroons listchannels
{
    "channels": [
        {
            "active": true,
            "remote_pubkey": "0376902731608e0d6c015f5585fc616a4c8050bfae25d065936df30458ee62bb20",
            "channel_point": "9bd26e335655ef70668a34f15dbd0a11b75bba849b2c1943f6dcba6fcdd68cd2:0",
            "chan_id": "880708813914112",
            "capacity": "123456",
            "local_balance": "123",
            "remote_balance": "114283",
            "commit_fee": "9173",
            "commit_weight": "552",
            "fee_per_kw": "12500",
            "unsettled_balance": "0",
            "total_satoshis_sent": "0",
            "total_satoshis_received": "123",
            "num_updates": "2",
            "pending_htlcs": [
            ],
            "csv_delay": 144,
            "private": false,
            "initiator": false,
            "chan_status_flags": "ChanStatusDefault"
        }
    ]
}


https://wiki.ion.radar.tech/tech/research/static-channel-backups
alice$ ./lncli-debug --rpcserver=localhost:10001 -no-macaroons exportchanbackup --all
{
	"chan_points": [
		"9bd26e335655ef70668a34f15dbd0a11b75bba849b2c1943f6dcba6fcdd68cd2:0"
	],
	"multi_chan_backup": "3ee57e9e62d6a10fa37b92c8b612b783ebdfb94beeee8c8b72a2cc832c120d4338b337ad58403fc7bd8ec73ef78587a0369e123a9fa1263884336e01e6fb09e315307dee0d77b9edd3f55dddd325811f34ab381a6fc5152951ef5ed2d2ec9f3e4dd0a5cfe5ebeb838b728ff813f940a3c9f58fbde156d8d771984560f30ce078801513c3b0e81cab626c8ce181f4f32a97e06244a421c07d982cf725a4b472ce2b3f197defe71826e28ecf8d8d8ea5e770afebcf959afa09336d6c5fe22b18831f455a1987f86f7a4a5560648a490a83e458ca5611f1c871f5c5718d6c2887de79442d9288561192d5be3e2778ff549f9444cda38e4ba6bad62bb0b68e9aa4e74ebd9f0deeb473facfc82da967a0992b4a4a6d6502f69cfe0eb44a0371080513ab8b51179c0979cd72336328f500d6d65c010cbf8a36cbab22f45cc19a9ec84170468fe891eb89f73db5e0a6fcdbe1ec350e481288bda6b8300b7c6751b72da0b7b0c594640bb2be30333eb5dcf458bd546ce5dd3a248e4084179ed5a9a68243587e8ef8e7ca2097bd75c8ab3934fed6d7b497017f0052066781364d0ad849f369df495f793a9f"
}

remove alice's wallet

// restore alice's wallet
alice$ ./lncli-debug --rpcserver=localhost:10001 --no-macaroons create
Input your 24-word mnemonic separated by spaces: absorb balcony live stuff sugar large south floor plunge turn ostrich lounge pear clump sun list add upgrade soldier food pretty advice amazing sock

// verify alice has an on-chain balance
alice$ ./lncli-debug --rpcserver=localhost:10001 --no-macaroons walletbalance

// alice's channels are obviously gone
$alice ./lncli-debug --rpcserver=localhost:10001 --no-macaroons listchannels

// restore alice's channel
$alice ./lncli-debug --rpcserver=localhost:10001 --no-macaroons restorechanbackup --multi_backup 3ee57e9e62d6a10fa37b92c8b612b783ebdfb94beeee8c8b72a2cc832c120d4338b337ad58403fc7bd8ec73ef78587a0369e123a9fa1263884336e01e6fb09e315307dee0d77b9edd3f55dddd325811f34ab381a6fc5152951ef5ed2d2ec9f3e4dd0a5cfe5ebeb838b728ff813f940a3c9f58fbde156d8d771984560f30ce078801513c3b0e81cab626c8ce181f4f32a97e06244a421c07d982cf725a4b472ce2b3f197defe71826e28ecf8d8d8ea5e770afebcf959afa09336d6c5fe22b18831f455a1987f86f7a4a5560648a490a83e458ca5611f1c871f5c5718d6c2887de79442d9288561192d5be3e2778ff549f9444cda38e4ba6bad62bb0b68e9aa4e74ebd9f0deeb473facfc82da967a0992b4a4a6d6502f69cfe0eb44a0371080513ab8b51179c0979cd72336328f500d6d65c010cbf8a36cbab22f45cc19a9ec84170468fe891eb89f73db5e0a6fcdbe1ec350e481288bda6b8300b7c6751b72da0b7b0c594640bb2be30333eb5dcf458bd546ce5dd3a248e4084179ed5a9a68243587e8ef8e7ca2097bd75c8ab3934fed6d7b497017f0052066781364d0ad849f369df495f793a9f
