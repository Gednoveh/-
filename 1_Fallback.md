1_Fallback
=
本关要求
--
 
    1.you claim ownership of the contract  
    2.you reduce its balance to 0  
    
合约代码
--
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.6.0;
    
    import '@openzeppelin/contracts/math/SafeMath.sol';
    
    contract Fallback {
      using SafeMath for uint256;
      mapping(address => uint) public contributions;
      address payable public owner;

      constructor() public {
        owner = msg.sender;
        contributions[msg.sender] = 1000 * (1 ether);
      }

      modifier onlyOwner {
            require(
                msg.sender == owner,
                "caller is not the owner"
            );
            _;
        }

      function contribute() public payable {
        require(msg.value < 0.001 ether);
        contributions[msg.sender] += msg.value;
        if(contributions[msg.sender] > contributions[owner]) {
          owner = msg.sender;
        }
      }

      function getContribution() public view returns (uint) {
        return contributions[msg.sender];
      }

      function withdraw() public onlyOwner {
        owner.transfer(address(this).balance);
      }

      receive() external payable {
        require(msg.value > 0 && contributions[msg.sender] > 0);
        owner = msg.sender;
      }
    }
分析
--

  首先关注<code>contribute()</code>方法，调用该方法可以单次支付小于0.001ETH，当sender的累计值大于当前owner的contributions值时，sender会成为owner  
  查看当前合约的构造函数，当前owner的contributions值为<code>1000 * (1 ether)</code>,所以我们需要调用<code>contribute()</code>最少<code>10^6</code>次才能成为owner，显然不合理  
  继续向下查看<code>receive()</code>方法，当sender发送任意值的ETH，且contributions不为0是，sender会成为owner  
  在Solidity中，一个合约最多有一个<code>receive()</code>回调函数，在对合约转账时（例如通过<code>transfer()</code>、<code>send()</code>或<code>call()</code>）会触发receive()回调  
  在成为owner后调用<code>withdraw()</code>方法可清空合约余额  
