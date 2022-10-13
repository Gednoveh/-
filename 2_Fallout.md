2_Fallout
=
本关要求
--

  1.Claim ownership of the contract below to complete this level.  

合约代码
--

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.6.0;

    import '@openzeppelin/contracts/math/SafeMath.sol';

    contract Fallout {

      using SafeMath for uint256;
      mapping (address => uint) allocations;
      address payable public owner;


      /* constructor */
      function Fal1out() public payable {
        owner = msg.sender;
        allocations[owner] = msg.value;
      }

      modifier onlyOwner {
              require(
                  msg.sender == owner,
                  "caller is not the owner"
              );
              _;
          }

      function allocate() public payable {
        allocations[msg.sender] = allocations[msg.sender].add(msg.value);
      }

      function sendAllocation(address payable allocator) public {
        require(allocations[allocator] > 0);
        allocator.transfer(allocations[allocator]);
      }

      function collectAllocations() public onlyOwner {
        msg.sender.transfer(address(this).balance);
      }

      function allocatorBalance(address allocator) public view returns (uint) {
        return allocations[allocator];
      }
    }  
分析
--
查看Fallout函数发现只要发起一笔转账就可成为owner  
仔细查看，发现函数名为Fal1out  
![image](https://user-images.githubusercontent.com/35074461/195491345-86bf325e-ddad-4c12-9906-42947ee0706d.png)  
交互
--
<code>await contract.Fal1out({value:1})</code>向合约发起一笔转账  


    
