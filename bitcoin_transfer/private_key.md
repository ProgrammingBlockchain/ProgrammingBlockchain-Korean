## 개인 키 {#private-key}

개인 키는 비트코인 주소처럼 **Bitcoin Secret**(**Wallet Import Format** 또는 간단히 **WIF**)로 불리는 Base58Check로 주로 표시됩니다.

![](../assets/BitcoinSecret.png)  

```cs  
Key privateKey = new Key(); // 랜덤 private key 생성
BitcoinSecret mainNetPrivateKey = privateKey.GetBitcoinSecret(Network.Main);  // mainnet의 개인 키를 이용해 Bitcoin secret 생성
BitcoinSecret testNetPrivateKey = privateKey.GetBitcoinSecret(Network.TestNet);  // testnet의 개인 키를 이용해 Bitcoin secret 생성
Console.WriteLine(mainNetPrivateKey); // L5B67zvrndS5c71EjkrTJZ99UaoVbMUAK58GKdQUfYCpAa6jypvn
Console.WriteLine(testNetPrivateKey); // cVY5auviDh8LmYUW8AfafseD6p6uFoZrP7GjS3rzAerpRKE9Wmuz

bool WifIsBitcoinSecret = mainNetPrivateKey == privateKey.GetWif(Network.Main);
Console.WriteLine(WifIsBitcoinSecret); // True
```  

**BitcoinSecret**을 사용하면 개인 **Key**를 쉽게 생성할 수 있습니다.

반면에, 비트코인 주소는 공개 키가 아닌 공개 키 해시를 가지고 있기에 비트코인 주소에서 공개 키 값을 생성할 수 없습니다.

다음 두 코드 블록 간의 유사성을 비교하면 위의 차이를 이해 할 수 있습니다:

```cs
Key privateKey = new Key(); // 랜덤 private key 생성
BitcoinSecret bitcoinSecret = privateKey.GetWif(Network.Main); // L5B67zvrndS5c71EjkrTJZ99UaoVbMUAK58GKdQUfYCpAa6jypvn
Key samePrivateKey = bitcoinSecret.PrivateKey;
Console.WriteLine(samePrivateKey == privateKey); // True
```  

```cs
PubKey publicKey = privateKey.PubKey;
BitcoinAddress bitcoinAddress = publicKey.GetAddress(ScriptPubKeyType.Legacy, Network.Main); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
// PubKey samePublicKey = bitcoinAddress.ItIsNotPossible; 불가능함!
```  

### 연습:
1. mainnet에서 개인 키를 생성하고 기록합니다.
2. 해당 주소를 가져옵니다.
3. 해당 주소에 비트코인을 보냅니다. 잃어버려도 될 만큼만 보내세요. 차후 수업에서 다시 찾기위한 집중력과 동기가 부여될 겁니다.
