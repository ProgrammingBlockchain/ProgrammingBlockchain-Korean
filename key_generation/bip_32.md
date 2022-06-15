## HD 지갑 (BIP 32) {#hd-wallet-bip-32}

우리가 해결하고자 하는 문제점들은 다음과 같습니다:

*  기간이 경과된 백업들
*  신뢰할 수 없는 피어에게 키/주소를 위임 하는 것

"결정론적(Deterministic)" 지갑은 이러한 백업 문제를 해결할 것입니다. 
이 지갑에 시드만 저장하면 됩니다. 이 시드와 동일한 연속적인 개인 키를 계속해서 생성할 수 있습니다.


이것이 "결정론적(Deterministic)"이 의미하는 것입니다.
보시다시피, 마스터 키에서 새로운 키들을 생성할 수 있습니다:

```cs
ExtKey masterKey = new ExtKey();
Console.WriteLine("Master key : " + masterKey.ToString(Network.Main));
for (int i = 0; i < 5; i++)
{
    ExtKey key = masterKey.Derive((uint)i);
    Console.WriteLine("Key " + i + " : " + key.ToString(Network.Main));
}
```

```
Master key : xprv9s21ZrQH143K3JneCAiVkz46BsJ4jUdH8C16DccAgMVfy2yY5L8A4XqTvZqCiKXhNWFZXdLH6VbsCsqBFsSXahfnLajiB6ir46RxgdkNsFk
Key 0 : xprv9tvBA4Kt8UTuEW9Fiuy1PXPWWGch1cyzd1HSAz6oQ1gcirnBrDxLt8qsis6vpNwmSVtLZXWgHbqff9rVeAErb2swwzky82462r6bWZAW6Ty
Key 1 : xprv9tvBA4Kt8UTuHyzrhkRWh9xTavFtYoWhZTopNHGJSe3KomssRrQ9MTAhVWKFp4d7D8CgmT7TRzauoAZXp3xwHQfxr7FpXfJKpPDUtiLdmcF
Key 2 : xprv9tvBA4Kt8UTuLoEZPpW9fBEzC3gfTdj6QzMp8DzMbAeXgDHhSMmdnxSFHCQXycFu8FcqTJRm2kamjeE8CCKzbiXyoKWZ9ihiF7J5JicgaLU
Key 3 : xprv9tvBA4Kt8UTuPwJQyxuZoFj9hcEMCoz7DAWLkz9tRMwnBDiZghWePdD7etfi9RpWEWQjKCM8wHvKQwQ4uiGk8XhdKybzB8n2RVuruQ97Vna
Key 4 : xprv9tvBA4Kt8UTuQoh1dQeJTXsmmTFwCqi4RXWdjBp114rJjNtPBHjxAckQp3yeEFw7Gf4gpnbwQTgDpGtQgcN59E71D2V97RRDtxeJ4rVkw4E
Key 5 : xprv9tvBA4Kt8UTuTdiEhN8iVDr5rfAPSVsCKpDia4GtEsb87eHr8yRVveRhkeLEMvo3XWL3GjzZvncfWVKnKLWUMNqSgdxoNm7zDzzD63dxGsm
```

동일한 개인 키를 계속해서 생성할 수 있으므로 **마스터 키**만 저장하면 됩니다.

보시다시피, 이 키는 이전에 사용했던 **Key**가 아니라 **ExtKey**입니다. 그러나 내부에 실제 개인 키가 있으므로 걱정 안해도 됩니다.

![](../assets/ExtKey.png)

**Key**와 **ChainCode**를 **ExtKey** 생성자에 사용하기 때문에, **ExtKey**에서 **Key**를 복원할 수 있습니다. 
이것은 다음과 같이 작동합니다:

```cs
ExtKey extKey = new ExtKey();
byte[] chainCode = extKey.ChainCode;
Key key = extKey.PrivateKey;

ExtKey newExtKey = new ExtKey(key, chainCode);
```

**ExtKey**에 해당하는 **base58**의 형태를 **BitcoinExtKey**라고 합니다.

그러나 두 번째 문제: 잠재적으로 해킹될 수 있는 피어(결제 서비스 등)에게 주소 생성을 위임하는 문제는 어떻게 해결할까요?

