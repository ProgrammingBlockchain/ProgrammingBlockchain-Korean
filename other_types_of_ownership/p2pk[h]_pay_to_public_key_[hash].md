## P2PK[H] \(Pay to Public Key [Hash]\) {#p2pk-h-pay-to-public-key-hash}

### P2PKH - Quick recap
우리는 **Bitcoin Address**가 **공개 키**의 **해시**임을 배웠습니다:


```cs
var publicKeyHash = new Key().PubKey.Hash;
var bitcoinAddress = publicKeyHash.GetAddress(Network.Main);
Console.WriteLine(publicKeyHash); // 41e0d7ab8af1ba5452b824116a31357dc931cf28
Console.WriteLine(bitcoinAddress); // 171LGoEKyVzgQstGwnTHVh3TFTgo5PsqiY
```  

우리는 또한 **비트코인 주소**라는 것은 없다는 것을 배웠습니다. 블록체인은 **ScriptPubKey**를 이용해 수신자를 식별하며, **ScriptPubKey**는 주소로부터 생성될 수 있습니다.

```cs
var scriptPubKey = bitcoinAddress.ScriptPubKey;
Console.WriteLine(scriptPubKey); // OP_DUP OP_HASH160 41e0d7ab8af1ba5452b824116a31357dc931cf28 OP_EQUALVERIFY OP_CHECKSIG
```  

반대도 동일합니다:  

```cs
var sameBitcoinAddress = scriptPubKey.GetDestinationAddress(Network.Main);
```

### P2PK

그러나 모든 **ScriptPubKey**가 비트코인 주소를 나타내는 것은 아닙니다. 제네시스 블록이라 불리는 모든 블록체인의 첫번째 블록의 첫번째 트랜잭션을 한 번 봅시다. 

```cs
Block genesisBlock = Network.Main.GetGenesis();
Transaction firstTransactionEver = genesisBlock.Transactions.First();
var firstOutputEver = firstTransactionEver.Outputs.First();
var firstScriptPubKeyEver = firstOutputEver.ScriptPubKey;
var firstBitcoinAddressEver = firstScriptPubKeyEver.GetDestinationAddress(Network.Main);
Console.WriteLine(firstBitcoinAddressEver == null); // True
```  

```cs
Console.WriteLine(firstTransactionEver);
```  

```json
{
…
  "out": [
    {
      "value": "50.00000000",
      "scriptPubKey": "04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f OP_CHECKSIG"
    }
  ]
}
```  

**scriptPubKey**의 형태가 다른 것을 볼 수 있습니다:  

```cs
Console.WriteLine(firstScriptPubKeyEver); // 04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f OP_CHECKSIG
```

비트코인 주소는 다음과 같이 나타내집니다: **OP_DUP OP_HASH160 &lt;hash&gt; OP_EQUALVERIFY OP_CHECKSIG**

그러나 지금 얻은 것은 다음과 같습니다:**&lt;pubkey&gt; OP_CHECKSIG**

사실, 처음에는 공개 키가 **ScriptPubKey**에 직접적으로 사용되었습니다.

```cs
var firstPubKeyEver = firstScriptPubKeyEver.GetDestinationPublicKeys().First();
Console.WriteLine(firstPubKeyEver); // 04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f
```

현재 우리는 공개 키 해시를 주로 사용합니다.


![](../assets/PPKH.png)  

```cs
key = new Key();
Console.WriteLine("Pay to public key : " + key.PubKey.ScriptPubKey);
Console.WriteLine();
Console.WriteLine("Pay to public key hash : " + key.PubKey.Hash.ScriptPubKey);
```  

``` 
Pay to public key : 02fb8021bc7dedcc2f89a67e75cee81fedb8e41d6bfa6769362132544dfdf072d4 OP_CHECKSIG
Pay to public key hash : OP_DUP OP_HASH160 0ae54d4cec828b722d8727cb70f4a6b0a88207b2 OP_EQUALVERIFY OP_CHECKSIG
```  

두 가지의 지불 방식을 **P2PK** (pay to public key), **P2PKH** (pay to public key hash)라고 지칭합니다.

사토시는 두 가지 이유로 P2PK 대신 P2PKH를 사용하기로 결정했습니다:

*   타원 곡선 암호(**public key**와 **private key**에 사용되는 암호화)는 타원 곡선에서 이산 로그 문제를 해결하기 위한 수정 쇼어 알고리즘에 취약합니다. 쉽게 말해서, 미래의 양자 컴퓨터가 공개 키로부터 개인 키를 되찾을 수 있다는 것을 의미합니다. 코인을 사용할 때만 공개 키를 사용함으로써(주소를 재사용하지 않는다고 가정하면), 이러한 공격을 막을 수 있습니다.
*   해시된 것은 더 작기에 (20바이트), QR 코드와 같은 작은 용량의 매체에 쉽게 포함시키고 인쇄할 수 있습니다.

오늘날에는 P2SH...와 함께 사용되는 것을 제외하면 더 이상 P2PK를 직접적으로 사용할 이유는 없습니다.

> ([Discussion](https://www.reddit.com/r/Bitcoin/comments/4isxjr/petition_to_protect_satoshis_coins/d30we6f)) If the issue of the early use of P2PK is not addressed it will have a serious impact on the Bitcoin price.  

### Exercise
([nopara73](https://github.com/nopara73)) While reading this chapter I found the the abbreviations (P2PK, P2PKH, P2W, etc..) very confusing.  
My trick was to force myself to pronounce the terms fully every time I encountered them during the following lessons. Suddenly everything made much more sense. I recommend you to do the same.
