## 비트코인 주소 {#bitcoin-address}

**비트코인 주소**는 다른 사람이 여러분에게 코인을 송금할 수 있도록 공개하는 주소입니다.

![](../assets/BitcoinAddress.png)  

공개된 주소로 송금받은 코인을 사용하려면 **개인 키**(Private Key)를 이용해야 합니다.

![](../assets/PrivateKey.png)  

개인 키는 네트워크에 저장되지 않으며 인터넷에 연결하지 않고 생성할 수 있습니다.

아래 코드는 NBitcoin을 사용해 개인키를 생성하는 방법입니다.

```cs  
Key privateKey = new Key(); // 랜덤 private key를 생성함.
```  

단방향 암호화 기능을 이용하여 개인 키로부터 **공개 키**(Public Key)를 생성할 수 있습니다. 

![](../assets/PrivKeyPubKey.png)

```cs 
PubKey publicKey = privateKey.PubKey;
Console.WriteLine(publicKey); // 0251036303164f6c458e9f7abecb4e55e5ce9ec2b2f1d06d633c9653a07976560c
```  

비트코인은 두 가지의 **네트워크**가 있습니다: 

* **TestNet**은 개발을 위한 네트워크입니다. 이 네트워크의 비트코인은 가치가 없습니다.  
* **MainNet**은 모두가 사용하는 네트워크입니다.

> **메모:** TestNet에서 사용되는 코인은 **faucets**에서 얻을 수 있습니다. "get testnet bitcoins"으로 구글링 하세요.

주소가 사용되는 **네트워크**와 공개 키를 이용하여 **비트코인 주소**를 쉽게 얻을 수 있습니다.

![](../assets/PubKeyToAddr.png)

```cs 
Console.WriteLine(publicKey.GetAddress(ScriptPubKeyType.Legacy, Network.Main)); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(publicKey.GetAddress(ScriptPubKeyType.Legacy, Network.TestNet)); // n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```  

**정확하게 표현하자면, 비트코인 주소는 버전을 나타내는 선두 1바이트(TestNet과 MainNet을 구분하는 1바이트)와 공개 키 해시로 구성되어 있습니다. 이를 병합한 후 Base58Check로 인코딩됩니다.**

![](../assets/PubKeyHashToBitcoinAddress.png)  

```cs 
var publicKeyHash = publicKey.Hash;
Console.WriteLine(publicKeyHash); // f6889b21b5540353a29ed18c45ea0031280c42cf
var mainNetAddress = publicKeyHash.GetAddress(Network.Main);
var testNetAddress = publicKeyHash.GetAddress(Network.TestNet);
```  

> **Fact:** 공개 키 해시는 Big Endian 표기법으로 공개 키를 SHA256을 해싱한 결과에 RIPEMD160을 해싱하여 만들어 집니다. 함수를 사용하면 다음과 같이 보입니다: RIPEMD160(SHA256(pubkey))  

Base58Check 인코딩에는 오타를 방지하기 위한 체크섬 기능과 '0'(Zero) 및 'O'(alphabet)와 같은 모호한 문자의 사용을 방지하는 기능이 있습니다.
또한 MainNet 코인을 TestNet 주소로 보내는 것을 방지하는 것처럼, 주어진 주소로 네트워크를 정하는 일관된 방법을 제공합니다.

```cs 
Console.WriteLine(mainNetAddress); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(testNetAddress); // n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```  

> **Tip:** MainNet에서 비트코인 프로그래밍을 연습하는 경우 문제 발생시 복구할 수 없습니다.