해결책은 마스터 키를 "중립화(neuter)" 시킬 수 있다는 것입니다. 
그러면 마스터 키의 공개 버전(개인 키가 없음)이 생깁니다. 
이 중립화된 버전에서는 제 3자가 개인 키를 몰라도 공개 키를 생성할 수 있습니다.

```cs
ExtPubKey masterPubKey = masterKey.Neuter();
for (int i = 0 ; i < 5 ; i++)
{
    ExtPubKey pubkey = masterPubKey.Derive((uint)i);
    Console.WriteLine("PubKey " + i + " : " + pubkey.ToString(Network.Main));
}
```

```
PubKey 0 : xpub67uQd5a6WCY6A7NZfi7yGoGLwXCTX5R7QQfMag8z1RMGoX1skbXAeB9JtkaTiDoeZPprGH1drvgYcviXKppXtEGSVwmmx4pAdisKv2CqoWS
PubKey 1 : xpub67uQd5a6WCY6CUeDMBvPX6QhGMoMMNKhEzt66hrH6sv7rxujt7igGf9AavEdLB73ZL6ZRJTRnhyc4BTiWeXQZFu7kyjwtDg9tjRcTZunfeR
PubKey 2 : xpub67uQd5a6WCY6Dxbqk9Jo9iopKZUqg8pU1bWXbnesppsR3Nem8y4CVFjKnzBUkSVLGK4defHzKZ3jjAqSzGAKoV2YH4agCAEzzqKzeUaWJMW
PubKey 3 : xpub67uQd5a6WCY6HQKya2Mwwb7bpSNB5XhWCR76kRaPxchE3Y1Y2MAiSjhRGftmeWyX8cJ3kL7LisJ3s4hHDWvhw3DWpEtkihPpofP3dAngh5M
PubKey 4 : xpub67uQd5a6WCY6JddPfiPKdrR49KYEuXUwwJJsL5rWGDDQkpPctdkrwMhXgQ2zWopsSV7buz61e5mGSYgDisqA3D5vyvMtKYP8S3EiBn5c1u4
```

따라서 결제 서비스가 pubkey1을 생성한다면, 개인 마스터 키로 해당하는 개인 키를 얻을 수 있습니다.

```cs
masterKey = new ExtKey();
masterPubKey = masterKey.Neuter();

// 결제 서비스가 pubkey1을 생성
ExtPubKey pubkey1 = masterPubKey.Derive(1);

// pubkey1의 개인키 생성
ExtKey key1 = masterKey.Derive(1);

// 올바른지 확인
Console.WriteLine("Generated address : " + pubkey1.PubKey.GetAddress(ScriptPubKeyType.Legacy, Network.Main));
Console.WriteLine("Expected address : " + key1.PrivateKey.PubKey.GetAddress(ScriptPubKeyType.Legacy, Network.Main));
```

```
Generated address : 1Jy8nALZNqpf4rFN9TWG2qXapZUBvquFfX
Expected address : 1Jy8nALZNqpf4rFN9TWG2qXapZUBvquFfX
```

**ExtPubKey**는 **Key**가 아닌 **PubKey**를 가진다는 점을 제외하고는 **ExtKey**와 유사합니다.

![](../assets/ExtPubKey.png)

이제 결정론적 키가 문제를 해결하는 방법을 보았고, "계층적(hierarchical)"이 무엇을 위한 것인지 이야기해 보겠습니다.

이전 연습에서 마스터 키 + 인덱스를 결합하여 다른 키를 생성할 수 있음을 보았습니다. 
우리는 이 프로세스를 **Derivation**이라고 부르고, 마스터 키는 **부모 키**이고, 생성된 모든 키는 **자식 키**라고 합니다.

그러나 자식 키에서 자식을 파생시킬 수도 있습니다. 이것이 "계층적(hierarchical)"이 의미하는 것입니다.

이것이 더 일반적이며 개념적으로 `부모 키 + KeyPath => 자식 키`라고 말할 수 있는 이유입니다.

![](../assets/Derive1.png)

![](../assets/Derive2.png)


이 다이어그램에서 두 가지 다른 방법으로 부모로부터 Child(1,1)을 파생시킬 수 있습니다.

