# Web API/ Block Explorer {#web-api}

일반적으로 블록 탐색기(block explorers)가 제공하는 웹 API를 사용하면 좀더 빠르게 시작할 수 있습니다. 
이 책에서 이미 `QBitNinja`를 사용했지만 더 많이 사용 하게 됩니다.
블록탐색기는 체인의 블록, 트랜젝션 및 주소에 대한 정보를 제공하는 자체 호스팅 또는 타사 호스팅 솔루션입니다.

![Explorer](../assets/Wallet-Explorer.png)

블록 탐색기는 비트 코인 노드에 연결하고 블록 체인의 데이터를 인덱싱하고 사용하기 쉬운 API를 노출합니다.
솔루션에는 `QBitNinja`, `Blockcypher`, `Smartbit`, `Electrum server`, `Insight`, `NBXplorer`등이 있습니다.

장점:

* Bitcoin Core RPC보다 더 좋은 API,
* 더 많은 부하(load)를 처리 할 수 있습니다.
* 많은 수의 지갑을 지원하고 동적으로 추가 할 수 있습니다.
* 빠른 클라이언트-서버 아키텍처

단점:

* 타사에서 호스팅 되는 문제있는 포크(fork)가 발견된 경우 포크(fork) 선택권이 없습니다.
* 때로는, 그들의 서비스가 전체 지갑에 필요한 모든 것을 처리하기에 충분하지 않습니다.
* 존재하지 않는 개인정보: 서버는 클라이언트에 대한 모든 것을 알고 있습니다. 자체 호스팅 유형에는 적용되지 않습니다.


서로 다른 블록 탐색기는 다른 API와 기능을 노출합니다. 
예를 들어 대부분의 블록 탐색기는 HTTP 웹 API를 사용하는 반면 Electrum은 [the Stratum](http://docs.electrum.org/en/latest/protocol.html) 프로토콜을 사용합니다.
블록 탐색기는 지갑의 개인 키를 가지고 있지 않습니다.

`QBitNinja`에서는 항상 주소를 변경하는 지갑인 경우 추적하기가 어렵습니다.
동일한 지갑에 속한 모든 주소를 폴링하여 변경 사항을 감지해야 하기 때문입니다.

그러나 `Electrum` 또는`NBXplorer` 및 `SmartBit`은 웹소켓(websockets) 또는 롱폴링(long polling)을 통해 알림을 노출하므로 지갑의 모든 주소를 폴링 할 필요가 없습니다.
`Insight`가 잘 유지되지 않습니다. `Blockcypher`,`QBitNinja` 및 `Smartbit`는 타사에서 호스팅됩니다. 
이러한 방식으로 지갑을 구축하는 데 관심이 있다면 nopara73의 CodeProject 기사를 참조하십시오: [Build your own Bitcoin wallet with QBitNinja in C#](https://www.codeproject.com/Articles/1115639/Build-your-own-Bitcoin-wallet)

[NBxplorer](https://github.com/dgarage/NBXplorer/)는 매우 간단한 API를 갖도록 만들어졌으며, 자체 호스팅이 가능하며, 지갑에 필요한 것만 추적합니다.
`QBitNinja`와는 달리 풀노드(full node)가 있어야 하지만, 웹소켓 알림을 제공하고, 지갑 잔액을 쉽게 조회 할 수 있습니다.

NBXplorer는 또한 단일 서버에서 다중 암호화 통화입니다. 2018 년 10 월 현재 비트 코인, 라이트 코인, BCash, BGold, Dash, Dogecoin, Dystem, Feathercoin, Groestlcoin, Monacoin, Polis, UFO, Viacoin 및 Zclassic를 지원 합니다.

또한, 'NBitcoin'과 원활하게 통합됩니다.

NBXplorer를 설정하려면 기본 매개 변수가 있는 완전히 동기화 된 `bitcoind` 노드가 필요합니다.
그런 다음 기본 매개 변수로 [NBXplorer](https://github.com/dgarage/NBXplorer)를 복제하고 실행합니다.

`NBXplorer.Client` nuget package를 참조한 다음 `NBXplorer`에 사용자 지갑을 추적하도록 알려야합니다:

```cs
var network = new NBXplorerNetworkProvider(NetworkType.Mainnet).GetBTC();
var userExtKey = new ExtKey();
var userDerivationScheme = network.DerivationStrategyFactory.CreateDirectDerivationStrategy(userExtKey.Neuter(), new DerivationStrategyOptions()
{
	// Use non-segwit
	Legacy = true
});
ExplorerClient client = new ExplorerClient(network);
client.Track(userDerivationScheme);
```

Testnet 또는 Regtest를 사용하려면 `NetworkType.Mainnet` 문장을 변경하십시오.

사용하지 않은 새 주소를 원하는 경우:

```cs
Console.WriteLine(client.GetUnused(userDerivationScheme, DerivationFeature.Deposit).Address);
```

그런 다음 사용자의 UTXO를 쿼리하고 다음과 같은 방식으로 사용할 수 있습니다:

```cs
var utxos = client.GetUTXOs(userDerivationScheme, null, false);
```

해당 UTXO를 사용하려면:

```cs
var coins = utxos.GetUnspentCoins();
var keys = utxos.GetKeys(userExtKey);
TransactionBuilder builder = Network.Main.CreateTransaction();
builder.AddCoins(coins);
builder.AddKeys(keys);
builder.Send(new Key(), Money.Coins(0.5m));
builder.SetChange(changeAddress.ScriptPubKey);

// Set the fee rate
var fallbackFeeRate = new FeeRate(Money.Satoshis(100), 1);
var feeRate = tester.Client.GetFeeRate(1, fallbackFeeRate).FeeRate;
builder.SendEstimatedFees(feeRate);
/////

var tx = builder.BuildTransaction(true);
Console.WriteLine(client.Broadcast(tx));
```

이 솔루션의 문제점은 동시에 두번 호출하면, 동일한 코인을 소비하는 두 개의 트랜잭션을 브로드캐스팅하면, 트랜잭션 중 하나가 삭제된다는 것입니다.

이 문제를 방지하려면 동일한 동전을 두 번 사용하지 않도록해야 합니다.

문제를 해결하는 방법은 간단히 재 시도하는 것입니다:

```cs
while(true)
{    
    var coins = utxos.GetUnspentCoins();
    var keys = utxos.GetKeys(userExtKey);
    TransactionBuilder builder = Network.Main.CreateTransactionBuilder();
    builder.AddCoins(coins);
    builder.AddKeys(keys);
    builder.Send(new Key(), Money.Coins(0.5m));
    builder.SetChange(changeAddress.ScriptPubKey);

    // Set the fee rate
    var fallbackFeeRate = new FeeRate(Money.Satoshis(100), 1);
    var feeRate = tester.Client.GetFeeRate(1, fallbackFeeRate).FeeRate;
    builder.SendEstimatedFees(feeRate);
    /////

    var tx = builder.BuildTransaction(true);
    var result = client.Broadcast(tx);
    if(result.Success)
    {
        Console.WriteLine("Success!");
        break;
    }
    else if(result.RPCCode.HasValue && result.RPCCode.Value == RPCErrorCode.RPC_TRANSACTION_REJECTED)
    {
        Console.WriteLine("We probably got a conflict, let's try again!");
        continue;
    }
    else
    {
        Console.WriteLine($"Something is really wrong {result.RPCCode} {result.RPCCodeMessage} {result.RPCMessage}");
        // Do something!!!
    }
}
```

또 다른 일반적인 방법은 확인할 수 있는 이미 사용 된 아웃포인트(outpoint)의 전체 목록을 만드는 것입니다.
