## 그게 정말 무작위일까요? {#is-it-random-enough}

**new Key()** 를 호출하면, 내부적으로는 개인 키를 생성하기 위해서 의사 난수 생성기([Pseudo-Random-Number-Generator](https://en.wikipedia.org/wiki/Pseudorandom_number_generator))를 사용합니다. Windows 환경에서는 Windows Crypto API의 .NET 래퍼인 **RNGCryptoServiceProvider**를 사용합니다.

Android 환경에서는 **SecureRandom** 클래스를 사용하지만, 원한다면 **RandomUtils.Random**을 이용한 자신만의 구현체를 사용할 수 있습니다.

iOS 환경에서는 아직 구현되지 않았기에 자신만의 **IRandom** 구현체가 필요합니다.

컴퓨터에게 있어서, 랜덤하다는 것은 어렵습니다. 가장 큰 문제는 주어진 숫자들이 정말 랜덤인지 확인하는 것은 불가능하다는 점입니다.

만약 악성코드가 난수 생성기를 조작한다면(그래서 우리가 생성한 숫자를 예측할 수 있다면), 상황이 발생하기 전까진 알아차릴 수 없습니다.

이는 크로스 플랫폼과 허술한 난수 생성기(예를 들면 CPU 속도와 연관된 컴퓨터의 시간 사용)는 위험합니다. 그러나 일이 벌어지기 전까진 알아차릴 수 없습니다.

성능의 이유로 대부분의 난수 생성기는 똑같은 방식으로 작동합니다: **시드**라고 부르는 난수를 고르고 예측 가능한 수식이 요청할 때마다 다음 숫자를 생성합니다.

시드의 난해함 정도는 **엔트로피**라고 불리는 측정으로 정의되지만, **엔트로피**의 정도 역시 관측자에 의존합니다.

시계의 시간으로부터 시드를 생성한다고 가정해 봅시다.
그리고 간단하게, 시계는 1ms의 분해능을 가지고 있다고 해봅시다. (사실, 15ms 정도 더 큽니다.)

만약 공격자가 저번주에 생성한 키를 알고있다면, 당신의 시드는
`1000 * 60 * 60 * 24 * 7 = 604800000가지`의 가능성
을 가집니다.

공격자에게 엔트로피는 log<sub>2</sub>(604800000) = 29.17비트 입니다.

그리고 그러한 수를 대입하는 건 최근 컴퓨터로 2초가 채 안걸립니다. 이러한 방식을 "브루트 포스"라고 합니다.

그러나 만약 시드로 프로세스 ID와 시간을 사용한다고 가정해 봅시다.
프로세스 ID는 1024까지 가능하다고 생각해 봅시다.

그렇다면, 공격자는 `604800000 * 1024가지`를 대입해야 합니다. 이는 대략 2000초가 소요됩니다.
돌아와서, 시드에 컴퓨터를 켠 시간을 추가하고 공격자는 오늘 켰다는 것을 알고 있다고 가정한다면, 86400000가지의 가능성이 추가됩니다.

즉, 공격자는 `604800000 * 1024 * 86400000 = 5,35088E+19가지`의 가능성을 대입해야 합니다.
하지만 공격자가 컴퓨터를 장악해서 마지막 정보(컴퓨터를 켠 시간)을 얻어, 가능성의 수를 줄여 엔트로피를 줄일 수 있다는 것을 명심하세요.

엔트로피는 **log<sub>2</sub>(가능성의 수)** 로 측정됩니다. 즉 log<sub>2</sub>(5,35088E+19) = 65비트 입니다.

이걸로 충분할까요? 뭐, 공격자가 시드를 생성할 때 사용된 영역에 대한 정보를 모른다고 가정한다면 말이죠.

그러나 공개 키의 해시는 20바이트 (160비트)이므로, 주소들의 모든 경우의 수보다 작습니다. 아직 멀었습니다.

> **Note:** 엔트로피를 추가하는 것은 선형적으로 어렵지만, 엔트로피를 깨는 것은 기하급수적으로 어렵습니다.

빠르게 엔트로피를 생성하는 재미있는 방법은 마우스 움직임과 같은 인간의 개입을 이용하는 것입니다.

만약 플랫폼의 난수 생성기를 완전히 신뢰할 수 없다면([꽤나 합리적인 의심](http://android-developers.blogspot.fr/2013/08/some-securerandom-thoughts.html)입니다.), NBitcoin을 사용한 난수 생성기에 엔트로피를 추가할 수 있습니다.

```cs
RandomUtils.AddEntropy("hello");
RandomUtils.AddEntropy(new byte[] { 1, 2, 3 });
var nsaProofKey = new Key();
```

**AddEntropy(data)**를 호출하면 NBitcoin은 다음과 같이 작동합니다:
**additionalEntropy = SHA(SHA(data) ^ additionalEntropy)**

새로운 수를 생성할 때:
**result = SHA(PRNG() ^ additionalEntropy)**

## Key Derivation 함수 {#key-derivation-function}

However, what is most important is not the number of possibilities. It is the time that an attacker would need to successfully break your key. That’s where KDF enters the game.

KDF, 혹은 **Key Derivation 함수**는 엔트로피가 낮더라도 강한 키를 만드는 방법입니다.

당신이 시드를 만드려고 하고, 공격자는 10,000,000가지의 가능성이 있다는 것을 알고 있다고 가정합시다.
그러한 시드는 일반적으로 꽤나 쉽게 뚫리게 됩니다.

하지만 대입을 느리게 만들 수 있다면 어떨까요?
KDF는 의도적으로 컴퓨팅 자원을 낭비하는 해시 함수입니다.

예시:

```cs
var derived = SCrypt.BitcoinComputeDerivedKey("hello", new byte[] { 1, 2, 3 });
RandomUtils.AddEntropy(derived);
```

공격자는 엔트로피가 5글자임을 알면서도 각 가능성을 확인하기 위해선, Scrypt를 수행해야 합니다. 이는 제 컴퓨터에서(역자주: 저자의 컴퓨터) 5초정도 걸리는 작업입니다.

결론: 난수 생성기를 불신하는 것은 전혀 편집증이 아니며, 엔트로피를 추가하거나 KDF를 사용함을 통해 공격을 완화할 수 있습니다.
공격자는 사용자나 시스템으로부터 정보를 얻어 엔트로피를 줄일 수 있음을 항상 염두하세요.
만약 시간을 엔트로피로 사용한다면, 공격자는 키를 저번주에 만들었고, 오전 9시부터 오후 6시에만 컴퓨터를 사용한다는 정보를 사용자로부터 알아내서 엔트로피를 줄일 수 있습니다.

이전 파트에서 **Scrypt**라 불리는 KDF에 대해 간략히 소개했습니다. 위에서 말했듯, KDF는 브루트 포스를 어렵게 만드는 것이 목표입니다.

그렇기에 KDF를 사용한 비밀번호로 개인 키를 암호화하는 표준이 존재합니다. 이것이 바로 [BIP38](http://www.codeproject.com/Articles/775226/NBitcoin-Cryptography-Part)입니다.

![](../assets/EncryptedKey.png)

```cs
var privateKey = new Key();
var bitcoinPrivateKey = privateKey.GetWif(Network.Main);
Console.WriteLine(bitcoinPrivateKey); // L1tZPQt7HHj5V49YtYAMSbAmwN9zRjajgXQt9gGtXhNZbcwbZk2r
BitcoinEncryptedSecret encryptedBitcoinPrivateKey = bitcoinPrivateKey.Encrypt("password");
Console.WriteLine(encryptedBitcoinPrivateKey); // 6PYKYQQgx947Be41aHGypBhK6TA5Xhi9TdPBkatV3fHbbKrdDoBoXFCyLK
var decryptedBitcoinPrivateKey = encryptedBitcoinPrivateKey.GetSecret("password");
Console.WriteLine(decryptedBitcoinPrivateKey); // L1tZPQt7HHj5V49YtYAMSbAmwN9zRjajgXQt9gGtXhNZbcwbZk2r

Console.ReadLine();
```

이러한 암호화는 두가지 경우에 쓰입니다.

*   스토리지 프로바이더를 신뢰할 수 없을 때 (해킹의 위험을 가지고 있을 때)
*   누군가를 대신하여 키를 보관할 때 (그리고 그들의 키를 알고 싶지 않을 때)

만약 자신만의 스토리지가 있다면, 데이터베이스 수준의 암호화도 충분합니다.

만약 서버가 복호화된 키를 소유하고 있다면 조심하세요. 공격자가 키를 복호화하기 위해 DDoS를 시도할 수 있습니다.

되도록이면 최종 사용자에게 복호화를 위임하세요.
