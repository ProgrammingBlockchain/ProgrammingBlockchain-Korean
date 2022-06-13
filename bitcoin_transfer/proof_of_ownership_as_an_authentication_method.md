## 인증 방법으로 소유권 증명 {#proof-of-ownership-as-an-authentication-method}
> [[2016.05.02](https://www.youtube.com/watch?v=dZNtbAFnr-0)] 저는 Craig Wright고, 비트코인에서 수행된 첫 번째 거래의 공개 키로 메시지의 서명을 시연하려고 합니다.

```cs
var bitcoinPrivateKey = new BitcoinSecret("XXXXXXXXXXXXXXXXXXXXXXXXXX", Network.Main);

var message = "I am Craig Wright";
string signature = bitcoinPrivateKey.PrivateKey.SignMessage(message);
Console.WriteLine(signature); // IN5v9+3HGW1q71OqQ1boSZTm0/DCiMpI8E4JB1nD67TCbIVMRk/e3KrTT9GvOuu3NGN0w8R2lWOV2cxnBp+Of8c=
```  

참 쉽죠?

여러분은 아마 사토시 나카모토라고 여겨지길 원했던 Craig Wright를 기억할 것입니다.

그는 소수의 영향력있는 비트코인 사람들과 언론인을 사회공학적 방법으로 성공적으로 설득했습니다.

다행히 디지털 서명은 그렇게 작동하지 않습니다. [Internet](https://en.bitcoin.it/wiki/Genesis_block)에서 제네시스 블록: [1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa](https://blockchain.info/address/1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa)와 관련된 최초의 비트코인 주소를 찾아보고 그의 주장을 확인해 봅시다:

```cs
var message = "I am Craig Wright";
var signature = "IN5v9+3HGW1q71OqQ1boSZTm0/DCiMpI8E4JB1nD67TCbIVMRk/e3KrTT9GvOuu3NGN0w8R2lWOV2cxnBp+Of8c=";

var address = new BitcoinPubKeyAddress("1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa", Network.Main);
bool isCraigWrightSatoshi = address.VerifyMessage(message, signature);

Console.WriteLine("Craig Wright가 Satoshi일까? " + isCraigWrightSatoshi);
```  

스포일러 경고! 결과는 거짓입니다.

다음은 코인을 이동하지 않고 주소의 소유자임을 증명하는 방법입니다:

**Address:**  
[1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB](https://blockchain.info/address/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB)  

**Message:**  
Nicolas Dorier Book Funding Address  

**Signature:**  
H1jiXPzun3rXi0N9v9R5fAWrfEae9WPmlL5DJBj1eTStSvpKdRR8Io6/uT9tGH/3OnzG6ym5yytuWoA9ahkC3dQ=  

이것은 Nicolas Dorier가 책의 개인키를 소유하고 있다는 증거입니다.

**Exercise:** Nicolas 센세가 거짓말을 하고 있는지 확인하세요!

### Sidenote

PGP가 어떻게 작동하는지 아시나요? 꽤나 비슷하죠?

아마 이것은 보다 사용자 친화적인 PGP 대안의 기초가 될 수 있습니다.

NBitcoin 위에 구축 해주세요 :-)
