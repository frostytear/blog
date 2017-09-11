---
title: 如何撰寫智能合約(Smart Contract)?(II)建立加密代幣
tags:
  - ethereum
  - solidity
  - javascript
date: 2017-09-12 00:47:31
---

[上一篇]中我們已寫好並部署完成了第一個智能合約，也驗證了合約確實可以運作。在閱讀完本文後，你將學會建立一個簡易的加密代幣🔒💵。本篇目的並非為了寫出一個安全可用的加密代幣，而是以介紹代幣合約的相關概念為主，所以對合約做了適當地簡化，好更容易被理解。

## 開發前的準備

延續上一篇的內容，在開發的過程中，我們將繼續使用`testrpc`[^1]工具在電腦上模擬智能合約所需的乙太坊區塊鏈測試環境。

首先確保已啟動testrpc，若尚未啟動，可以使用以下命令啟動

```
$ testrpc
...
```

這樣我們就可以開始建立加密代幣智能合約專案了。

## 建立一個代幣合約

在`contracts/`目錄下建立一個`SimpleToken.sol`檔案。也可以使用以下命令來產生檔案：

```sh
$ truffle create contract SimpleToken
```

`SimpleToken.sol`檔案內容如下：

```
pragma solidity ^0.4.11;

contract SimpleToken {
  uint256 INITIAL_SUPPLY = 88888;
  mapping(address => uint256) balances;

  function SimpleToken() {
    balances[msg.sender] = INITIAL_SUPPLY;
  }

  // transfer token for a specified address
  function transfer(address _to, uint256 _value) {
    balances[msg.sender] -= _value;
    balances[_to] += _value;
  }

  // Gets the balance of the specified address
  function balanceOf(address _owner) constant returns (uint256) {
    return balances[_owner];
  }
}

```

### 講解

```
pragma solidity ^0.4.11;
```

第一行指名目前使用的solidity版本，不同版本的solidity可能會編譯出不同的bytecode。

```
uint256 INITIAL_SUPPLY = 88888;
mapping(address => unit256) balances;
```

我們定義了初始代幣數目`INITIAL_SUPPLY`。這邊隨意選擇了一個吉祥數字`88888`。

我們用`mapping`來定義一個可以儲存鍵值對(key-value pair)的資料結構(類似Javascript中的`{"0xaabbccddeeff": 888}`)，同時也需要分別指定`address`作為鍵的型別，指定`uint256`作為值的型別。定義好後不能像Javascript那樣可隨時更改儲存的型別。

```
contract SimpleToken {
  function SimpleToken() {
    balances[msg.sender] = INITIAL_SUPPLY;
  }
}
```

和合約同名的`SimpleToken`函式，就是`SimpleToken`這個合約的建構函式(constructor)。函式中我們拿`msg.sender`當作key，`INITIAL_SUPPLY`當作值，將所有的初始代幣`INITIAL_SUPPLY`都指定給`msg.sender`帳號。
`msg`是一個全域(Global)物件[^2]，`msg.sender`表示用作呼叫當前函式的帳號。由於建構函式只有在合約部署時會被執行，因此這邊用到的`msg.sender`，也就代表著用來部署這個合約的帳號。

```
function transfer(address _to, uint256 _value) {
  balances[msg.sender] -= _value;
  balances[_to] += _value;
}
```

`transfer`函式定義了如何`轉帳`，只要指定要傳送的帳號與數目，就會從呼叫者手上把對應數目的代幣移轉到指定的帳號上。這樣寫當然是過度簡化，但就先這樣吧。

```
function balanceOf(address _owner) constant returns (uint256) {
  return balances[_owner];
}
```

`balanceOf`函式的作用，是讓使用者可以查詢任一帳號的餘額，透過傳入`_owner`帳號，可以查詢儲存在`balances`對照表中的值。

如此一來，我們已寫好一個新加密代幣🔒💵合約。

### 編譯與部署

在`migrations/`目錄下建立一個`3_deploy_token.js`檔案，內容如下：

```js
var SimpleToken = artifacts.require("SimpleToken");

module.exports = function(deployer) {
  deployer.deploy(SimpleToken);
};
```

現在執行compile與migrate命令

