## P2WSH (Pay to Witness Script Hash) {#p2wsh-pay-to-witness-script-hash}

P2PKH와 P2WPKH처럼, P2SH와 P2WSH 간에 유일한 차이점은 ```scriptSig```에 있던 것의 위치와 ```scriptPubKey```가 수정되었다는 점입니다.

```scriptPubKey```는 이러한 것에서:

```OP_HASH160 10f400e996c34410d02ae76639cbf64f4bdf2def OP_EQUAL```

이렇게 바뀝니다:

```0 e4d3d21bab744d90cd857f56833252000ac0fade318136b713994b9319562467```

아래 코드로 출력해볼 수 있습니다:  

```cs
var key = new Key();
Console.WriteLine(key.PubKey.ScriptPubKey.WitHash.ScriptPubKey);
```  

```scriptSig```에 있던 내용(서명 + redeem 스크립트)도 ```witness``` 옮겨집니다:

```json
"in": [
    {
      "prev_out": {
        "hash": "ffa2826ba2c9a178f7ced0737b559410364a62a41b16440beb299754114888c4",
        "n": 0
      },
      "scriptSig": "",
      "witness": "304402203a4d9f42c190682826ead3f88d9d87e8c47db57f5c272637441bafe11d5ad8a302206ac21b2bfe831216059ac4c91ec3e4458c78190613802975f5da5d11b55a69c601 210243b3760ce117a85540d88fa9d3d605338d4689bed1217e1fa84c78c22999fe08ac"
    }
  ]

```

이전에 설명했던 P2SH처럼, P2WSH는 서명하기 위해 ```ScriptCoin```을 완전히 동일하게 사용합니다.
