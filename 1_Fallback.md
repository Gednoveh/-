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

  首先关注<code>contribute()</code>函数，调用该方法可以单次支付小于0.001ETH，当sender的累计值大于当前owner的contributions值时，sender会成为owner
  查看当前合约的构造函数，当前owner的contributions值为<coed>1000 * (1 ether)</code>,所以我们需要调用<code>contribute()</code>最少10^6次才能成为owner，显然合理
