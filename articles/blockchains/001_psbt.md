### Chương này nói về PSBT

##### 1. PSBT là gì?
-  Một PSBT là một transaction rằng nó không được ký.
- Có các Role sau:
```
- Creator
- Updaters
- Signers
- Finalizer
- Extracter
- Combiner
```

- Một Creator đề nghị một transaction
- Một hoặc nhiều Updaters bổ sung nó
- Một hoặc nhiều Signers chứng thực nó
- Trước khi một Finalizer hoàn thành nó
- Và một Extracter biến nó thành một giao dịch cho mạng Bitcoin
- Có thể có một Combiner sẽ kết hợp các PSBT song song từ những users khác.

##### 2. Cách tạo một PSBT từ theo cách cũ

###### PSBT Role: Creator
- Sử dụng converttopsbt - đây không phải cách tốt nhất nữa.

```
$ utxo_txid_1=$(bitcoin-cli listunspent | jq -r '.[0] | .txid')
$ utxo_vout_1=$(bitcoin-cli listunspent | jq -r '.[0] | .vout')
$ utxo_txid_2=$(bitcoin-cli listunspent | jq -r '.[1] | .txid')
$ utxo_vout_2=$(bitcoin-cli listunspent | jq -r '.[1] | .vout')
$ echo $utxo_txid_1 $utxo_vout_1 $utxo_txid_2 $utxo_vout_2
c6de60427b28d8ec8102e49771e5d0348fc3ef6a5bf02eb864ec745105a6951b 1 8748eff5f12ca886e3603d9e30227dcb3f0332e0706c4322fec96001f7c7f41c 0
$ recipient=tb1qcaedd724gts3aug73m78c7nfsv9d8zs9q6h2kd
$ rawtxhex=$(bitcoin-cli -named createrawtransaction inputs='''[ { "txid": "'$utxo_txid_1'", "vout": '$utxo_vout_1' }, { "txid": "'$utxo_txid_2'", "vout": '$utxo_vout_2' } ]''' outputs='''{ "'$recipient'": 0.0000065 }''')
```

Sau đó convert:
```
$ psbt=$(bitcoin-cli -named converttopsbt hexstring=$rawtxhex)
$ echo $psbt
cHNidP8BAHsCAAAAAhuVpgVRdOxkuC7wW2rvw4800OVxl+QCgezYKHtCYN7GAQAAAAD/////HPTH9wFgyf4iQ2xw4DIDP8t9IjCePWDjhqgs8fXvSIcAAAAAAP////8BigIAAAAAAAAWABTHctb5VULhHvEejvx8emmDCtOKBQAAAAAAAAAA
```


##### 3. Cách tạo một PSBT theo cách Hard Way
###### PSBT Role: Creator

Tương tự `createrawtransaction` thì cũng sẽ tạo có phương thức được gọi là: `createpsbt`

```
$ psbt_1=$(bitcoin-cli -named createpsbt inputs='''[ { "txid": "'$utxo_txid_1'", "vout": '$utxo_vout_1' }, { "txid": "'$utxo_txid_2'", "vout": '$utxo_vout_2' } ]''' outputs='''{ "'$recipient'": 0.0000065 }''')
```

Nó sẽ giống `converttopsbt`
```
$ echo $psbt_1
cHNidP8BAHsCAAAAAhuVpgVRdOxkuC7wW2rvw4800OVxl+QCgezYKHtCYN7GAQAAAAD/////HPTH9wFgyf4iQ2xw4DIDP8t9IjCePWDjhqgs8fXvSIcAAAAAAP////8BigIAAAAAAAAWABTHctb5VULhHvEejvx8emmDCtOKBQAAAAAAAAAA
$ if [ "$psbt" == "$psbt_1" ]; then     echo "PSBTs are equal"; else     echo "PSBTs are not equal"; fi
PSBTs are equal
```


##### 4. Kiểm tra một PSBT (Examine)
###### PSBT Role: Any

Decode một PSBT: sử dụng hàm: `decodepsbt`:
```
$ bitcoin-cli -named decodepsbt psbt=$psbt
{
  "tx": {
    "txid": "ea73a631b456d2b041ed73bf5767946408c6ff067716929a68ecda2e3e4de6d3",
    "hash": "ea73a631b456d2b041ed73bf5767946408c6ff067716929a68ecda2e3e4de6d3",
    "version": 2,
    "size": 123,
    "vsize": 123,
    "weight": 492,
    "locktime": 0,
    "vin": [
      {
        "txid": "c6de60427b28d8ec8102e49771e5d0348fc3ef6a5bf02eb864ec745105a6951b",
        "vout": 1,
        "scriptSig": {
          "asm": "",
          "hex": ""
        },
        "sequence": 4294967295
      },
      {
        "txid": "8748eff5f12ca886e3603d9e30227dcb3f0332e0706c4322fec96001f7c7f41c",
        "vout": 0,
        "scriptSig": {
          "asm": "",
          "hex": ""
        },
        "sequence": 4294967295
      }
    ],
    "vout": [
      {
        "value": 0.00000650,
        "n": 0,
        "scriptPubKey": {
          "asm": "0 c772d6f95542e11ef11e8efc7c7a69830ad38a05",
          "hex": "0014c772d6f95542e11ef11e8efc7c7a69830ad38a05",
          "reqSigs": 1,
          "type": "witness_v0_keyhash",
          "addresses": [
            "tb1qcaedd724gts3aug73m78c7nfsv9d8zs9q6h2kd"
          ]
        }
      }
    ]
  },
  "unknown": {
  },
  "inputs": [
    {
    },
    {
    }
  ],
  "outputs": [
    {
    }
  ]
}
```

