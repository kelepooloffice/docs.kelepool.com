# How to query user staking information?

Cola Mining Pool has opened all ETH2.0 revenue query interfaces. Developers can use wallet addresses to query users' staking amount, total revenue, APR, curve (hours, days), node status, operation history, historical revenue and other data.

## API Node

[Kele Pool Mainnet API：https://api.kelepool.com](https://api.kelepool.com)

[Kele Pool Goerli API：https://test-api.kelepool.com](https://test-api.kelepool.com)

> Generic request returns result:
> - `code` : an integer number, equal to 0 for success, greater than 0 for failure
> - `message` : the message to return after failure


## API Authorization
Third-party developers need to contact Kele Pool to apply for a long-term valid signature `authority_key` and `token`, the third party can use these two keys for signature and data source confirmation.

### 1. Authorization step
- Prepare an eth address as the receiving address of the partner's mev fee (the address is recommended for collection, and the flow of funds is clearer)
- Choose a `graffiti` logo as the name of the node on the network, such as BXKelePool
- Contact Cola Pool to apply for `authority_key` and `token`/`source`
- If the user is calling the Coke Pool API for the first time, he needs to call `/user/v2/anonymouslogin` in advance to [user address registration] (#user address registration), and pass the agreed source parameter value (partner source identification )
- Use `authority_key` and `token` to sign each interface of the Coke mining pool and put it in the Header for verification

### 2. How to use

- Add Kele-ThirdParty-Authority=`token` in the request header
- Add Kele-ThirdParty-Sign = `sign` to the request header
    - The logic for getting `sign` is as follows:
    - Arrange request parameters in ascending lexicographical order and use '&' to concatenate
    - Sign with `authority_key` with `hmac_blake2b`, get `sign`

```json
Only for test : authority_key & token

{
    "authority_key": "2fb8098e1cac29c559191993e606e692b7d15314164ac8c55bcaa5a05b635843f067a35bf50ab9707675f7dff7dae934f6b2c189311e9c53ba874f572643b8ed",
    "token": "eyJ0eXAiOiJqd3QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoiZnVsbCIsIm9wZW5pZCI6Im9uZWtleSIsInZlcnNpb24iOiIwIiwiZXhwIjoxODMwNTcwNTU4fQ.gNdTZxcThOBKJB2oGFUAC1vxP9FRXQBPPx36jpgZRWc"
}
```


### 3.Python Sample code
```python

import hashlib
import hmac
import requests


url = 'https://test-api.kelepool.com/eth2/v2/miner/dashboard?address=0xf48b98bbeeb81033a227f576da98a32c3a2d8515'
params = {
    'address':'0xf48b98bbeeb81033a227f576da98a32c3a2d8515'
}

# url = 'https://test-api.kelepool.com/eth2/v2/partner/validator'
# params = {

# }

sign_str = '&'.join(['%s=%s' % (k, params[k]) for k in sorted(params)])

authority_key='dccc6ce732fe9011ee4e12b2e0de8ecbe743f630f3ff02bceb23052d9afa692d50540d6221f095427f903db80f781dd6cfaef8c6678ad5bbcc74475cd76cf629'
token='eyJ0eXAiOiJqd3QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoiZnVsbCIsIm9wZW5pZCI6Ik9uZUtleSIsInZlcnNpb24iOiIwIiwiZXhwIjoxODM3MDYzMjE0fQ.Z22fUeKbo6AmrsvvJ2nrrDjXQCcMwFd7GtIIAGUe6DU'

sign=hmac.new(authority_key.encode('utf-8'), sign_str.encode('utf-8'), digestmod=hashlib.blake2b).hexdigest()
headers = {
'Content-Type': 'application/json', 
'Accept':'application/json',
'Kele-ThirdParty-Authority':token,
'Kele-ThirdParty-Sign':sign
}

r_json = requests.get(url,params=params,headers=headers)
print()
print("paramters: "+sign_str)
print()
print("signature: "+sign)
print()
print("response: "+r_json.text)

```

### 4.NodeJs Sample code

- yarn add sodium-universal
- yarn add request

```javascript


var { sodium_malloc, sodium_memzero } = require('sodium-universal/memory')
var { crypto_generichash, crypto_generichash_batch } = require('sodium-universal/crypto_generichash')

// calculate signature
function hmac (data, key) {
  var mac = Buffer.alloc(64)
  var scratch = sodium_malloc(128 * 3)
  var hmacKey = scratch.subarray(128 * 0, 128 * 1)
  var outerKeyPad = scratch.subarray(128 * 1, 128 * 2)
  var innerKeyPad = scratch.subarray(128 * 2, 128 * 3)
  if (key.byteLength > 128) {
    crypto_generichash(hmacKey.subarray(0, 64), key)
    sodium_memzero(hmacKey.subarray(64))
  } else {
    hmacKey.set(key)
    sodium_memzero(hmacKey.subarray(key.byteLength))
  }
  for (var i = 0; i < hmacKey.byteLength; i++) {
    outerKeyPad[i] = 0x5c ^ hmacKey[i]
    innerKeyPad[i] = 0x36 ^ hmacKey[i]
  }
  sodium_memzero(hmacKey)
  crypto_generichash_batch(mac, [innerKeyPad].concat(data))
  sodium_memzero(innerKeyPad)
  crypto_generichash_batch(mac, [outerKeyPad].concat(mac))
  sodium_memzero(outerKeyPad)
  return mac.toString('hex')
}

// contact paramaters
function combines(data){
  var builder = []
  Object.entries(data).sort((a,b)=> (a[0].localeCompare(b[0]) || a[1].localeCompare(b[1]))).forEach((item,index)=>{
    builder.push(item[0] +'=' + item[1])
  })
  return builder.join("&")
}

// request
function execute(){

  var params = {
    'address':'0xf48b98bbeeb81033a227f576da98a32c3a2d8515'
  }
  var url = 'https://test-api.kelepool.com/eth2/v2/miner/dashboard?address=0xf48b98bbeeb81033a227f576da98a32c3a2d8515'

  // var url = 'https://test-api.kelepool.com/eth2/v2/partner/validator'
  // var params = {}

  var authority_key='dccc6ce732fe9011ee4e12b2e0de8ecbe743f630f3ff02bceb23052d9afa692d50540d6221f095427f903db80f781dd6cfaef8c6678ad5bbcc74475cd76cf629'
  var token='eyJ0eXAiOiJqd3QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoiZnVsbCIsIm9wZW5pZCI6Ik9uZUtleSIsInZlcnNpb24iOiIwIiwiZXhwIjoxODM3MDYzMjE0fQ.Z22fUeKbo6AmrsvvJ2nrrDjXQCcMwFd7GtIIAGUe6DU'
  var parameters = combines(params)
  var key = Buffer.from(authority_key,'utf8')
  var data = Buffer.from(parameters,'utf8')
  var signature = hmac(data, key)

  console.log("paramaters: "+parameters)
  console.log("signature: "+signature)

  const request = require('request');
  request({
    url: url,
    headers: {
      'Content-Type': 'application/json', 
      'Accept':'application/json',
      'Kele-ThirdParty-Authority':token,
      'Kele-ThirdParty-Sign':signature
    }
  }, (error, response, body) => {
    if (!error && response.statusCode == 200) {
      const data = JSON.parse(body);
      console.log(data);

    }
  });
}


execute();

```



## ETH private key signature authorization

### 1.How to use

- Add Kele-Private-Sign=`Kele-Private-Sign` in the request header
- Add "_pirv_sign_raw":"sign input data" to the request json body

- _pirv_sign_raw Information (Input data signed as a private key after json stringfy)
```json
{
    "sign_time":1651200959, // signature time
    "token":"eth", // signature algorithm
    "addr":"0x71c7aDBF701f5724291953561790c9c4e870b029",// signer
    "url":"/eth2/v2/miner/unstake", // request api routing
    "method":"post", // request api method
    "api_param":{ // request api parameters
        "source":"kelepool",
        "type":"retail",
        "address":"0xd8f8799bc41b9eb55b5c22c6f75e54b5b98f6f87",
        "unstake_amt":"123.3244"
    }
}
```

Sample api request body
```json
{
    "_pirv_sign_raw":"{\"sign_time\":1651200959,\"token\":\"eth\",\"addr\":\"0x71c7aDBF701f5724291953561790c9c4e870b029\",\"url\":\"/eth2/v2/miner/unstake\",\"method\":\"post\",\"api_param\":{\"source\":\"kelepool\",\"type\":\"retail\",\"address\":\"0xd8f8799bc41b9eb55b5c22c6f75e54b5b98f6f87\",\"unstake_amt\":\"123.3244\"}}"
    "source":"kelepool",
    "type":"retail",
    "address":"0xd8f8799bc41b9eb55b5c22c6f75e54b5b98f6f87",
    "unstake_amt":"123.3244"
}
```


### 2.Sample Python signature

```python

import web3
from eth_account.messages import encode_defunct

priv = '0x004a79ef53fc93c919201f4bfe00ee28cc701627899da0147dee6e4adf0ec52b'
addr = '0xaF73D1072794A386F9505906299F3E2e963581ce'
input = 'input msg 6a957501785f6c211e606c1fd945169a3f35691f3b9be11146146200e99a8bcd'
# https://eips.ethereum.org/EIPS/eip-191
sign_str = web3.eth.Account.sign_message(encode_defunct(input.encode()), priv).signature.hex()
print("sign_str",sign_str) # 0x011bb13f789dcdbbb0e407e071751ae2d6b4726525cdc8791b9af96efd77f95262143b64faca866843c24031411ac95a1ad7fb69ed0ca502580b030b00e624fc1c

# Signature verification
signer = web3.eth.Account.recover_message(encode_defunct(input.encode()), signature=bytes.fromhex(sign_str[2:]))
print(signer, addr) # 0xaF73D1072794A386F9505906299F3E2e963581ce 0xaF73D1072794A386F9505906299F3E2e963581ce
```


### 3.Sample JS signature

```js
// npm install ethers

import { ethers } from 'ethers'

const privKey = '57973a896b37e2ed2228162e4d0d448795f3b2515c198bf4c19812c3f1ee94f0'

const message = 'hello sign message'

const signer = new ethers.Wallet(privKey)

// Signing the message
const sig = await signer.signMessage(message)
console.log(sig)
// 0x4c89155fd4068e96f3f58a39330f1e58a705bee289d0af1ccf4fd8299851fc1e4b372dce0b80c5c9d47729242ac56f8f2b72ba59ba8225765693f5e6fc2478081c

const address = await signer.getAddress()

console.log('Does it match the address', address == ethers.utils.verifyMessage(message, sig))
// Does it match the address true
```

## User Address Registration
#### POST [/user/v2/anonymouslogin](https://test-api.kelepool.com/user/v2/anonymouslogin)

This interface only needs to be called when the user stakings for the first time. Of course, you can also call it every time the user stakings. Note that this interface must be called before the user stakings.

> Request parameters:
> - `payee_addr` : User staking wallet address
> - `token` : the staked token (eth)
> - `source` : The data source is convenient for business cooperation statistics (eg: ThirdParty)

```bash
https://test-api.kelepool.com/user/v2/anonymouslogin

{
    "payee_addr":"0xA49F98416aa4B158c2e752FD8031Fb295D330B22",
    "token":"eth",
    "source":"ThirdParty"
}
```

> Request return value:
> - Judging that `code` is 0 means success, otherwise the registration fails
> - return `token` is not a required field for authentication, ignore it
> - Other fields returned are ignored and not used as registered address
```bash
{
   "code":0,
   "message":"success"
}
```

## User Staking Overview
#### GET [/eth2/v2/miner/dashboard](https://test-api.kelepool.com/eth2/v2/miner/dashboard?address=0x5dd3bd08cbc8498c8640abc26d19480219bb0606&interval=day)

> Request parameters:
> - `address` : user's staking wallet address
> - `interval` : returns yield curve type hour=hour, day=day
> - `num2str` ：whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v2/miner/dashboard?address=0x5dd3bd08cbc8498c8640abc26d19480219bb0606&interval=day&num2str=1
```

> Response Result:
> - `total_amount` : total amount staked (ETH)
> - `staked_amount` : Amount taken (ETH)
> - `staking_amount` : Amount to take effect (ETH)
> - `ongoing_amount` ：withdrawable amount（recommended to use withdrawable）
> - `withdrawable` ：withdrawable amount（ETH）
> - `retail_staked` ：amount of retail staked（ETH）
> - `retail_unstaking` ：amount of retail unstaking（ETH）
> - `whale_staked` ：amount of whale staked（ETH）
> - `whale_unstaking` ：amount of whale unstaking（ETH）
> - `total_reward` : consensus total reward (ETH)
> - `mev_total_reward` : mev total reward (ETH)
> - `staked_days` : total number of days staked
> - `apr` ：estimate total APR
> - `apr_detail`.`basic` ：estimate consensus APR
> - `apr_detail`.`mev` ：estimate mev APR
> - `total_validaters` : total number of validators
> - `unactived_validater` : the number of nodes to be valid
> - `actived_validater` : Number of active nodes
> - `closed_validater` : number of closed nodes
> - `reward` : the consensus reward (ETH) on the graph
> - `mev_reward` : the mev reward (ETH) on the graph (It should be noted that mev revenue is settled immediately and consensus revenue is settled daily. The settlement progress of the two is different)
> - `snap_time` : time on the graph
```json
{
    "code":0,
    "message":"success",
    "data":{
        "amount":{
            "total_amount":173.3,
            "staked_amount":173.23,
            "staking_amount":0.07,
            "ongoing_amount":0,
            "withdrawable":"0.123",
            "retail_staked":"0.123",
            "retail_unstaking":"0.123",
            "whale_staked":"0.123",
            "whale_unstaking":"0.123"
        },
        "income":{
            "total_reward":0.82885946,
            "mev_total_reward": 0.006513841990230327,
            "staked_days":34,
            "apr":0.0487,
            "apr_detail":{
                "basic":0.0367,
                "mev":0.012
            }
        },
        "validater":{
            "total_validaters":8,
            "unactived_validater":1,
            "actived_validater":7,
            "closed_validater":0
        },
        "income_curve":[
            {
                "reward":"0.02563727",
                "mev_reward": "0.000145631351140014",
                "snap_time":"2022-06-13 00:00:00"
            },
            {
                "reward":"0.02423282",
                "mev_reward": "0.000145631351140014",
                "snap_time":"2022-06-14 00:00:00"
            }
        ]
    }
}
```

## Platform Data Overview
#### GET [/eth2/v2/global](https://test-api.kelepool.com/eth2/v2/global)

> Request parameters:
> - `num2str` ：whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v2/global&num2str=1
```

> Response Result:
> - `whale_fee` : large staking fee (ETH)
> - `retail_fee` : Small staking fee (%)
> - `online_ratio` : node online ratio (%)
> - `reward_cycle` : Calculate the time of dividend income
> - `reward_total` : total platform revenue (ETH)
> - `staking_ratio` : ETH network staking ratio (%)
> - `staking_total` : The total amount of staking on the platform (ETH)
> - `validator_total` : The total number of validator nodes on the platform
> - `predicted_reward` : estimate total APR
> - `apr_detail`.`basic` ：estimate consensus APR
> - `apr_detail`.`mev` ：estimate mev APR
> - `whale_min_amount` : Minimum stake amount for large amount (ETH)
> - `retail_min_amount` : Small minimum stake amount (ETH)
> - `retail_deposit_far` : How much ETH is left for the small staking to create a validator
> - `withdraw_predicted_hour` : How long will the withdrawal take to the account
> - `validator_alive_predicted_hour` : how many hours after the validator is created by the staking now, the validator will take effect
```json
{
    "code":0,
    "message":"success",
    "data":{
        "whale_fee":0.05,
        "retail_fee":0.1,
        "online_ratio":1,
        "reward_cycle":"00:00-24:00 (UTC+0)",
        "reward_total":1.72538703,
        "staking_ratio":0.13682066,
        "staking_total":486.03,
        "validator_total":15,
        "predicted_reward":0.0487,
        "whale_min_amount":32,
        "retail_min_amount":0.01,
        "retail_deposit_far":27.6,
        "withdraw_predicted_hour":216,
        "validator_alive_predicted_hour":24,
        "apr_detail":{
                "basic":0.0367,
                "mev":0.012
        }
    }
}
```

## Earnings history list

### Consensus Benefits
#### GET [/eth2/v2/miner/income/query](https://test-api.kelepool.com/eth2/v2/miner/income/query?bill_type=0,1,2&address=0x3ef51b5079021a11b1cab3d36eea45facf2b00ce)

> Request parameters:
> - `address`: user staking wallet address
> - `bill_type`: bill type default value 0,1: query consensus income daily bills; can use 0,1,2: to query the overall daily bill of consensus reward+mev reward
> - `num2str` ：whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v2/miner/income/query?bill_type=0,1,2&address=0x3ef51b5079021a11b1cab3d36eea45facf2b00ce&num2str=1
```

> Request return value:
> - `date` : dividend date
> - `reward` ：Current day earnings
> - `deposit` ：accumulated principal as of the current day (accumulated recharge principal as of the current day - accumulated withdrawal as of the current day)
> - `balance` ：total account balance as of the current day (accumulated recharge principal as of the current day+accumulated income as of the current day - accumulated withdrawal as of the current day)
> - `total_deposit` ：accumulated recharge amount as of the current day
> - `total_withdrawal` ：accumulated withdrawal amount as of the current day
> - `total_reward` ：accumulated income as of the current day

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "date": "2023-04-25 00:00:00",
      "reward": 0.00043776,
      "deposit": 157.68972252,
      "balance": 158.18080674,
      "total_deposit": "254.6243",
      "total_withdrawal": "96.934577477",
      "total_reward": "0.289752687771953251"
    },
    {
      "date": "2023-04-24 00:00:00",
      "reward": 0.20050686,
      "deposit": 157.69163767,
      "balance": 158.18228413,
      "total_deposit": "254.6243",
      "total_withdrawal": "96.932662325",
      "total_reward": "0.289314919943093251"
    }
    ]
}
```

### MEV Earnings

- Large staked nodes from partners will be independently deployed according to the private pool model, and the mev income obtained by the nodes will be settled independently
- mev revenue is credited to the staking address
- The mev handling fee is credited to the partner's dedicated address, and the handling fee ratio is configurable
- Small stakings from partners, unified as the overall settlement of Coke's retail investors

#### GET [/eth2/v2/mev_reward](https://test-api.kelepool.com/eth2/v2/mev_reward?timezone=8&page_number=1&page_size=5&address=0x1ba59c6ba6fa7b14ec63fe499d649595cf3b8689)

> Request parameters:
> - `page_number`/`page_size`: page number, page size
> - `address`: user staking wallet address / partner mev fee address
> - `timezone` ：specify the time zone for the return time
> - `num2str` ：whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v2/mev_reward?timezone=8&page_number=1&page_size=5&address=0x1ba59c6ba6fa7b14ec63fe499d649595cf3b8689&num2str=1
```

> Request return value:
> - `timezone` ：timezone
> - `amount` : the amount of a single income
> - `balance` : account balance
> - `total_reward` : historical cumulative reward
> - `staked_amt` : staked amount
> - `record_type`: record type (reward:reward record withdrawal:withdrawal record)
> - `height` mev reward block height
> - `mev_addr`: node mev receiving address
> - `trx_id`: transaction id (mev reward/withdrawal)
> - `time` : settlement time

```json
{
    "code":0,
    "message":"success",
    "data":{
        "total":1428,
        "page_size":1,
        "page_number":1,
        "timezone":"8",
        "data":[
            {
                "amount":"0.03249061",
                "balance":"40.32607236",
                "total_reward":"40.32607236",
                "staked_amt":"96.00000000",
                "record_type":"reward",
                "height":16612641,
                "mev_addr":"0x4675c7e5baafbffbca748158becba61ef3b0a263",
                "trx_id":"0x3de7acf868ee76a82a9e70c8d8d6c30f57b1a13d2967b1dfb365d5d1dc1870c3",
                "time":"2023-02-12 20:24:33"
            }
        ]
    }
}
```

## Validator Node Status
#### GET [/eth2/v2/miner/validator/query](https://test-api.kelepool.com/eth2/v2/miner/validator/query?address=0x5dd3bd08cbc8498c8640abc26d19480219bb0606)

Only large staking node records can be queried, and small staking nodes will not be returned

> Request parameters:
> - `address` User wallet address
> - `page_size` Page Size
> - `page_number` Page Number
> - `num2str` ：whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v2/miner/validator/query?address=0x5dd3bd08cbc8498c8640abc26d19480219bb0606&num2str=1
```

> Response Result:
> - `identifer` : the verification node number (only after the verification node takes effect)
> - `public_key` : validating node public key
> - `amount` : amount to stake
> - `staked_amount` ：current effective amount of staked (may have been partially unstaked)
> - `status` : node status 1: not active, 2: active, 5: exited
> - `effective_time`: effective time, format: %Y-%m-%d %H:%M:%S, null if not effective
> - `address` ETH1 deposit address
> - `deposit_credentials` : ETH2 withdrawal credentials
> - `type` : staking account type 0: small staking, 1: large staking
> - `reward` :node consensus benefits
> - `mev_reward` :node mev benefits
> - `settle`.`reward` :cumulative consensus income after deduction of fees
> - `settle`.`mev_reward` :cumulative mev income after deduction of fees
> - `settle`.`7d_reward` :consensus income for the past 7 days after deducting fees
> - `settle`.`7d_mev_reward` :mev income for the past 7 days after deducting fees
> - `apr` :estimate total APR
> - `apr_detail`.`basic` :estimate consensus APR
> - `apr_detail`.`mev` :estimate mev APR
```json
{
    "code":0,
    "message":"success",
    "page_size":0,
    "page_number":0,
    "total_count":0,
    "data":[
        {
            "identifer":0,
            "public_key":"852bf5000e370c1baa849defefc30a99c76ac1b41d2991b39e3f631bac3d11f9cbb961d3b17d5c4255137dc902dbbb6f",
            "amount":0.07,
            "staked_amount":"0.07",
            "status":1,
            "effective_time":null,
            "address":"0x5dd3bd08cbc8498c8640abc26d19480219bb0606",
            "deposit_credentials":"",
            "type":0,
            "reward": 0.5368926599999995,
            "mev_reward":0.5368926599999995,
            "apr":0.0487,
            "apr_detail":{
                "basic":0.0367,
                "mev":0.012
            },
            "settle":{
                "reward":"0.123",
                "mev_reward":"0.123",
                "7d_reward":"0.123",
                "7d_mev_reward":"0.123",
            }
        },
        {
            "identifer":118838,
            "public_key":"8333ce3b794a6a4fd5045f2853884aef34f1a9a3aaf4dcf09af474e67d01865ae5e7e23f77dac7e41313d665afbe5a12",
            "amount":32,
            "status":2,
            "effective_time":"2022-06-10 13:06:59",
            "address":"0x5dd3bd08cbc8498c8640abc26d19480219bb0606",
            "deposit_credentials":"003283e7b0701bd85c8aea1fb70021571a4732ba965c0309d4ea54b4dc26707d",
            "type":1,
            "reward": 0.5368926599999995,
            "mev_reward":0.5368926599999995,
            "apr":0.0487,
            "apr_detail":{
                "basic":0.0367,
                "mev":0.012
            },
            "settle":{
                "reward":"0.123",
                "mev_reward":"0.123",
                "7d_reward":"0.123",
                "7d_mev_reward":"0.123",
            }
        },
        {
            "identifer":119856,
            "public_key":"b7701b5a7dd2ceccd7f51daef59dbc74fb2273f2682df98feedb89464b4ff07f857707378f16677e5b80ef1b6257c582",
            "amount":32,
            "status":2,
            "effective_time":"2022-06-10 13:06:59",
            "address":"0x5dd3bd08cbc8498c8640abc26d19480219bb0606",
            "deposit_credentials":"003283e7b0701bd85c8aea1fb70021571a4732ba965c0309d4ea54b4dc26707d",
            "type":1,
            "reward": 0.5368926599999995,
            "mev_reward":0.5368926599999995,
            "apr":0.0487,
            "apr_detail":{
                "basic":0.0367,
                "mev":0.012
            },
            "settle":{
                "reward":"0.123",
                "mev_reward":"0.123",
                "7d_reward":"0.123",
                "7d_mev_reward":"0.123",
            }
        }
    ]
}
```

## User Operation History

#### GET [/eth2/v4/op_history](https://test-api.kelepool.com/eth2/v4/op_history?address=0xd8f8799bc41b9eb55b5c22c6f75e54b5b98f6f87&op_type=1,2,3,4)

> Request parameters:
> - `address` : User wallet address
> - `op_type` : query record type，default:1,2,3,4; 1: stake 2: unstake 3: withdrawal 4:on chain node automatic transfer
> - `op_id` ：operation id, default to empty. can be used to filter and query the on-chain transaction id for withdrawal operations
> - `page_size` : Page Size
> - `page_number` : Page Number
> - `num2str` : whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v4/op_history?address=0xd8f8799bc41b9eb55b5c22c6f75e54b5b98f6f87&op_type=0,1,2,3,4,5,6,7,8&num2str=1
```


> Response Result:
> - `transaction_id` : Transaction Hash
> - `amount` : Amount(ETH)
> - `op_type` : opertion type
> - `op_id` : opertion id
> - `history_time` : operation time

```json
{
    "code":0,
    "message":"success",
    "data": {
        "total":30,
        "page_size":20,
        "page_number":1,
        "data":[
            {
                "transaction_id":"0x2090670ba4810ebd4683e98dee19a26128c1e5263c6e9cf7ea637cf1a006b28f",
                "amount":0.01,
                "op_type":0,
                "op_id":"0bc9a32803054b5a8c6138c3df2bc959",
                "history_time":"2023-03-22 06:49:33"
            }
        ]
    }
}
```

#### GET [/eth2/v3/op_history](https://test-api.kelepool.com/eth2/v3/op_history?address=0xd8f8799bc41b9eb55b5c22c6f75e54b5b98f6f87&op_type=0,1,2,3,4,5,6)

> Request parameters:
> - `address` User wallet address
> - `op_type` ：query record typ，default:0,6; 0: deposit 1: stakeing 2: effective staked 3:wait unstake 4: unstakeing  5: unstaked  6: withdrawing 7:withdrawal done 8:on chain node automatic transfer
> - `page_size` Page Size
> - `page_number` Page Number
> - `num2str` ：whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v3/op_history?address=0xd8f8799bc41b9eb55b5c22c6f75e54b5b98f6f87&op_type=0,1,2,3,4,5,6,7,8&num2str=1
```

> not recommended to use v3, please use eth2/v4/op_history

> Response Result:
> - `transaction_id` : Transaction Hash
> - `amount` : Amount(ETH)
> - `op_type` : opertion type
> - `history_time` operation time

```json
{
    "code":0,
    "message":"success",
    "page_size":20,
    "page_number":1,
    "total_count":30
    "data":[
        {
            "transaction_id":"0x2090670ba4810ebd4683e98dee19a26128c1e5263c6e9cf7ea637cf1a006b28f",
            "amount":0.01,
            "op_type":0,
            "history_time":"2023-03-22 06:49:33"
        }
    ]
}
```

#### GET [/eth2/v2/op_history](https://test-api.kelepool.com/eth2/v2/op_history?address=0x5dd3bd08cbc8498c8640abc26d19480219bb0606)

> Request parameters:
> - `address` User wallet address
> - `num2str` ：whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v2/op_history?address=0x5dd3bd08cbc8498c8640abc26d19480219bb0606&num2str=1
```

> Response Result:
> - `address` User wallet address
> - `transaction_id` : Transaction Hash
> - `amount` : Amount to stake (ETH)
> - `type` : this field is not used
> - `status` : this field is not used
> - `history_time` operation time
> - `unactive_amount` : Amount to take effect (ETH)
> - `active_amount` : Active amount (ETH)
```json
{
    "code":0,
    "message":"success",
    "data":[
        {
            "address":"0x5dd3bd08cbc8498c8640abc26d19480219bb0606",
            "transaction_id":"0x03eae6e5048b53d867ba26147940255ebdd1f3488020885ff0a9929460a599e5",
            "amount":0.01,
            "type":0,
            "status":0,
            "history_time":"2022-06-10 10:23:59",
            "unactive_amount":0,
            "active_amount":0.01
        },
        {
            "address":"0x5dd3bd08cbc8498c8640abc26d19480219bb0606",
            "transaction_id":"0xb3bb11cbd3cbb85d49e5920c719bfe0ff2cc3574292dc5e79117b3071ca78453",
            "amount":32,
            "type":1,
            "status":0,
            "history_time":"2022-06-10 10:23:51",
            "unactive_amount":0,
            "active_amount":32
        }
    ]
}
```

## User Unstake

### Query the list of redeemable nodes

##### GET [/eth2/v2/miner/unstake_check](https://test-api.kelepool.com/eth2/v2/miner/unstake_check?address=0x3ef51B5079021a11b1CAB3d36eEa45FaCF2B00CE&unstake_amt=0&node_ids=468230,468231)


> Request parameters:
> - `address` : user address
> - `unstake_amt`: Redeem the amount of ETH, the system selects nodes to redeem according to the amount in descending order of time (this field and node_ids can be filled in for redemption by amount)
> - `node_ids`: A list of IDs on the redemption node chain, separated by multiple commas (this field and unstake_amt are optional to fill in, for redemption by node ID)
> - `num2str`: whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v2/miner/unstake_check?address=0x3ef51B5079021a11b1CAB3d36eEa45FaCF2B00CE&unstake_amt=0&node_ids=468106,468105,464352,468230
```

> Request return value: availables is the node that can be redeemed at present, and unusables is the node that cannot be redeemed temporarily (node activated within 1 day).

> - `code`: integer number, equal to 0 means success, greater than 0 means failure
> - `message` : the message to return on failure
> - `identifer` : validator on-chain ID
> - `public_key` : validator public key

```json
{
     "code": 0,
     "message": "success",
     "data": {
         "available": [
             {
                 "identifer": 468106,
                 "public_key": "893c775be276f3b908a5bc7c06d82119947ea15223738d61222d29d491d0dbc826544b1989bb41834a2ed28112052d32"
             },
             {
                 "identifer": 468105,
                 "public_key": "b945c815c0151966a3da434298b8634be71d0015064acefa84f7900bbd87a2eb42d404ea9550bca65d5d7ac4692224fb"
             },
             {
                 "identifer": 464352,
                 "public_key": "a6b53f3fb8c35a4b8ebb0fd4046dc5235655fde408222bb3feab7b81432e11e0766abe0abfd1f3b0017e08da75b59017"
             }
         ],
         "unusables": [
             {
                 "identifer": 468230,
                 "public_key": "a432e7d747543b9d646c9e5aea05a8681092c24549cc17743e283f4dd4f7b667754212b5c14399f13784bef4f5b65abc"
             }
         ]
     }
}
```

### Query the amount that can be unstake

#### GET [/eth2/v2/miner/unstake](https://test-api.kelepool.com/eth2/v2/miner/unstake?address=0xd8f8799bc41b9eb55b5c22c6f75e54b5b98f6f87)


> Request parameters：
> - `address` ：User wallet address
> - `num2str` ：whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v2/miner/unstake?address=0xd8f8799bc41b9eb55b5c22c6f75e54b5b98f6f87&num2str=1
```

> Response Result:
> - `code` : an integer number, equal to 0 for success, greater than 0 for failure
> - `message` : the message to return after failure
> - `retail_staked` ：amount of retail staked（ETH）
> - `retail_unstaking` ：amount of retail unstaking（ETH）
> - `whale_staked` ：amount of whale staked（ETH）
> - `whale_unstaking` ：amount of whale unstaking（ETH）
> - `estimate_use_sec` ：Estimated unstaking time, seconds
> - `fast_fee_ratio` ：Quick unstak fee 5%

```json
{
    "code":0,
    "message":"success",
    "data":{
        "retail_staked":"0.123",
        "retail_unstaking":"0.123",
        "whale_staked":"0.123",
        "whale_unstaking":"0.123",
        "estimate_use_sec":1234,
        "fast_fee_ratio":0.05,
  }
}
```

### User unstake 

- Step 1: user private key signature, see the eth private key signature section
- Step 2: sign the entire json body with an auth token

#### POST [/eth2/v2/miner/unstake](https://test-api.kelepool.com/eth2/v2/miner/unstake)

> - Request parameters
> - `type` ：unstake type;  retail:retail staked; retail_fast:no need to wait,but has fee; whale:whale staked
> - `address` ：User wallet address
> - `unstake_amt` ：unstake amount

```bash
https://test-api.kelepool.com/eth2/v2/miner/unstake

{
    "type":"retail",
    "address":"0xd8f8799bc41b9eb55b5c22c6f75e54b5b98f6f87",
    "unstake_amt":"123.3244"
}
```

> Response Result:
> - `code` : an integer number, equal to 0 for success, greater than 0 for failure
> - `message` : the message to return after failure
> - `withdrawable` : withdrawable amount

```json
{
    "code":0,
    "message":"success",
    "data":{
        "withdrawable":"123.3244"
    }
}
```

## User Withdrawal

### Query the amount that can be withdrawal

#### GET [/eth2/v2/miner/withdrawal](https://test-api.kelepool.com/eth2/v2/miner/withdrawal?address=0xd8f8799bc41b9eb55b5c22c6f75e54b5b98f6f87)


> Request parameters：
> - `address` ：User wallet address

```bash
https://test-api.kelepool.com/eth2/v2/miner/withdrawal?address=0xd8f8799bc41b9eb55b5c22c6f75e54b5b98f6f87
```

> Response Result:
> - `code` : an integer number, equal to 0 for success, greater than 0 for failure
> - `message` : the message to return after failure
> - `balance` ：amount that can be withdrawn
> - `user_fee` ：estimated on-chain tx fee
> - `fee_free_threshold` ：minimum withdrawal amount for exemption from tx fees
> - `pay_addr` : user wallet address

```json
{
    "code":0,
    "message":"success",
    "data":{
        "balance":"123.248",
        "user_fee":"0.12",
        "fee_free_threshold": "0.1",
        "pay_addr": "0xd8f8799bc41b9eb55b5c22c6f75e54b5b98f6f87",
    }
}
```

### user withdrawal

#### POST [/eth2/v2/miner/withdrawal](https://test-api.kelepool.com/eth2/v2/miner/withdrawal)

> Request parameters：
> - `address` ：User wallet address
> - `amount` ：withdrawal amount

```bash
https://test-api.kelepool.com/eth2/v2/miner/withdrawal

{
    "address":"0xd8f8799bc41b9eb55b5c22c6f75e54b5b98f6f87",
    "amount":"12.23",
}
```

> Response Result:
> - `code` : an integer number, equal to 0 for success, greater than 0 for failure
> - `message` : the message to return after failure
> - `op_id` ：withdrawal operation id, which can be query trx_id by eth2/v4/op_history

```json
{
    "code":0,
    "message":"success",
    "data":{
        "op_id":"ef3b5866c3f146f18c5b93e1a51a0506"
    }
}
```


## Generate Validator Public Key
#### POST [/eth2/v2/validator/keypair](https://test-api.kelepool.com/eth2/v2/validator/keypair)

> Request parameters:
> - `deposit_credentials` : User withdrawal credentials
> - `count` : Generate the number of validating nodes. When staking in batches, the number of `count` parameters can be obtained according to `staking quantity / 32`.
> - `recreate` ：Whether to regenerate a new keystore. (0=no, 1=yes)

```bash
https://test-api.kelepool.com/eth2/v2/validator/keypair

{
    "deposit_credentials":"001ae74d19004b360d02d411795cee1451dc20679f13a13aafce7de2448b60cb",
    "count":2,
    "recreate":0
}
```

> Response Result:
> - `code` : an integer number, equal to 0 for success, greater than 0 for failure
> - `message` : the message to return after failure
> - `pubkey` : validator public key
> - `withdrawal_credentials` : Withdrawal Credentials
> - `signature` : Validator signature
> - `deposit_data_root` : Merkle tree root
> - `network_name` : ETH network name
```json
{
    "code":0,
    "message":"success",
    "data":[
        {
            "pubkey":"86ee4eecf1c83725020cf8667c555b286b54445691da44aa7a671b6d18abf118452e60876216f9adec5e64ff09c3e231",
            "withdrawal_credentials":"001ae74d19004b360d02d411795cee1451dc20679f13a13aafce7de2448b60cb",
            "signature":"a61e5ed96b5b22ec9da92cf3f09c24cf9230ec1db99918e9dedfc9440de473f64b7520b5fb40558d0bc9f009dd20731917c3dbf6b3cfd98b48377a190d9e2959df3d2fa2dcec9c09e8be420accc9daa25301d4a2ce1636a5413ac066e7a4628f",
            "deposit_data_root":"ebb84a75e241501cc64c4e42dd3cdb7a2f72e6af60ab828b2fb246905eb629e5",
            "network_name":"Goerli"
        },
        {
            "pubkey":"83909737754d15dd3ad1281a3f0e62baa64d3c0abb3ed218c3baf7ff250058a24fe1143a5243c3b015e3f93ed6af1e18",
            "withdrawal_credentials":"001ae74d19004b360d02d411795cee1451dc20679f13a13aafce7de2448b60cb",
            "signature":"b95af475d67e8438e49cfaad12dacd789c705938fd6a8fee93a1a170ef6322c2cf37c643d1d010b23734c04e9028b58d034435dd6c9f19610090bfdefb7522c69e99b0a7830f6d967f1d07e3ff30128c8b516d40232e5595ac91d746420da993",
            "deposit_data_root":"f08ca526395300d60ccc6db28d931ba129944f44d4bb92c773424e120dde222b",
            "network_name":"Goerli"
        }
    ]
}
```



## Query the public key of the verifier
#### GET [/eth2/v2/validator/keypair](https://test-api.kelepool.com/eth2/v2/validator/keypair?deposit_credentials=001ae74d19004b360d02d411795cee1451dc20679f13a13aafce7de2448b60cb&is_us)

> Request parameters:
> - `deposit_credentials`: User withdrawal credentials
> - `is_used` : usage status (0=not used, 1=used)

```bash

https://test-api.kelepool.com/eth2/v2/validator/keypair?deposit_credentials=001ae74d19004b360d02d411795cee1451dc20679f13a13aafce7de2448b60cb&is_used=0

```

> Request return value:
> - `code`: integer number, equal to 0 means success, greater than 0 means failure
> - `message` : the message to return on failure
> - `pubkey`: validator public key
> - `withdrawal_credentials` : withdrawal credentials
> - `signature` : the verifier signature
> - `deposit_data_root` : Merkle root
> - `network_name`: ETH network name
> - `create_time` : creation time
```json
{
    "code":0,
    "message":"success",
    "data":[
        {
            "pubkey":"86ee4eecf1c83725020cf8667c555b286b54445691da44aa7a671b6d18abf118452e60876216f9adec5e64ff09c3e231",
            "withdrawal_credentials":"001ae74d19004b360d02d411795cee1451dc20679f13a13aafce7de2448b60cb",
            "signature":"a61e5ed96b5b22ec9da92cf3f09c24cf9230ec1db99918e9dedfc9440de473f64b7520b5fb40558d0bc9f009dd20731917c3dbf6b3cfd98b48377a190d9e2959df3d2fa2dcec9c09e8be420accc9daa25301d4a2ce1636a5413ac066e7a4628f",
            "deposit_data_root":"ebb84a75e241501cc64c4e42dd3cdb7a2f72e6af60ab828b2fb246905eb629e5",
            "network_name":"Goerli",
            "create_time":"2022-06-02 17:52:50"
        },
        {
            "pubkey":"83909737754d15dd3ad1281a3f0e62baa64d3c0abb3ed218c3baf7ff250058a24fe1143a5243c3b015e3f93ed6af1e18",
            "withdrawal_credentials":"001ae74d19004b360d02d411795cee1451dc20679f13a13aafce7de2448b60cb",
            "signature":"b95af475d67e8438e49cfaad12dacd789c705938fd6a8fee93a1a170ef6322c2cf37c643d1d010b23734c04e9028b58d034435dd6c9f19610090bfdefb7522c69e99b0a7830f6d967f1d07e3ff30128c8b516d40232e5595ac91d746420da993",
            "deposit_data_root":"f08ca526395300d60ccc6db28d931ba129944f44d4bb92c773424e120dde222b",
            "network_name":"Goerli",
            "create_time":"2022-06-02 17:52:50"
        }
    ]
}
```


## Partner staking overview
#### GET [/eth2/v2/partner/dashboard](https://test-api.kelepool.com/eth2/v2/partner/dashboard)

> Request parameters:
> - `num2str` ：whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v2/partner/dashboard?num2str=1
```

> Request return value:
> - `total_amount`: total amount of stake (ETH)
> - `staked_amount`: staked amount (ETH)
> - `staking_amount`: Amount to take effect (ETH)
> - `ongoing_amount` : amount to be withdrawn (ETH)
> - `total_reward` : total reward (ETH)
> - `total_validaters` : total number of validators
> - `unactive_validater`: the number of nodes to be validated
> - `actived_validater`: the number of active nodes
> - `closed_validater` : number of closed nodes
```json
{
    "code":0,
    "message":"success",
    "data":{
        "staking":{
            "total_amount":173.3,
            "staked_amount":173.23,
            "staking_amount":0.07,
            "ongoing_amount":0,
            "total_reward":0.82885946,
        },
        "validater":{
            "total_validaters":8,
            "unactived_validater":1,
            "actived_validater":7,
            "closed_validater":0
        }
    }
}
```


## Partner reward history list
#### GET [/eth2/v2/partner/income](https://test-api.kelepool.com/eth2/v2/partner/income)

> Request parameters:
> - `num2str` ：whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v2/partner/income?num2str=1
```

> Request return value:
> - `date` : dividend date
> - `reward` : accumulated earnings as of the current day
> - `deposit` : the cumulative recharge principal as of the current day
> - `balance`: the total balance of the account as of the day (accumulated recharge principal as of the day + accumulated income as of the day)
```json
{
    "code":0,
    "message":"success",
    "data":[
        {
            "date":"2022-07-09 00:00:00",
            "reward":0.0172946,
            "deposit":173.3,
            "balance":174.12885933
        },
        {
            "date":"2022-07-08 00:00:00",
            "reward":0.03071118,
            "deposit":173.3,
            "balance":174.11156473
        }
    ]
}
```


## List of partner verification nodes
#### GET [/eth2/v2/partner/validator](https://test-api.kelepool.com/eth2/v2/partner/validator)

> Request parameters:
> - `page_size` Page Size
> - `page_number` Page Number
> - `num2str` ：whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v2/partner/validator?num2str=1
```

> Request return value:
> - `identifer`: validator ID (only after the validator takes effect)
> - `public_key` : validator public key
> - `amount` : the staked amount
> - `status` : 0: pending 1: staking, 2: effective, 3: exiting, 4: withdrawing, 5: withdrawn
> - `effective_time`: effective time, format: %Y-%m-%d %H:%M:%S, null if not effective
> - `address` ETH1 deposit address
> - `deposit_credentials`: ETH2 withdrawal credentials
> - `type`: staking account type 0: small staking, 1: large staking
> - `reward` :node consensus benefits
> - `mev_reward` :node mev benefits
> - `settle`.`reward` :cumulative consensus income after deduction of fees
> - `settle`.`mev_reward` :cumulative mev income after deduction of fees
> - `settle`.`7d_reward` :consensus income for the past 7 days after deducting fees
> - `settle`.`7d_mev_reward` :mev income for the past 7 days after deducting fees
> - `apr` : estimate total APR
> - `apr_detail`.`basic`:estimate consensus APR
> - `apr_detail`.`mev`:estimate mev APR

```json
{
    "code":0,
    "message":"success",
    "page_size":0,
    "page_number":0,
    "total_count":0,
    "data":[
        {
            "identifer":0,
            "public_key":"852bf5000e370c1baa849defefc30a99c76ac1b41d2991b39e3f631bac3d11f9cbb961d3b17d5c4255137dc902dbbb6f",
            "amount":0.07,
            "status":1,
            "effective_time":null,
            "address":"0x5dd3bd08cbc8498c8640abc26d19480219bb0606",
            "deposit_credentials":"",
            "type":0,
            "reward": 0.5368926599999995,
            "mev_reward":0.5368926599999995,
            "apr":0.0487,
            "apr_detail":{
                "basic":0.0367,
                "mev":0.012
            },
            "settle":{
                "reward":"0.123",
                "mev_reward":"0.123",
                "7d_reward":"0.123",
                "7d_mev_reward":"0.123",
            }
        },
        {
            "identifer":118838,
            "public_key":"8333ce3b794a6a4fd5045f2853884aef34f1a9a3aaf4dcf09af474e67d01865ae5e7e23f77dac7e41313d665afbe5a12",
            "amount":32,
            "status":2,
            "effective_time":"2022-06-10 13:06:59",
            "address":"0x5dd3bd08cbc8498c8640abc26d19480219bb0606",
            "deposit_credentials":"003283e7b0701bd85c8aea1fb70021571a4732ba965c0309d4ea54b4dc26707d",
            "type":1,
            "reward": 0.5368926599999995,
            "mev_reward":0.5368926599999995,
            "apr":0.0487,
            "apr_detail":{
                "basic":0.0367,
                "mev":0.012
            },
            "settle":{
                "reward":"0.123",
                "mev_reward":"0.123",
                "7d_reward":"0.123",
                "7d_mev_reward":"0.123",
            }
        },
        {
            "identifer":119856,
            "public_key":"b7701b5a7dd2ceccd7f51daef59dbc74fb2273f2682df98feedb89464b4ff07f857707378f16677e5b80ef1b6257c582",
            "amount":32,
            "status":2,
            "effective_time":"2022-06-10 13:06:59",
            "address":"0x5dd3bd08cbc8498c8640abc26d19480219bb0606",
            "deposit_credentials":"003283e7b0701bd85c8aea1fb70021571a4732ba965c0309d4ea54b4dc26707d",
            "type":1,
            "reward": 0.5368926599999995,
            "mev_reward":0.5368926599999995,
            "apr":0.0487,
            "apr_detail":{
                "basic":0.0367,
                "mev":0.012
            },
            "settle":{
                "reward":"0.123",
                "mev_reward":"0.123",
                "7d_reward":"0.123",
                "7d_mev_reward":"0.123",
            }
        }
    ]
}
```


## Node reward line chart
#### GET [/eth2/v2/validator_reward](https://test-api.kelepool.com/eth2/v2/validator_reward?page_number=1&page_size=20&timezone=8&unit=day&pubkey=8d9f04df4879680625ce6f3b9df0536160bb706e4242abc317ae53903abb804a5f26390ee4b739eacaecf8776bd0d0ce)

> Request parameters:
> - `pubkey` : validator public key
> - `timezone`: the time zone
> - `unit`: statistical unit (day/hour)
> - `num2str` ：whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v2/validator_reward?page_number=1&page_size=20&timezone=8&unit=day&pubkey=8d9f04df4879680625ce6f3b9df0536160bb706e4242abc317aeec53903abb804acef4d8eed7ea&num2str=1
```

> Request return value:
> - [recording period, cumulative total node rewards, staking amount, total node balance]

```json
{
    "code":0,
    "message":"success",
    "data":{
        "total":3,
        "page_size":5,
        "page_number":1,
        "data":[
            [
                "2023-02-08",
                "0.00639918",
                "32.00",
                "32.00639918"
            ]
        ]
    }
}
```

### Node chain automatic transfer record query

> large amount pledge nodes, whose basic income extraction/principal redemption are automatically transferred by the node, and transfer records can be queried through this interface
##### GET [/eth2/v2/validator/node_withdrawal](https://test-api.kelepool.com/eth2/v2/validator/node_withdrawal?vids=464352,468105&address=0x3ef51B5079021a11b1CAB3d36eEa45FaCF2B00CE&order_by=-time&page_size=5&page_number=1)

> Request parameters：
> - `page_size` Page Size
> - `page_number` Page Number
> - `vids` ：node id filter, can transmit multiple, recommended not to exceed 10
> - `address` ：staking user filter (vids and address must be at least one valid, and the query results take the intersection of the two)
> - `order_by` ：sort transfer records, currently supported:`time`,`-time`
> - `timezone` ：specify the time zone for the return time

```bash
https://test-api.kelepool.com/eth2/v2/validator/node_withdrawal?timezone=0&vids=464352,468105&address=0x3ef51B5079021a11b1CAB3d36eEa45FaCF2B00CE&order_by=-time&page_size=5&page_number=1
```

> Request return value：
> - `timezone` ：timezone
> - `index` ：unique index number for transfer records on the chain
> - `amount` ：transfer amount(gwei)
> - `amount_eth` ：transfer amount(eth)
> - `address` ：transfer recipient address
> - `time` ：transfer time
> - `validator_index` ：node unique ID

```json
{
    "code":0,
    "message":"success",
    "data":{
        "total":21,
        "page_size":3,
        "page_number":1,
        "timezone":"8",
        "data":[
            {
                "index":"0x329de0",
                "amount":32000000000,
                "address":"0x3ef51b5079021a11b1cab3d36eea45facf2b00ce",
                "time":"2023-04-21 08:47:36",
                "validator_index":468105,
                "amount_eth":"32"
            },
            {
                "index":"0x328ffd",
                "amount":32000000000,
                "address":"0x3ef51b5079021a11b1cab3d36eea45facf2b00ce",
                "time":"2023-04-21 07:51:00",
                "validator_index":464352,
                "amount_eth":"32"
            },
            {
                "index":"0x303d39",
                "amount":1371538,
                "address":"0x3ef51b5079021a11b1cab3d36eea45facf2b00ce",
                "time":"2023-04-19 14:17:00",
                "validator_index":468105,
                "amount_eth":"0.001371538"
            }
        ]
    }
}
```

## Node penalty record
#### GET [/eth2/v2/slashes/history](https://test-api.kelepool.com/eth2/v2/slashes/history?page_number=1&page_size=2&pubkey=8d9f04df4879680625ce6f3b9df0536160bb706e4242abc317ae53903abb804a5f26390ee4b739eacaecf8776bd0d0ce)

> Request parameters:
> - `pubkey` : validator public key
> - `num2str` ：whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v2/slashes/history?page_number=1&page_size=20&pubkey=8d9f04df4879680625ce6f3b9df0536160bb706e4242abc317ae53903abb804a5f26390ee4b879eabecdf&num2str=1
```

> Request return value:
> - `epoch`: node period
> - `slash_amount` : the penalty amount
> - `snap_time` : the cycle time

```json
{
    "code":0,
    "message":"success",
    "data":{
        "total":0,
        "page_size":1,
        "page_number":2,
        "data":[
            {
                "epoch":27551,
                "slash_amount":"0.00240799",
                "snap_time":"2023-02-09 11:40:16"
            }
        ]
    }
}
```


## Batch query of validator status
##### GET [/eth2/v2/validators](https://test-api.kelepool.com/eth2/v2/validators?vids=460009,459869&pubkeys=a1e60756f5a0fe6aaed34f6fb85f1cc3b3d823a11d000febfde2f8982c2a398324dd36f8fee97fde9e63a8fde131e630)

> Request parameters:
> - `pubkeys` ：public key list, comma separated
> - `vids` ：validator id filtering, separated by commas
> - `page_size` Page Size
> - `page_number` Page Number

```bash
https://test-api.kelepool.com/eth2/v2/validators?vids=460009,459869&pubkeys=a1e60756f5a0fe6aaed34f6fb85f1cc3b3d823a11d000febfde2f8982c2a398324dd36f8fee97fde9e63a8fde131e630
```

> Request return value:
> - `identifer` ：validator id
> - `public_key` ：public key
> - `chain_status` ：status on chain
> - `status` ：validator status 0-pending 1-staking 2-effective 3-exiting 4-withdrawing 5-withdrawn_done
> - `effective_ts` ：effective time, 0 is an invalid value
> - `exiting_ts` ：start exit time, 0 is an invalid value
> - `exited_ts` ：exit work time，0 is an invalid value
> - `withdrawal_done_ts` ：withdrawal done time, exit completely，0 is an invalid value

```json
{
    "code":0,
    "message":"success",
    "data":{
        "total":3,
        "page_size":20,
        "page_number":1,
        "data":[
            {
                "identifer":459869,
                "public_key":"90604806e530b73ad9c2d1949b5d241098293d2d5ca73d9c9da0f3d581f8dd5f9121ec9a028aa718c48e0106f8944186",
                "status":2,
                "chain_status":"active_ongoing",
                "effective_ts":1679368172,
                "pending_queued_ts":0,
                "exiting_ts":0,
                "exited_ts":0,
                "withdrawal_done_ts":0
            },
            {
                "identifer":460009,
                "public_key":"9296eeffaca8103a5279ea9e1ef0694e28e5c90df845a8e3218b6e83706e912f178269ea206943f2ef3695b5a13525c9",
                "status":5,
                "chain_status":"withdrawal_done",
                "effective_ts":1679517069,
                "pending_queued_ts":0,
                "exiting_ts":222,
                "exited_ts":444,
                "withdrawal_done_ts":5555
            },
            {
                "identifer":459868,
                "public_key":"a1e60756f5a0fe6aaed34f6fb85f1cc3b3d823a11d000febfde2f8982c2a398324dd36f8fee97fde9e63a8fde131e630",
                "status":2,
                "chain_status":"active_ongoing",
                "effective_ts":1679343856,
                "pending_queued_ts":0,
                "exiting_ts":0,
                "exited_ts":0,
                "withdrawal_done_ts":0
            }
        ]
    }
}
```

### Quick pledge interface for partners 
##### GET [/eth2/v2/miner/fund/fast_stake](https://test-api.kelepool.com/eth2/v2/miner/fund/fast_stake)

In order to allow users to take effect quickly after staking/redemption (currently staking takes about 38 days), and improve the utilization rate of funds, we have added a system advance account. Users can directly transfer ETH from the wallet to the system advance address, and only need to wait for 64 blocks (about 13 minutes) to obtain the pledge income on the chain. The principle of fast redemption is the same. You only need to pay 1% of the redemption funds as a handling fee, and you can arrive immediately without waiting for the long time for redemption by nodes on the chain.

The third party can contact us to configure an independent fast deposit address. If no independent deposit address is configured, this interface will return the Coke Pool deposit address by default.

```
Fast pledge:
- The user transfers a certain amount of ETH to the system advance address returned by this interface
- If the balance of the advance address is sufficient, the system will automatically transfer the funds of the advance address to the user's account, and it will take effect after 64 blocks on the chain are confirmed
- If the balance of the deposit address is insufficient, the system will automatically deposit the insufficient part into the withdrawable balance under the user account, which can be withdrawn directly

Quick pledge example: Suppose there is 100ETH effective funds under the advance address, the user transfers 200ETH from the wallet to the advance address, the 100ETH under the advance address is transferred to the user, and the other 100ETH is deposited into the user's withdrawal balance.

Quick redemption:
- The user pays 1% of the redemption amount as a handling fee, and the pledged funds can be redeemed immediately
- If the deposit address has enough funds, the system will automatically transfer the effective funds under the user address to the deposit address, and transfer the user's redemption funds to the cashable
- If the deposit address has insufficient funds, the user will be prompted that the current deposit address balance is insufficient and cannot be redeemed quickly

Example of quick redemption: Assume that the initial maximum fund of the deposit address is set to 200ETH, user A redeems 100ETH, and there is only 100ETH left under the deposit address, and user B redeems 200, which prompts that the balance is insufficient.
```

> Request parameters:
> - `num2str`: whether to convert all returned fields to string type

```bash
https://test-api.kelepool.com/eth2/v2/miner/fund/fast_stake
```

> Request return value:
> - `fund_addr`: System advance fund address
> - `stake_fee`: fast pledge fee, if it is 0, no fee will be charged
> - `init_max_eth`: the maximum redeemable funds (ETH) under the deposit address
> - `fast_stake_balance` : current available funds (ETH) when user fast stakes
> - `fast_unstake_balance`: current available funds (ETH) when the user fast redeems
> - `fast_stake_pending`: When the user fast stakes, the funds under the advance address are waiting for 64 blocks to be confirmed
```json
{
   "code": 0,
   "message": "success",
   "data": {
     "fund_addr": "0x5dd3bd08cbc8498c8640abc26d19480219bb0606",
     "stake_fee": "0",
     "init_max_eth": "10",
     "fast_stake_balance": "0",
     "fast_unstake_balance": "10",
     "fast_stake_pending": "0"
   }
}

## Set partner fee and payment address

1. Partners can contact Coke Mining Pool to set up large pledge procedures, channel marks, payment address, fee type, etc. After the pledge is completed, the contract will automatically transfer the handling fee to the payment address set by the partner. Kele Pool currently charges 0.05ETH as a handling fee for 32ETH staking. 


- If the source when the user pledges matches the partner channel flag set by the partner, the contract will require the user to pay the partner fee

- Partners can query their own handling fee information through the contract's getPartnerInfo

- The partner does not set the handling fee or the handling fee is set to 0. By default, each node charges 0.05 handling fee


2. There are two ways to collect the handling fee (take the user as a pledge of 10 verification nodes at a time, and the partner sets a handling fee of 0.1ETH as an example)

- Charged according to the number of nodes: the contract will charge 1.5ETH as a handling fee, of which 0.5ETH will be given to the Coke mining pool, and 1ETH will be automatically transferred to the partner

- Charged per pledge: the contract will charge 0.6ETH as a handling fee, of which 0.5ETH will be given to the Coke mining pool, and 0.1ETH will be automatically transferred to the partner