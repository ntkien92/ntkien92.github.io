### Questions

##### 1. Transaction Fee

Khi mà sử dụng: `createrawtransaction` --> Cách tính fee như thế nào.

Fee = Total Input - Total OutPut

Các bước thực hiện:

```
$ utxo_txid_1=$(bitcoin-cli listunspent | jq -r '.[0] | .txid')
$ utxo_vout_1=$(bitcoin-cli listunspent | jq -r '.[0] | .vout')
$ utxo_txid_2=$(bitcoin-cli listunspent | jq -r '.[2] | .txid')
$ utxo_vout_2=$(bitcoin-cli listunspent | jq -r '.[2] | .vout')
```

Create Transactions:

```
$ rawtxhex2=$(bitcoin-cli -named createrawtransaction inputs='''[ { "txid": "'$utxo_txid_1'", "vout": '$utxo_vout_1' }, { "txid": "'$utxo_txid_2'", "vout": '$utxo_vout_2' } ]''' outputs='''{ "'$recipient'": 0.009, "'$changeaddress'": 0.0009 }''')
```

Sign Transaction:

```
$ signedtx2=$(bitcoin-cli -named signrawtransactionwithwallet hexstring=$rawtxhex2 | jq -r '.hex')
```

Send Transaction:

```
bitcoin-cli -named sendrawtransaction hexstring=$signedtx2
```

##### 2. Muốn Fund lại tiền mà đã seting fee cố định

Để muốn thối lại tiền thì cần sử dụng: `fundrawtransaction`

Các bước thực hiện:

Create Raw Transactions:

```
unfinishedtx=$(bitcoin-cli -named createrawtransaction inputs='''[]''' outputs='''{ "'$recipient'": 0.0002 }''')
```

Fund Raw Transaction:

```
rawtxhex3=$(bitcoin-cli -named fundrawtransaction hexstring=$unfinishedtx | jq -r '.hex')
```

DocodeRawTransaction:

```
bitcoin-cli -named decoderawtransaction hexstring=$rawtxhex3
```

Send Funded Transaction:

```
$ signedtx3=$(bitcoin-cli -named signrawtransactionwithwallet hexstring=$rawtxhex3 | jq -r '.hex')
$ bitcoin-cli -named sendrawtransaction hexstring=$signedtx3
```

##### 3. Listunspent UTXO sẽ lấy theo thứ tự nào?

- Sau khi tìm hiểu thì nó sẽ lấy amount lớn trước cho đến khi tổng lớn hơn thì sẽ không lấy nữa.

