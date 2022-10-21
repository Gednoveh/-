5_Token
==

本关要求
--

The goal of this level is for you to hack the basic token contract below.

You are given 20 tokens to start with and you will beat the level if you somehow manage to get your hands on any additional tokens. Preferably a very large amount of tokens.

合约代码
--

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.6.0;

    contract Token {

      mapping(address => uint) balances;
      uint public totalSupply;

      constructor(uint _initialSupply) public {
        balances[msg.sender] = totalSupply = _initialSupply;
      }

      function transfer(address _to, uint _value) public returns (bool) {
        require(balances[msg.sender] - _value >= 0);
        balances[msg.sender] -= _value;
        balances[_to] += _value;
        return true;
      }

      function balanceOf(address _owner) public view returns (uint balance) {
        return balances[_owner];
      }
    }

合约分析
--

查看转账方法<code>transfer</code>发现，对于<code>uint _value</code>的数值操作没有做溢出检查，会发生上溢出和下溢出
上溢出：对于8位无符号整型uint a = 255，a + 1 = 0
下溢出：对于8位无符号整型uint a = 0，a - 1 = 255

攻击
--

查询当前账户余额，为20个token  

![image](https://user-images.githubusercontent.com/35074461/197104266-8e5ee32e-eaaf-460e-8895-34b45f6a540b.png)

对合约地址转账21个token，此时发生下溢出，再次查看账户余额，发现获得大量Token  

![image](https://user-images.githubusercontent.com/35074461/197120041-236908a9-3c2b-490e-a6b9-44e2690cbc46.png)  

PS:
--

Overflows are very common in solidity and must be checked for with control statements such as:  

    if(a + c > a) {
      a = a + c;
    }

An easier alternative is to use OpenZeppelin's SafeMath library that automatically checks for overflows in all the mathematical operators. The resulting code looks like this:  

    a = a.add(c);

If there is an overflow, the code will revert.  