```cs
ExtKey parent = new ExtKey();
ExtKey child11 = parent.Derive(1).Derive(1);
```

Or

```cs
ExtKey parent = new ExtKey();
ExtKey child11 = parent.Derive(new KeyPath("1/1"));
```

요약하자면:

![](../assets/DeriveKeyPath.png)

이는 **ExtPubKey**에서도 동일하게 작동합니다.

계층 키가 필요한 이유는 무엇일까요? 여러 계정에 대한 키 유형을 분류하는 좋은 방법일 수 있기 때문입니다.
더 자세한 내용은 [BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)를 참고하세요.

또한 조직 전체에서 계정 권한을 분할할 수 있습니다.

여러분이 회사의 CEO라고 상상해 보세요. 여러분은 모든 지갑에 대한 통제권을 원하지만, 회계 부서가 마케팅 부서의 돈을 쓰는 것을 원하지 않을 수 있습니다.

따라서 첫 번째 아이디어는 각 부서에 대해 하나의 계층 구조를 생성하는 것입니다.

![](../assets/CeoMarketingAccounting.png)

하지만 이 경우 **Accounting**와 **Marketing**은 CEO의 개인 키를 복구할 수 있게 됩니다.

이러한 하위 키를 **non-hardened**로 정의합니다.

![](../assets/NonHardened.png)

```cs
ExtKey ceoKey = new ExtKey();
Console.WriteLine("CEO: " + ceoKey.ToString(Network.Main));
ExtKey accountingKey = ceoKey.Derive(0, hardened: false);

ExtPubKey ceoPubkey = ceoKey.Neuter();

//Recover ceo key with accounting private key and ceo public key
ExtKey ceoKeyRecovered = accountingKey.GetParentExtKey(ceoPubkey);
Console.WriteLine("CEO recovered: " + ceoKeyRecovered.ToString(Network.Main));
```

```
CEO: xprv9s21ZrQH143K2XcJU89thgkBehaMqvcj4A6JFxwPs6ZzGYHYT8dTchd87TC4NHSwvDuexuFVFpYaAt3gztYtZyXmy2hCVyVyxumdxfDBpoC
CEO recovered: xprv9s21ZrQH143K2XcJU89thgkBehaMqvcj4A6JFxwPs6ZzGYHYT8dTchd87TC4NHSwvDuexuFVFpYaAt3gztYtZyXmy2hCVyVyxumdxfDBpoC
```

즉, **non-hardened 키**는 계층 구조를 "오를(climb)" 수 있습니다. **non-hardened 키**는 **중앙 통제**에 사용되는 계정 분류에만 사용해야 합니다.

따라서 이 경우 CEO는 **hardened key**를 만들어야만, 회계 부서가 계층을 올라갈 수 없습니다.

```cs
ExtKey ceoKey = new ExtKey();
Console.WriteLine("CEO: " + ceoKey.ToString(Network.Main));
ExtKey accountingKey = ceoKey.Derive(0, hardened: true);

ExtPubKey ceoPubkey = ceoKey.Neuter();

ExtKey ceoKeyRecovered = accountingKey.GetParentExtKey(ceoPubkey); //Crash
```

자식 인덱스 뒤에 작은 따옴표(apostrophe)를 사용하여 **ExtKey.Derivate**(**KeyPath)** 를 통해 강화된 키를 만들 수도 있습니다.

```cs
var nonHardened = new KeyPath("1/2/3");
var hardened = new KeyPath("1/2/3'");
```

따라서 회계 부서에서 각 고객에 대해 1개의 상위(부모) 키를 생성하고 각 고객의 지불(payment)에 대해 하위 키를 생성한다고 가정해 보겠습니다.

CEO가 이 주소 중 하나에 돈을 사용하고 싶은 경우 다음과 같이 진행하게 됩니다.

```cs
ceoKey = new ExtKey();
string accounting = "1'";
int customerId = 5;
int paymentId = 50;
KeyPath path = new KeyPath(accounting + "/" + customerId + "/" + paymentId);
//Path : "1'/5/50"
ExtKey paymentKey = ceoKey.Derive(path);
```
