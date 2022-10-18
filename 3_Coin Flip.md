3_CoinFlip
=
本关要求
--

  1.This is a coin flipping game where you need to build up your winning streak by guessing the outcome of a coin flip. To complete this level you'll need to use your psychic abilities to guess the correct outcome 10 times in a row.  

合约代码
--
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.6.0;

    import '@openzeppelin/contracts/math/SafeMath.sol';

    contract CoinFlip {

      using SafeMath for uint256;
      uint256 public consecutiveWins;
      uint256 lastHash;
      uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

      constructor() public {
        consecutiveWins = 0;
      }

      function flip(bool _guess) public returns (bool) {
        uint256 blockValue = uint256(blockhash(block.number.sub(1)));

        if (lastHash == blockValue) {
          revert();
        }

        lastHash = blockValue;
        uint256 coinFlip = blockValue.div(FACTOR);
        bool side = coinFlip == 1 ? true : false;

        if (side == _guess) {
          consecutiveWins++;
          return true;
        } else {
          consecutiveWins = 0;
          return false;
        }
      }
    }  
分析
--
本关要求我们预测对10次硬币的正反  
查看<code>flip</code>方法可知，<code>coinFlip</code>的值决定硬币正反，<code>coinFlip</code>的值通过<code>blockValue.div(FACTOR)</code>计算得到  
<code>blockValue</code>的值为进行预测时当前区块的<code>blockhash</code>  
因为<code>FACTOR</code>的值为常量，所以如果能提前预知<code>blockValue</code>的值，就能预测硬币的正反  
因为有<code>if (lastHash == blockValue)</code>的判断，同一个区块的合约只能提交一次。所以我们采用合约调用的方法，先在我们编写的合约内计算硬币的正反，再调用本关合约，提交预测的结果  
