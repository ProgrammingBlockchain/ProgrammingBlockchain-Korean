## P2WPKH (Pay to Witness Public Key Hash) {#p2wpkh-pay-to-witness-public-key-hash}

2015년, Pieter Wuille는 축약어인 **Segwit**(세그윗)이라고도 알려진 **Segregated Witness**라고 부르는 새로운 기능을 도입했습니다. 기본적으로, 세그윗은 소유권 증명을 트랜잭션의 **scriptSig** 부분에서 **witness**라고 불리는 새로운 입력 부분으로 옮깁니다.

새로운 방식을 사용하는 것이 유리한 몇 가지 이유가 있는데, 아래에 요약된 내용이 있습니다. 더 자세한 사항은 [링크](https://bitcoincore.org/en/2016/01/26/segwit-benefits/)를 방문하세요.

*   **제3자 가단성 수정:** 이전에는, 제3자가 트랜잭션이 승인되기 전에 트랜잭션 아이디를 변경할 수 있었습니다. 이는 세그윗에서 불가능합니다.
*   **선형 서명 해시 스케일링:** 트랜잭션 서명은 모든 입력에 대해 전체 트랜잭션을 해싱해야 했습니다. 이는 대규모 트랜잭션이 잠재적인 디도스 공격으로 변할 수 있었습니다.
*   **입력 값 서명:** 입력으로 사용된 금액 역시 서명되기에, 서명자는 실제로 사용된 수수료의 양을 조작할 수 없습니다.
*   **수용량 증가:** 블록당 1MB가 넘는 트랜잭션을 가질 수 있게 됩니다. 세그윗은 2016년 11월의 평균 트랜잭션을 기준으로 수용량을 2.1배 가량 증가 시켰습니다.
*   **사기 증명:** 나중에 개발될 것이지만, SPV(Simple Payment Verification) 지갑은 단순이 긴 체인을 따르는 것이 아닌 더 많은 합의 규칙을 검증할 수 있게 될 것입니다.

세그윗 이전에 트랜잭션 서명은 트랜잭션 ID의 계산에 사용됐습니다.

![](../assets/segwit.png)

서명은 P2PKH의 사용과 동일한 정보를 담고 있지만, scriptSig 대신 witness에 위치하게 됩니다. 그래서 ```scriptPubKey```는

```
OP_DUP OP_HASH160 0067c8970e65107ffbb436a49edd8cb8eb6b567f OP_EQUALVERIFY OP_CHECKSIG
```  

에서

```
0 0067c8970e65107ffbb436a49edd8cb8eb6b567f
```  

로 변경됩니다.

구버전의 노드에서는 이것이 스택에 두 개의 푸시가 있는 것으로 보입니다.
이는 모든 ```scriptSig```가 그것들을 사용할 수 있음을 의미합니다.
따라서 서명이 없더라도 구버전의 노드는 이러한 트랜잭션을 유효하다고 간주합니다.
새로운 노드는 첫 번째 푸시를 **witness 버전**, 두 번째 푸시를 **witness 프로그램**이라고 해석합니다.

따라서 새로운 노드도 트랜잭션을 검증하기 위해 서명이 필요합니다.

**NBitcoin에서, P2WPKH 출력을 사용하는 것은 일반적인 P2PKH을 사용하는 것과 다르지 않습니다.
공개 키에서 ```ScriptPubKey```를 가져오기 위해선 간단히 ```PubKey.Hash``` 대신 ```PubKey.WitHash```를 사용하면 됩니다.**

```cs
var key = new Key();
Console.WriteLine(key.PubKey.WitHash.ScriptPubKey);
```  

결과는 아래와 같이 보입니다.  

```
0 0067c8970e65107ffbb436a49edd8cb8eb6b567f
```  

이러한 코인의 지출을 서명하는 것은 이후 "```TransactionBuilder``` 사용하기" 부분에서 설명할 것이며, P2PKH 출력을 서명할 때 사용한 코드와 어떠한 방식도 다르지 않습니다.

```witness``` 데이터는 P2PKH의 ```scriptSig```와 유사하며, ```scriptSig```는 비어 있습니다:

```json
"in": [
{
  "prev_out": 
    {
      "hash": "725497eaef527567a0a18b310bbdd8300abe86f82153a39d2f87fef713dc8177",
      "n": 0
    },
  "scriptSig": "",
  "witness": "3044022079d443be2bd39327f92adf47a34e4b6ad7c82af182c71fe76ccd39743ced58cf0220149de3e8f11e47a989483f371d3799a710a7e862dd33c9bd842c417002a1c32901 0363f24cd2cb27bb35eb2292789ce4244d55ce580218fd81688197d4ec3b005a67"
}
```

다시 한 번 말하지만, 서명이 동일한 위치에 있지 않다는 것을 제외한다면 P2PKH와 P2WPKH는 의미가 완전히 동일합니다.