Dùng `analyzepsbt` để kiểm tra
```
standup@btctest20:~$ bitcoin-cli -named analyzepsbt psbt=$psbt
{
  "inputs": [
    {
      "has_utxo": false,
      "is_final": false,
      "next": "updater"
    },
    {
      "has_utxo": false,
      "is_final": false,
      "next": "updater"
    }
  ],
  "next": "updater"
}
```

##### 5. Hoàn thành một PSBT (Finalize)

###### PSBT Role: Updater, Signer, Finalizer

```
Ghi chú: 
```

Nên sử dụng: `walletprocesspsbt`, nó sẽ Update, Sign, and Finalize:
```
$ bitcoin-cli walletprocesspsbt $psbt
{
  "psbt": "cHNidP8BAHsCAAAAAhuVpgVRdOxkuC7wW2rvw4800OVxl+QCgezYKHtCYN7GAQAAAAD/////HPTH9wFgyf4iQ2xw4DIDP8t9IjCePWDjhqgs8fXvSIcAAAAAAP////8BigIAAAAAAAAWABTHctb5VULhHvEejvx8emmDCtOKBQAAAAAAAQEfAQAAAAAAAAAWABRsRdOvqHYghsS9dtinGsfJduGRlgEIawJHMEQCIAqJbxz6dBzNpfaDu4XZXb+DbDkM3UWnhezh9UdmeVghAiBRxMlW2o0wEtphtUZRWIiJOaGtXfsQbB4lovkvE4eRIgEhArrDpkX9egpTfGJ6039faVBYxY0ZzrADPpE/Gpl14A3uAAEBH0gDAAAAAAAAFgAU1ZEJG4B0ojde2ZhanEsY7+z9QWUBCGsCRzBEAiB+sNNCO4xiFQ+DoHVrqqk9yM0V4H9ZSyExx1PW7RbjsgIgUeWkQ3L7aAv1xIe7h+8PZb8ECsXg1UzbtPW8wd2qx0UBIQKIO7VGPjfVUlLYs9XCFBsAezfIp9tiEfdclVrMXqMl6wAA",
  "complete": true
}
```

```
$ psbt_f=$(bitcoin-cli walletprocesspsbt $psbt | jq -r '.psbt')
```

Sau đó decode
```
$ bitcoin-cli decodepsbt $psbt_f
{
  "tx": {
    "txid": "ea73a631b456d2b041ed73bf5767946408c6ff067716929a68ecda2e3e4de6d3",
    "hash": "ea73a631b456d2b041ed73bf5767946408c6ff067716929a68ecda2e3e4de6d3",
    "version": 2,
    "size": 123,
    "vsize": 123,
    "weight": 492,
    "locktime": 0,
    "vin": [
      {
        "txid": "c6de60427b28d8ec8102e49771e5d0348fc3ef6a5bf02eb864ec745105a6951b",
        "vout": 1,
        "scriptSig": {
          "asm": "",
          "hex": ""
        },
        "sequence": 4294967295
      },
      {
        "txid": "8748eff5f12ca886e3603d9e30227dcb3f0332e0706c4322fec96001f7c7f41c",
        "vout": 0,
        "scriptSig": {
          "asm": "",
          "hex": ""
        },
        "sequence": 4294967295
      }
    ],
    "vout": [
      {
        "value": 0.00000650,
        "n": 0,
        "scriptPubKey": {
          "asm": "0 c772d6f95542e11ef11e8efc7c7a69830ad38a05",
          "hex": "0014c772d6f95542e11ef11e8efc7c7a69830ad38a05",
          "reqSigs": 1,
          "type": "witness_v0_keyhash",
          "addresses": [
            "tb1qcaedd724gts3aug73m78c7nfsv9d8zs9q6h2kd"
          ]
        }
      }
    ]
  },
  "unknown": {
  },
  "inputs": [
    {
      "witness_utxo": {
        "amount": 0.00000001,
        "scriptPubKey": {
          "asm": "0 6c45d3afa8762086c4bd76d8a71ac7c976e19196",
          "hex": "00146c45d3afa8762086c4bd76d8a71ac7c976e19196",
          "type": "witness_v0_keyhash",
          "address": "tb1qd3za8tagwcsgd39awmv2wxk8e9mwryvktqmkkg"
        }
      },
      "final_scriptwitness": [
        "304402200a896f1cfa741ccda5f683bb85d95dbf836c390cdd45a785ece1f54766795821022051c4c956da8d3012da61b5465158888939a1ad5dfb106c1e25a2f92f1387912201",
        "02bac3a645fd7a0a537c627ad37f5f695058c58d19ceb0033e913f1a9975e00dee"
      ]
    },
    {
      "witness_utxo": {
        "amount": 0.00000840,
        "scriptPubKey": {
          "asm": "0 d591091b8074a2375ed9985a9c4b18efecfd4165",
          "hex": "0014d591091b8074a2375ed9985a9c4b18efecfd4165",
          "type": "witness_v0_keyhash",
          "address": "tb1q6kgsjxuqwj3rwhkenpdfcjccalk06st9z0k0kh"
        }
      },
      "final_scriptwitness": [
        "304402207eb0d3423b8c62150f83a0756baaa93dc8cd15e07f594b2131c753d6ed16e3b2022051e5a44372fb680bf5c487bb87ef0f65bf040ac5e0d54cdbb4f5bcc1ddaac74501",
        "02883bb5463e37d55252d8b3d5c2141b007b37c8a7db6211f75c955acc5ea325eb"
      ]
    }
  ],
  "outputs": [
    {
    }
  ],
  "fee": 0.00000191
}
```

