## Some gas optimziation tricks in Solidity
#### Author: Mo Ashouri 
#### Contact: ashourics@protonmail.com



I explain some trick to reduce gas consumotions in your solidity smart contracts by example.

The tricks as the followings:
- Using calldata
- Short-circuiting 
- Loop increment
- Loading local state variables into local variable (memory)
- Loading array slots into memory


Look the GasOptmized contract and my comments in the code:


```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.7;

contract GasOptimized{

    uint public sum;

    function Is_Even(uint[] memory numbers) external {
        for(uint i=0; i< numbers.length; i+=1){
            bool isEven=numbers[i] % 2 ==0;
            bool isLess99 = numbers[i] < 99;
           if(isEven && isLess99){
               sum+=numbers[i];
           }
        }

    }
   ````
   
  Now look at the transaction cost --> 50970 before the gas optimization
  
  ### Now change memory to calldata in the function argument

   
  ```
     function Is_Even(uint[] calldata numbers) external { // calldata
        for(uint i=0; i< numbers.length; i+=1){
            bool isEven=numbers[i] % 2 ==0;
            bool isLess99 = numbers[i] < 99;
           if(isEven && isLess99){
               sum+=numbers[i];
           }
        }

    }
    
    ```
    
    
    Now if you compile and deploy the contract and execute the Is_even function the transaction cost is 49141 gas
    
    ### Short-circuiting trick!

     
  ```
        function Is_Even(uint[] calldata numbers) external {
           
            uint _sum=sum;
            for(uint i=0; i< numbers.length; i+=1){
               // bool isEven=numbers[i] % 2 ==0;    --> short circuiting
               // bool isLess99 = numbers[i] < 99;   --> short circuiting
               if(numbers[i] % 2 ==0 && numbers[i] < 99){
                   _sum+=numbers[i];
               }
            }
            sum=_sum;
    
        }
       
  ```
  
 Now after the short-circuiting optimization we got the transaction cost 48612

    ### Loop increment trick


    
    ```
        function Is_Even(uint[] calldata numbers) external {
            uint _sum=sum;
            for(uint i=0; i< numbers.length; ++i){ // --> loop increment
               // bool isEven=numbers[i] % 2 ==0;
               // bool isLess99 = numbers[i] < 99;
               if(numbers[i] % 2 ==0 && numbers[i] < 99){
                   _sum+=numbers[i];
               }
            }
            sum=_sum;
    
        }
   
   ```
   
   Now the transaction cost is 48222 
   
   ### Loading local state variables into local variable (memory)


```

   function Is_Even(uint[] calldata numbers) external {
            uint _sum=sum; //  < -- check here
            uint _len=numbers.length;
            for(uint i=0; i< _len; ++i){ 
            
               // bool isEven=numbers[i] % 2 ==0;
               // bool isLess99 = numbers[i] < 99;
               if(numbers[i] % 2 ==0 && numbers[i] < 99){
                   _sum+=numbers[i];
               }
            }
            sum=_sum; // and here <---
    
        }
  
  ```
  

 Now the transaction cost is --> 48187 


### Loading array slots into memory

    
 ```
           function Is_Even(uint[] calldata numbers) external {
            //load the local state into the memory
            uint _sum=sum;
            uint _len=numbers.length;
            for(uint i=0; i< _len; ++i){ // --> loop increment
               // bool isEven=numbers[i] % 2 ==0;
               // bool isLess99 = numbers[i] < 99;
               uint num=numbers[i]; // caching an array slot into memory
               if(num % 2 ==0 && num < 99){
                   _sum+=num;
               }
            }
            sum=_sum;
    
        }

```

Now the transaction cost is -->  48025 
           
       

