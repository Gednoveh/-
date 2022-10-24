6_Delegation
==

本关要求
--
The goal of this level is for you to claim ownership of the instance you are given.

合约代码
--

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.6.0;

    contract Delegate {

      address public owner;

      constructor(address _owner) public {
        owner = _owner;
      }

      function pwn() public {
        owner = msg.sender;
      }
    }

    contract Delegation {

      address public owner;
      Delegate delegate;

      constructor(address _delegateAddress) public {
        delegate = Delegate(_delegateAddress);
        owner = msg.sender;
      }

      fallback() external {
        (bool result,) = address(delegate).delegatecall(msg.data);
        if (result) {
          this;
        }
      }
    }
    
合约分析
--

改变owner的代码在<code>Delegate</code>合约的<code>pwn</code>函数中，通过<code>Delegation</code>合约中的fallback回调函数中的<code>delegatecall方法</code>调用。  
查看<code>delegatecall</code>在官方文档中的定义  

      There exists a special variant of a message call, named delegatecall which is identical to a message call apart from 
      the fact that the code at the target address is executed in the context (i.e. at the address) of the calling contract 
      and msg.sender and msg.value do not change their values.

      This means that a contract can dynamically load code from a different address at runtime. Storage, current address 
      and balance still refer to the calling contract, only the code is taken from the called address.
call和delegatecall是两种常见的外部函数调用方法，两种方法的区别为：  

call：执行环境为被调用者的环境，相当于代码在被调用的合约环境中运行

delegatecall：执行环境为调用者的环境，相当于代码在调用者的合约环境中运行

攻击
--
https://view.inews.qq.com/a/20220526A09KFZ00
