# コンテキスト

このリポジトリは、以下のブログ記事の基礎となっています：https://blog.sigmaprime.io/solidity-security.html

これは、Mastering Ethereumブックのスマートコントラクトセキュリティセクションの基礎となっています：https://github.com/ethereumbook/ethereumbook

# 概要

この投稿は、将来の開発者が歴史を繰り返すことを防ぐために、Solidity開発者によって過去に犯された間違いを比較的詳細かつ最新の紹介記事として提供することを目的としています。

まだ初期段階ではありますが、Solidityは広く採用され、今日私たちが見るEthereumスマートコントラクトの多くのバイトコードをコンパイルするために使用されています。開発者とユーザーの両方が、言語とEVMの微妙な点を発見する中で、多くの厳しい教訓を学んできました。この投稿は、将来の開発者が歴史を繰り返すことを防ぐために、Solidity開発者によって過去に犯された間違いを比較的詳細かつ最新の紹介記事として提供することを目的としています。

*これは急速に変化している分野であるため、この投稿を[Github](https://github.com/sigp/solidity-security-blog)に置いて、誰でもこの投稿に貢献したり、私が確実に犯したエラーについての問題を提出したりすることを奨励しています。*

# 目次

#### [1. リエントランシー](#reentrancy)
 * [脆弱性](#re-vuln)
 * [予防技術](#re-prev)
 * [実例：The DAO](#re-example)

#### [2. 算術オーバーフロー/アンダーフロー](#ouflow)
 * [脆弱性](#ou-vuln)
 * [予防技術](#ou-prev)
 * [実例：PoWHCとBatch Transfer Overflow (CVE-2018-10299)](#ou-example)

#### [3. 予期しないイーサ](#ether)
 * [脆弱性](#ether-vuln)
 * [予防技術](#ether-prev)
 * [実例：不明](#ether-example)

#### [4. Delegatecall](#delegatecall)
 * [脆弱性](#dc-vuln)
 * [予防技術](#dc-prev)
 * [実例：Parity Multisig Wallet（第2のハック）](#dc-example)

#### [5. デフォルト可視性](#visibility)
 * [脆弱性](#visibility-vuln)
 * [予防技術](#visibility-prev)
 * [実例：Parity MultiSig Wallet（第1のハック）](#visibility-example)

#### [6. エントロピーの錯覚](#entropy)
 * [脆弱性](#entropy-vuln)
 * [予防技術](#entropy-prev)
 * [実例：PRNGコントラクト](#entropy-example)

#### [7. 外部コントラクト参照](#contract-reference)
 * [脆弱性](#cr-vuln)
 * [予防技術](#cr-prev)
 * [実例：リエントランシーハニーポット](#cr-example)

#### [8. ショートアドレス/パラメータ攻撃](#short-address)
 * [脆弱性](#short-vuln)
 * [予防技術](#short-prev)
 * [実例：不明](#short-example)

#### [9. 未チェックのCALL戻り値](#unchecked-calls)
 * [脆弱性](#unchecked-calls-vuln)
 * [予防技術](#unchecked-calls-prev)
 * [実例：EtherpotとKing of the Ether](#unchecked-calls-example)

#### [10. 競合状態/フロントランニング](#race-conditions)
 * [脆弱性](#race-conditions-vuln)
 * [予防技術](#race-conditions-prev)
 * [実例：ERC20とBancor](#race-conditions-example)

#### [11. サービス拒否（DOS）](#dos)
 * [脆弱性](#dos-vuln)
 * [予防技術](#dos-prev)
 * [実例：GovernMental](#dos-example)

#### [12. ブロックタイムスタンプ操作](#block-timestamp)
 * [脆弱性](#block-timestamp-vuln)
 * [予防技術](#block-timestamp-prev)
 * [実例：GovernMental](#block-timestamp-example)

#### [13. コンストラクタの注意点](#constructors)
 * [脆弱性](#constructors-vuln)
 * [予防技術](#constructors-prev)
 * [実例：Rubixi](#constructors-example)

#### [14. 未初期化のストレージポインタ](#storage)
 * [脆弱性](#storage-vuln)
 * [予防技術](#storage-prev)
 * [実例：ハニーポット：OpenAddressLotteryとCryptoRoulette](#storage-example)

#### [15. 浮動小数点と精度](#precision)
 * [脆弱性](#precision-vuln)
 * [予防技術](#precision-prev)
 * [実例：Ethstick](#precision-example)

#### [16. tx.origin認証](#tx-origin)
 * [脆弱性](#tx-origin-vuln)
 * [予防技術](#tx-origin-prev)
 * [実例：不明](#tx-origin-example)

## [Ethereumの特異性](#ethereum-quirks)
* [キーレスイーサ](#keyless-eth)
* [ワンタイムアドレス](#one-time-addresses)
* [単一トランザクションエアドロップ](#single-transaction-airdrops)

## [興味深い暗号関連ハック/バグのリスト](#hacks)


## 参考文献/さらなる読書リスト

- [Ethereum Wiki - Safety](https://github.com/ethereum/wiki/wiki/Safety)
- [Solidity Docs - Security Considerations](solidity.readthedocs.io/en/latest/security-considerations.html)
- [Consensus - Ethereum Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices)
- [History of Ethereum Security Vulnerabilities, Hacks and Their Fixes](https://applicature.com/blog/history-of-ethereum-security-vulnerabilities-hacks-and-their-fixes)
- [Decentralized Application Security Project (DASP) Top 10 of 2018](http://www.dasp.co/)
- [A Survey of attacks on Ethereum Smart Contracts](https://eprint.iacr.org/2016/1007.pdf)
- [Ethereum Smart Contract Security](https://medium.com/cryptronics/ethereum-smart-contract-security-73b0ede73fa8)
- [Lessons Learnt from the Underhanded Solidity Contest](https://medium.com/@chriseth/lessons-learnt-from-the-underhanded-solidity-contest-8388960e09b1)

<h2 id="reentrancy"><span id="SP-1">1. リエントランシー</span></h2>

Ethereumスマートコントラクトの機能の1つは、他の外部コントラクトのコードを呼び出して利用する能力です。コントラクトは通常イーサも扱い、様々な外部ユーザーアドレスにイーサを送信することがよくあります。外部コントラクトを呼び出したり、アドレスにイーサを送信したりする操作には、コントラクトが外部呼び出しを行う必要があります。これらの外部呼び出しは、攻撃者によってハイジャックされる可能性があり、攻撃者はコントラクトに（フォールバック関数などを通じて）さらにコードを実行させ、自身への呼び出しを含めることができます。このようにコード実行がコントラクトに「再入」します。この種の攻撃は、悪名高いDAOハックで使用されました。

リエントランシー攻撃についてさらに読むには、[スマートコントラクトに対するリエントランシー攻撃](https://medium.com/@gus_tavo_guim/reentrancy-attack-on-smart-contracts-how-to-identify-the-exploitable-and-an-example-of-an-attack-4470a2d8dfe4)と[Consensus - Ethereum Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/known_attacks/#reentrancy)を参照してください。

<h3 id="re-vuln">脆弱性</h3>

この攻撃は、コントラクトが未知のアドレスにイーサを送信する場合に発生する可能性があります。攻撃者は、[フォールバック関数](https://solidity.readthedocs.io/en/latest/contracts.html?highlight=fallback#fallback-function)に悪意のあるコードを含む外部アドレスのコントラクトを慎重に構築することができます。したがって、コントラクトがこのアドレスにイーサを送信すると、悪意のあるコードが呼び出されます。通常、悪意のあるコードは脆弱なコントラクト上の関数を実行し、開発者が予期していない操作を実行します。「リエントランシー」という名前は、外部の悪意のあるコントラクトが脆弱なコントラクト上の関数を呼び戻し、脆弱なコントラクト上の任意の場所でコード実行に「再入」するという事実に由来しています。

これを明確にするために、イーサリアムの金庫として機能し、預金者が週に1イーサのみを引き出すことを許可する単純な脆弱なコントラクトを考えてみましょう。

EtherStore.sol:
```solidity
contract EtherStore {

    uint256 public withdrawalLimit = 1 ether;
    mapping(address => uint256) public lastWithdrawTime;
    mapping(address => uint256) public balances;

    function depositFunds() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdrawFunds (uint256 _weiToWithdraw) public {
        require(balances[msg.sender] >= _weiToWithdraw);
        // 引き出しを制限する
        require(_weiToWithdraw <= withdrawalLimit);
        // 引き出しが許可される時間を制限する
        require(now >= lastWithdrawTime[msg.sender] + 1 weeks);
        require(msg.sender.call.value(_weiToWithdraw)());
        balances[msg.sender] -= _weiToWithdraw;
        lastWithdrawTime[msg.sender] = now;
    }
 }
```

このコントラクトには2つのパブリック関数があります。`depositFunds()`と`withdrawFunds()`です。`depositFunds()`関数は単に送信者の残高を増加させます。`withdrawFunds()`関数は、送信者が引き出すweiの量を指定することを許可します。これは、要求された引き出し量が1イーサ未満であり、過去1週間以内に引き出しが行われていない場合にのみ成功します。それともそうでしょうか？...

脆弱性は、ユーザーに要求されたイーサの量を送信する\[17\]行にあります。次のようなコントラクトを作成する悪意のある攻撃者を考えてみましょう。

Attack.sol:
```solidity
import "EtherStore.sol";

contract Attack {
  EtherStore public etherStore;

  // コントラクトアドレスでetherStore変数を初期化する
  constructor(address _etherStoreAddress) {
      etherStore = EtherStore(_etherStoreAddress);
  }

  function pwnEtherStore() public payable {
      // 最も近いイーサに攻撃する
      require(msg.value >= 1 ether);
      // depositFunds()関数にイーサを送信する
      etherStore.depositFunds.value(1 ether)();
      // 魔法を開始する
      etherStore.withdrawFunds(1 ether);
  }

  function collectEther() public {
      msg.sender.transfer(this.balance);
  }

  // フォールバック関数 - 魔法が起こる場所
  function () payable {
      if (etherStore.balance > 1 ether) {
          etherStore.withdrawFunds(1 ether);
      }
  }
}
```

この悪意のあるコントラクトが`EtherStore`コントラクトをどのように悪用できるかを見てみましょう。攻撃者は上記のコントラクトを作成し（例えばアドレス`0x0...123`で）、コンストラクタパラメータとして`EtherStore`のコントラクトアドレスを指定します。これにより、攻撃したいコントラクトを指すパブリック変数`etherStore`が初期化されます。

攻撃者は次に`pwnEtherStore()`関数を呼び出し、いくらかのイーサ（1イーサ以上）、例えばこの例では`1イーサ`を送ります。この例では、他の多くのユーザーがこのコントラクトにイーサを預けており、現在の残高が`10イーサ`であると仮定します。次のことが起こります：

1. **Attack.sol - Line \[15\]** - `EtherStore`コントラクトの`depositFunds()`関数が`msg.value`が`1イーサ`（および多くのガス）で呼び出されます。送信者（`msg.sender`）は私たちの悪意のあるコントラクト（`0x0...123`）になります。したがって、`balances[0x0..123] = 1 ether`となります。

2. **Attack.sol - Line \[17\]** - 悪意のあるコントラクトは、パラメータとして`1イーサ`を指定して`EtherStore`コントラクトの`withdrawFunds()`関数を呼び出します。これは、以前の引き出しを行っていないため、`EtherStore`コントラクトの\[12\]-\[16\]行のすべての要件を満たします。

3. **EtherStore.sol - Line \[17\]** - コントラクトは`1イーサ`を悪意のあるコントラクトに送り返します。

4. **Attack.sol - Line \[25\]** - 悪意のあるコントラクトに送られたイーサはフォールバック関数を実行します。

5. **Attack.sol - Line \[26\]** - `EtherStore`コントラクトの総残高は`10イーサ`で、現在は`9イーサ`なので、このif文は通過します。

6. **Attack.sol - Line \[27\]** - フォールバック関数は`EtherStore`の`withdrawFunds()`関数を再び呼び出し、`EtherStore`コントラクトに「再入」します。

7. **EtherStore.sol - Line \[11\]** - この`withdrawFunds()`への2回目の呼び出しでは、\[18\]行がまだ実行されていないため、残高はまだ`1イーサ`です。したがって、`balances[0x0..123] = 1 ether`のままです。これは`lastWithdrawTime`変数についても同様です。再び、すべての要件を満たします。

8. **EtherStore.sol - Line \[17\]** - さらに`1イーサ`を引き出します。

9. **ステップ4-8が繰り返されます** - `Attack.sol`の\[26\]行で指示されているように、`EtherStore.balance >= 1`になるまで。

10. **Attack.sol - Line \[26\]** - `EtherStore`コントラクトに1（またはそれ以下の）イーサが残ると、このif文は失敗します。これにより、`EtherStore`コントラクトの\[18\]と\[19\]行が（`withdrawFunds()`関数への各呼び出しに対して）実行されることが可能になります。

11. **EtherStore.sol - Lines \[18\] and \[19\]** - `balances`と`lastWithdrawTime`マッピングが設定され、実行が終了します。

最終的な結果として、攻撃者は単一のトランザクションで瞬時に`EtherStore`コントラクトからすべての（1を除く）イーサを引き出しています。

<h3 id="re-prevention">予防技術</h3>

スマートコントラクトでの潜在的なリエントランシー脆弱性を回避するための一般的な技術がいくつかあります。1つ目は、可能な限り組み込みの[transfer()](http://solidity.readthedocs.io/en/latest/units-and-global-variables.html#address-related)関数を使用して外部コントラクトにイーサを送信することです。transfer関数は外部呼び出しに`2300ガス`のみを送信し、これは宛先アドレス/コントラクトが別のコントラクトを呼び出す（つまり、送信元コントラクトに再入する）のに十分ではありません。

2つ目の技術は、状態変数を変更するすべてのロジックがコントラクトからイーサが送出される前（または任意の外部呼び出しの前）に行われるようにすることです。`EtherStore`の例では、`EtherStore.sol`の\[18\]行と\[19\]行を\[17\]行の前に配置する必要があります。未知のアドレスへの外部呼び出しを実行するコードを、ローカライズされた関数やコード実行の最後の操作として配置することは良い習慣です。これは[checks-effects-interactions](http://solidity.readthedocs.io/en/latest/security-considerations.html#use-the-checks-effects-interactions-pattern)パターンとして知られています。

3つ目の技術は、ミューテックスを導入することです。つまり、コード実行中にコントラクトをロックし、リエントランシー呼び出しを防止する状態変数を追加することです。

これらすべての技術を適用すると（3つすべては不要ですが、デモンストレーションのために行われています）、`EtherStore.sol`は以下のようなリエントランシーのないコントラクトになります：
```solidity
contract EtherStore {

    // ミューテックスを初期化
    bool reEntrancyMutex = false;
    uint256 public withdrawalLimit = 1 ether;
    mapping(address => uint256) public lastWithdrawTime;
    mapping(address => uint256) public balances;

    function depositFunds() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdrawFunds (uint256 _weiToWithdraw) public {
        require(!reEntrancyMutex);
        require(balances[msg.sender] >= _weiToWithdraw);
        // 引き出しを制限する
        require(_weiToWithdraw <= withdrawalLimit);
        // 引き出しが許可される時間を制限する
        require(now >= lastWithdrawTime[msg.sender] + 1 weeks);
        balances[msg.sender] -= _weiToWithdraw;
        lastWithdrawTime[msg.sender] = now;
        // 外部呼び出しの前にリエントランシーミューテックスを設定
        reEntrancyMutex = true;
        msg.sender.transfer(_weiToWithdraw);
        // 外部呼び出しの後にミューテックスを解放
        reEntrancyMutex = false;
    }
 }
```

<h3 id="re-example">実例：The DAO</h3>

[The DAO](https://en.wikipedia.org/wiki/The_DAO_(organization))（分散型自律組織）は、Ethereumの初期開発で発生した主要なハックの1つでした。当時、このコントラクトには1億5000万ドル以上のUSDが保持されていました。リエントランシーは、最終的にEthereum Classic（ETC）を生み出したハードフォークにつながった攻撃において主要な役割を果たしました。DAOエクスプロイトの優れた分析については、[Phil Daianの投稿](http://hackingdistributed.com/2016/06/18/analysis-of-the-dao-exploit/)を参照してください。

<h2 id="ouflow"><span id="SP-2">2. 算術オーバーフロー/アンダーフロー</span></h2>

Ethereum Virtual Machine（EVM）は整数に対して固定サイズのデータ型を指定します。これは、整数変数が特定の範囲の数値しか表現できないことを意味します。例えば、`uint8`は\[0,255\]の範囲の数値しか格納できません。`uint8`に`256`を格納しようとすると、結果は`0`になります。注意が払われない場合、Solidityのコントラクトは、ユーザー入力が未チェックであり、それらを格納するデータ型の範囲外の数値を生成する計算が実行される場合、悪用される可能性があります。

算術オーバーフロー/アンダーフローについてさらに読むには、[How to Secure Your Smart Contracts](https://medium.com/loom-network/how-to-secure-your-smart-contracts-6-solidity-vulnerabilities-and-how-to-avoid-them-part-1-c33048d4d17d)、[Ethereum Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/known_attacks/#integer-overflow-and-underflow)、および[Ethereum, Solidity and integer overflows: programming blockchains like 1970](https://randomoracle.wordpress.com/2018/04/27/ethereum-solidity-and-integer-overflows-programming-blockchains-like-1970/)を参照してください。

<h3 id="ou-vuln">脆弱性</h3>

オーバーフロー/アンダーフローは、固定サイズの変数が変数のデータ型の範囲外の数値（またはデータ）を格納する必要がある操作が実行されるときに発生します。

例えば、値として`0`を格納する`uint8`（8ビットの符号なし整数、つまり正の数のみ）変数から`1`を引くと、数値`255`になります。これはアンダーフローです。`uint8`の範囲外の数値を割り当てたため、結果は「ラップアラウンド」し、`uint8`が格納できる最大の数値を与えます。同様に、`2^8=256`を`uint8`に追加すると、`uint`の長さ全体をラップアラウンドしたため、変数は変更されません（数学者のために、これは三角関数の角度に$2\pi$を加えるのと似ています、$\sin(x) = \sin(x+2\pi)$）。データ型の範囲より大きい数値を追加することをオーバーフローと呼びます。明確にするために、現在ゼロ値を持つ`uint8`に`257`を追加すると、数値`1`になります。固定型変数が循環的であると考えるのが時々有益です。ゼロから始めて、格納可能な最大数より上の数値を追加すると再びゼロから始まり、ゼロの場合も同様です（最大数からカウントダウンを始めます）。

このような数値の注意点により、攻撃者はコードを誤用し、予期しないロジックフローを作成することができます。例として、以下のタイムロックコントラクトを考えてみましょう。

TimeLock.sol:
```solidity
contract TimeLock {

    mapping(address => uint) public balances;
    mapping(address => uint) public lockTime;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
        lockTime[msg.sender] = now + 1 weeks;
    }

    function increaseLockTime(uint _secondsToIncrease) public {
        lockTime[msg.sender] += _secondsToIncrease;
    }

    function withdraw() public {
        require(balances[msg.sender] > 0);
        require(now > lockTime[msg.sender]);
        uint transferValue = balances[msg.sender];
        balances[msg.sender] = 0;
        msg.sender.transfer(transferValue);
    }
}
```

このコントラクトは、ユーザーがイーサをコントラクトに預け、少なくとも1週間ロックされるタイムボールトのように機能するように設計されています。ユーザーは待機時間を1週間以上に延長することを選択できますが、一度預けると、ユーザーは少なくとも1週間はイーサが安全にロックされていることを確信できます。それとも確信できるでしょうか？...

ユーザーが秘密鍵を引き渡すことを強制される場合（人質状況を考えてください）、このようなコントラクトは、短期間でイーサが取得できないようにするのに便利かもしれません。ユーザーがこのコントラクトに`100イーサ`をロックし、攻撃者に鍵を渡した場合、攻撃者はオーバーフローを使用して、`lockTime`に関係なくイーサを受け取ることができます。

攻撃者は、現在保持している鍵のアドレスの現在の`lockTime`を決定することができます（これはパブリック変数です）。これを`userLockTime`と呼びましょう。彼らは`increaseLockTime`関数を呼び出し、引数として`2^256 - userLockTime`を渡すことができます。この数値は現在の`userLockTime`に追加され、オーバーフローを引き起こし、`lockTime[msg.sender]`を`0`にリセットします。攻撃者は単に`withdraw`関数を呼び出して報酬を得ることができます。

別の例を見てみましょう。これは[Ethernaut Challanges](https://github.com/OpenZeppelin/ethernaut)からのものです。

**ネタバレ注意：** *Ethernautチャレンジをまだ行っていない場合、これはレベルの1つの解決策を提供します*。

```solidity
pragma solidity ^0.4.18;

contract Token {

  mapping(address => uint) balances;
  uint public totalSupply;

  function Token(uint _initialSupply) {
    balances[msg.sender] = totalSupply = _initialSupply;
  }

  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }

  function balanceOf(address _owner) public constant returns (uint balance) {
    return balances[_owner];
  }
}
```

これは、参加者がトークンを移動できるようにする`transfer()`関数を採用したシンプルなトークンコントラクトです。このコントラクトのエラーがわかりますか？

欠陥は`transfer()`関数にあります。\[13\]行のrequire文は、アンダーフローを使用してバイパスすることができます。残