```sh
$ truffle compile
...
$ truffle migrate
Using network 'development'.

Running migration: 3_deploy_token.js
  Deploying HelloToken...
  ... 0x2c4659528c68b4e43d1edff6c989fba05e8e7e56cc4085d408426d339b4e9ba4
  SimpleToken: 0x352fa9aa18106f269d944935503afe57a00a9a0d
Saving successful migration to network...
  ... 0x1434c1b61e9719f410fc6090ce37c0ec141a1738aba278dd320738e4a5d229fa
Saving artifacts...
```

如此一來我們已將 SimpleToken代幣合約部署到testrpc上。

## 驗證

使用`truffle console`命令開啟console，輸入以下命令來驗證合約是否成功。

```sh
$ truffle console
> let contract
> SimpleToken.deployed().then(instance => contract = instance)
> contract.balanceOf(web3.eth.coinbase)
{ [String: '88888'] s: 1, e: 4, c: [ 88888 ] }
> contract.balanceOf(web3.eth.accounts[1])
{ [String: '0'] s: 1, e: 0, c: [ 0 ] }
> contract.transfer(web3.eth.accounts[1], 123)
...
> contract.balanceOf(web3.eth.coinbase)
{ [String: '88765'] s: 1, e: 4, c: [ 88765 ] }
> contract.balanceOf.call(web3.eth.accounts[1])
{ [String: '123'] s: 1, e: 2, c: [ 123 ] }
>
```

### 講解

```sh
> let contract
> SimpleToken.deployed().then(instance => contract = instance)
```


```sh
> contract.balanceOf(web3.eth.coinbase)
{ [String: '88888'] s: 1, e: 4, c: [ 88888 ] }
> contract.balanceOf(web3.eth.accounts[1])
{ [String: '0'] s: 1, e: 0, c: [ 0 ] }
```

還記得啟動testrpc後會產生10個帳號(Accounts)與每個帳號對應的私鑰(Private Key):key:嗎?。`web3.eth.coinbase` 代表預設的帳號，即10個帳號中的第一個帳號`web3.eth.accounts[0]`。在進行操作前，先呼叫`balanceOf`函式來查詢前兩個帳號所擁有的代幣餘額。可以看到第一個帳號(部署合約的帳號)裡有所有的代幣。

```
> contract.transfer(web3.eth.accounts[1], 123)
...
```

接著使用`transfer`函式來傳送123個代幣到第二個帳號`web3.eth.accounts[0]`。

```
> contract.balanceOf(web3.eth.coinbase)
{ [String: '88765'] s: 1, e: 4, c: [ 88765 ] }
> contract.balanceOf.call(web3.eth.accounts[1])
{ [String: '123'] s: 1, e: 2, c: [ 123 ] }
>
```

最後再次查詢原帳號和新帳號各自剩下的SimpleToken數目。發現SimpleToken的基本運作正常。


## 一堆安全漏洞，怎解?

這樣寫智能合約看起來並不困難，但其實我們尚未處理智能合約會預到的諸多安全問題。
例如`transfer`函式中沒有判斷帳戶中是否還有足夠轉出的餘額，沒有禁止傳入負數金額等等。有著一堆安全漏洞的合約，輕則執行失敗白花交易費用，嚴重則影響到合約中的代幣或以太幣。已有多起因為代幣或以太幣被轉走，造成ICO失敗的案例。

可以查看參考資料[^4]了解智能合約當前的最佳實現。

下一篇會接著介紹如何建立一份可以放到乙太幣錢包:purse:的加密代幣🔒💵合約，並將使用。

## 參考資料

* [1] https://github.com/ethereumjs/testrpc
* [2] Units and Globally Available Variables http://solidity.readthedocs.io/en/develop/units-and-global-variables.html
* [3] An Ethereum Hello World Smart Contract for Beginners [part 1](http://www.talkcrypto.org/blog/2017/04/17/an-ethereum-hello-world-smart-contract-for-beginners-part-1/), [part 2](http://www.talkcrypto.org/blog/2017/04/22/an-ethereum-hello-world-smart-contract-for-beginners-part-2/)
* [4] https://blog.zeppelin.solutions/onward-with-ethereum-smart-contract-security-97a827e47702