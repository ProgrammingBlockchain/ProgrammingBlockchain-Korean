# HiddenBitcoin: Managing keys (HD wallet) {#hiddenbitcoin-keystorage}  

([nopara73](https://github.com/nopara73)) 저는 [HiddenWallet](https://github.com/nopara73/HiddenWallet)이라는 개인정보보호 지향 비트코인 지갑을 개발 중입니다. [HiddenBitcoin](https://github.com/nopara73/HiddenBitcoin) 라이브러리는 NBitcoin과 사용자 인터페이스 사이에 다른 추상화 계층을 도입 한 것입니다. 

비트 코인 지갑에는 세 가지 주요 기능이 있으며 다음과 같이 구성됩니다:

1. 안전한 키 저장과 접근(access) 관리
2. 이러한 키들과 블록체인 상의 키들을 모니터링 합니다.
3. 트랜잭션을 작성하고 제출합니다.

이 강의에서는 키 저장 기능을 다룰 것입니다.
코드를 좀 더 자세히 살펴보고 싶다면 [GitHub](https://github.com/nopara73/HiddenBitcoin)에서 해결책을 찾을 수 있습니다.
빠르게 설정하고 사용하는 방법 만 알고 싶다면 [CodeProject](http://www.codeproject.com/Articles/1096320/HiddenBitcoin-High-level-Csharp-Bitcoin-wallet-lib)에서 저의 고급 튜토리얼을 찾을 수 있습니다.

**How high level is it?** GUI 개발자, 디자이너들이 많은 실수를 하지 않도록 해야 합니다. 그들은 입력, 출력 및 scriptpubkey에 대해 알지 못합니다. 주소, 개인키 및 지갑 수준을 고수(?) 해야합니다. 또한 NBitcoin은 완전히 추상화 되어야 합니다. 

## Key storage design decisions  

이제 키를 저장하기 위해 어떤 결정을 내려야 하는지 그리고 키를 만드는 동안 염두에 두어야 할 사항에 대해 템플릿을 만들어 보겠습니다. 

### Using only one key  

이것은 바로가기(shortcut)입니다. 이 길을 가는 것이 정당화 될 수 있는 상황은 그리 많지 않습니다만 여러분의 목적에는 대부분 적합 할 것입니다.
여기에 나쁜 본보기는 내가 만든 비트 코인 지갑에 대한 예시인데, 하나의 키만 사용합니다. 결과에 대해 생각하는 것은 여러분에게 맡깁니다. 

![](../assets/TransparentWallet2.png)  

### JBOK wallets  

**J**ust a **B**unch **O**f **K**eys를 의미합니다. 
참조 클라이언트를 쓸 때 이 방법을 사용하여 키를 저장합니다.
이 문제는 사용자가 주기적으로 지갑을 백업해야 한다는 것입니다. 
하지만 키를 가져오거나 삭제하고 암호를 변경하려면 암호와 결정론적 지갑을 조합하거나 하이브리드 방식으로 사용해야 합니다. 
히든지갑(HiddenWallet)이 사생활 보호를 위해 혁신을 시도하고 있고, 그런 지갑 없이도 더 건전한 지갑 구조를 가질 수 있기 때문에 그것을 사용하지 않기로 했습니다. 


### BIP38 (Part 2) - Untrusted third party key generator  

다시 말하자면, 아이디어는 키 생성기에 PassphraseCode를 추가하는 것입니다. 이 PassphraseCode를 사용하면 사용자의 암호 또는 개인 키를 몰라도 사용자를 대신하여 암호화 된 키를 생성 할 수 있습니다.
HiddenWallet은 데스크톱 지갑입니다 (아마도 얼마 동안은 변경되지 않을 것입니다). 따라서 키를 생성 하거나 저장 하기 위해 믿을 수 없는 써드파티를 사용 할 필요가 없습니다. 아직까지 구현 여부는 결정하지 못했습니다. 


### SHD wallet  

이것이 내가 구현한 지갑 구조입니다. 
방금 이 단어를 생각해냈는데, 그것은 존재하지도 않고 아무도 사용하지 않고 있습니다만, SHD는 **S**theals 와 **H**ierarchical **D**eterministic 지갑을 의미합니다. 내가 만든 것을 잘 설명 하는 문장 입니다.
성능이 떨어졌기 때문에, 이 코드를 작성하기 전까지는 스텔스(Stheals) 부분만 구현했습니다. 스텔스(Stheals) 주소가 비트코인의 미래에 어떻게 사용 될 지는 미지수 입니다.


스텔스(Stheals) 주소는 다음과 같습니다: ```waPXAvDCDGv8sXYRY6XDEymDGscYeepXBV5tgSDF1JHn61rzNk4EXTuBfx22J2W9rPAszXFmPXwD2m52psYhXQe5Yu1cG26A7hkPxs```  

## Black box  

**Safe** 클래스를 구현했습니다. 이 클래스를 사용하면 블랙박스 처럼 직관적입니다. 

```cs
var network = Network.MainNet;
```  

**Network**는 **NBitcoin.Network**가 아닙니다. GUI 개발자는 NBitcoin을 사용하고 있다는 것을 알면 안됩니다. 
또한 NBitcoin에서 선택할 수 있는 더 많은 네트워크 옵션이 있지만 HiddenBitcoin은 모든 옵션을 처리 할 수 없습니다. 현재는 ```MainNet```과 ```TestNet''`을 지원합니다.
네트워크는 열거 형이며 **HiddenBitcoin.DataClasses** 네임 스페이스에서 찾을 수 있습니다. 


```cs
string mnemonic;
Safe safe = Safe.Create(out mnemonic, "password", walletFilePath: @"Wallets\hiddenWallet.hid", network);
Console.WriteLine(mnemonic);
```  

**safe**를 불러오거나 복구 가능 합니다:  

```cs
Safe loadedSafe = Safe.Load("password", walletFilePath: @"Wallets\hiddenWallet.hid");
if (network != loadedSafe.Network)
    throw new Exception("WrongNetwork");

Safe recoveredSafe = Safe.Recover(mnemonic, "password", walletFilePath: @"Wallets\sameHiddenWallet.hid", network);
```  

**safe**에서 키를 문자열로 얻을 수 있습니다:

```cs
Console.WriteLine("Seed private key: " + safe.Seed);
Console.WriteLine("Seed public key: " + safe.SeedPublicKey);
Console.WriteLine("Third child address: " + safe.GetAddress(2));
Console.WriteLine("First child private key: " + safe.GetPrivateKey(0));
Console.WriteLine("Second child private key and the corresponding address: ");
Console.WriteLine(safe.GetPrivateKeyAddressPair(1).PrivateKey);
Console.WriteLine(safe.GetPrivateKeyAddressPair(1).Address);
Console.WriteLine("The stealth address: " + safe.StealthAddress);
Console.WriteLine("Scan and spend private keys for stealth payments:");
Console.WriteLine(loadedSafe.ScanPrivateKey);
Console.WriteLine(loadedSafe.SpendPrivateKey);
```  

```
Seed private key: xprv9s21ZrQH143K4RBm26TMm3qwTtR3Eyh22xDEN3TBebgfAvHPPSjxxFnFGDtnNHvqZ7pihGmAc8o9y1UvfEzcxSzyXAnmvTBowCNi69nXsqJ
Seed public key: xpub661MyMwAqRbcGuGE87zN8Bng1vFXeSQsQB8qARroCwDe3icXvz4DW46j7U6fX8NsKhqcxR7K1mDX4gTbtvCGdeJz5M7py3yEqMsjUH2DYhb
Third child address: 17pGpPX1A2sCdqJXsC5BiwdFphFVgJR9nk
First child private key: xprv9ubnoo3dgCYfrWbYBEM71WoBvzwTtQemEdjW836CeWJYunYBskQhq3nrJMvNBCCFpnU5GbgbL1b2QbPHA4rRPESEhqfKzae5oWe7SAMuxAV
Second child private key and the corresponding address:
xprv9ubnoo3dgCYfuE1hVB3F3Sh5YFJUNUjyZ68PDzPNhpmtqWDtD45zucZYMUAjY22HNxaY6tsvGAdJdcyALCMm2mTAvA4pEp1m7y3BSccKY4r
19FHdsj2YT79TuxbWcDMz9opTU28L1memr
The stealth address: vJmuFuLggpgzivm3UUjQguLhMA6C1SnYFJu5N6QkmXYRCU3nG1Ww36VcXy6zXpJvGeVTidxcsu7U19sfB1rxHhzvSNV5eGGLk6G1Cb
Scan and spend private keys for stealth payments:
L5CTS4U27umRfSBu2ztxsyUeMEYzJJk3HvCp3deSQBJWmRSUqCLg
KyXveppF4Xm3KwJgG7EBSi6SxfMTkaDXYYmv7c7xWRcF7yUNpswp
```  

**Note:** 이상적으로 씨앗(sedd) 키는 사용되지 않습니다. **safe**의 getters로 키를 반복 시작하는게 더 좋은 방법입니다. 

## White box  

### Safe.Create  

```cs
// Creates a mnemonic, a seed, encrypts it and stores in the specified path.
public static Safe Create(out string mnemonic, string password, string walletFilePath, Network network)
{
    var safe = new Safe(password, walletFilePath, network);
    mnemonic = safe.SetSeed(password).ToString();
    safe.Save(password, walletFilePath, network);

    return safe;
}
```  

```safe.SetSeed```는 니모닉(기억을 돕는)을 만들고 ```_seedPrivateKey```를 설정 합니다. 
니모닉을 반환하므로, 클래스 사용자가 다시 이용 할 수 있습니다. 

![](../assets/RootKey.png)  

```cs
private ExtKey _seedPrivateKey;
private Mnemonic SetSeed(string password)
{
    var mnemonic = new Mnemonic(Wordlist.English, WordCount.Twelve);

    _seedPrivateKey = mnemonic.DeriveExtKey(password);

    return mnemonic;
}
```  

### safe.Save  

지갑 파일을 저장합니다. 그 안에 무엇이 있을까요?

```json
{
  "EncryptedSeed":"6PYXR8U5Nu9UoGZcU95DWWKCXppKnYBUKyJgze6DX6bQDNwFzNdJApUzXT",
  "ChainCode":"C+2MiZU7R/33bkvgdDqdQp7xx3nXHSIzS6bUgRsnaus=",
  "Network":"MainNet"
}
```

지갑 파일은 JSON 형식입니다.
ExtKey에서 체인 코드와 개인 키를 얻을 수 있습니다. 다른 방식으로도 작동합니다. 

```cs
Key privateKey = _seedPrivateKey.PrivateKey;
byte[] chainCode = _seedPrivateKey.ChainCode;
```  

마지막으로 개인 키를 암호화합니다. 

![](../assets/EncryptedKey.png)  

```cs
string encryptedBitcoinPrivateKeyString = privateKey.GetEncryptedBitcoinSecret(password, _network).ToWif();
string chainCodeString = Convert.ToBase64String(chainCode);
string networkString = network.ToString();
```  

### Safe.Load

저장(Safe.save) 과정을 반대로 합니다. 

```cs
public static Safe Load(string password, string walletFilePath)
{
    if (!File.Exists(walletFilePath))
        throw new Exception("WalletFileDoesNotExists");

    var walletFileRawContent = WalletFileSerializer.Deserialize(walletFilePath);

    var encryptedBitcoinPrivateKeyString = walletFileRawContent.EncryptedSeed;
    var chainCodeString = walletFileRawContent.ChainCode;

    var chainCode = Convert.FromBase64String(chainCodeString);

    Network network;
    var networkString = walletFileRawContent.Network;
    if (networkString == Network.MainNet.ToString())
        network = Network.MainNet;
    else if (networkString == Network.TestNet.ToString())
        network = Network.TestNet;
    else throw new Exception("NotRecognizedNetworkInWalletFile");

    var safe = new Safe(password, walletFilePath, network);

    var privateKey = Key.Parse(encryptedBitcoinPrivateKeyString, password, safe._network);
    var seedExtKey = new ExtKey(privateKey, chainCode);
    safe._seedPrivateKey = seedExtKey;

    return safe;
}
```  

다음은 Safe 생성자에서 일어나는 일들 입니다:  

```cs
private Safe(string password, string walletFilePath, Network network)
{
    SetNetwork(network);
    
    SetSeed(password, mnemonicString);
    
    WalletFilePath = walletFilePath;
}
```  

### SetNetwork

클래스 내부에 ```NBitcoin.Network```로 작업하는 것이 좋습니다. 그러기 위해 private member 함수를 작성 합니다. 

```cs
private NBitcoin.Network _network;
private void SetNetwork(Network network)
{
    if (network == Network.MainNet)
        _network = NBitcoin.Network.Main;
    else if (network == Network.TestNet)
        _network = NBitcoin.Network.TestNet;
    else throw new Exception("WrongNetwork");
}
```  

### Safe.Recover  

```cs
public static Safe Recover(string mnemonic, string password, string walletFilePath, Network network)
{
    var safe = new Safe(password, walletFilePath, network, mnemonic);
    safe.Save(password, walletFilePath, network);
    return safe;
}
```  

이 작업을 위해서 생성자를 추가 해야 합니다:

```cs
private Safe(string password, string walletFilePath, Network network, string mnemonicString = null)
{
    SetNetwork(network);

    if (mnemonicString != null)
    {
        var mnemonic = new Mnemonic(mnemonicString);
        _seedPrivateKey = mnemonic.DeriveExtKey(password);
    }

    WalletFilePath = walletFilePath;
}
```  

### Getters

키를 파생하는 방법은 다음과 같습니다. 목적을 위해 복잡한 키 경로를 사용하는 것은 의미가 없습니다: 

```cs
public PrivateKeyAddressPair GetPrivateKeyAddressPair(int index)
{
    var foo = _seedPrivateKey.Derive(index, true).GetWif(_network);
    return new PrivateKeyAddressPair
    {
        PrivateKey = foo.ToWif(),
        Address = foo.ScriptPubKey.GetDestinationAddress(_network).ToWif()
    };
}
```  

### Stealth  

```cs
private Key _spendPrivateKey => _seedPrivateKey.PrivateKey;
public string SpendPrivateKey => _spendPrivateKey.GetWif(_network).ToWif();
private Key _scanPrivateKey => _seedPrivateKey.Derive(0, hardened: true).PrivateKey;
public string ScanPrivateKey => _scanPrivateKey.GetWif(_network).ToWif();

public string StealthAddress => new BitcoinStealthAddress
    (_scanPrivateKey.PubKey, new[] {_spendPrivateKey.PubKey}, 1, null, _network
    ).ToWif();
```


