---
layout: post
title: EVM Assembly
tags: [solidity]
---

## assembly
assembler는 assembly language를 machine code로 변경한다. assembly는 assembler language라고도 부른다. Intel x86에는 1503개의 machine instruction이 존재한다. 이러한 machine instruction을 opcode라고도 부른다.

## EVM assembly

EVM에도 자신만의 [instruction set](https://github.com/ethereum/go-ethereum/blob/1d57f22d58f91bcb4e1052f82cc4d6532cc611fc/core/vm/opcodes.go#L222-L389)이 존재한다. 이러한 instruction들은 모두 Solidity라는 컨트랙트 언어로 추상화 된다.

Solidity 언어는 inline assembly를 지원한다.
~~~ javascript
contract C {
    function doSomething () public {
        assembly {
            // start writing evm assembler language
        }
    }
}
~~~

EVM은 stack machine이다. 이 데이터 자료형에서는 오직 PUSH와 POP만 수행한다. stack은 Last in, First out 구조다. stack machine에서 명령어는 하나 또는 그 이상의 값을 stack에서 pop하여 연산하고 결과값을 다시 push한다.
~~~text
a + b // Standard Notation
a b add // Reverse Polish Notation
~~~

### Everything is an uint in Assembly
efficiency를 위해, EVM Assembly는 모든 값들을 256-bit 숫자로 취급한다.

Solidity는 256 bit보다 narrow한 타입을 스스로 인지하기 때문에, 해당 타입의 값을 memory에 쓰거나 comparison을 수행하기 전에 higher-order bits를 직접 clean하게 만든다. **이에 반해 assembly에서는 이를 위해 직접 higher-order bits를 clean하게 만들어줘야 할 수도 있다.**

{% include image.html
           img="/images/test1.png"
           title="title for image"
           caption="그림 1. higher-order and lower-order bits"
           url="https://i.imgur.com/WjaEO1u.png"
%}

## Why use Assembly in Solidity ?

### Fine-grained control
opcode를 사용하는 EVM과 직접적으로 상호작용할 수 있다. 이를 통해 프로그램이 하고자 하는 일을 세밀하게 컨트롤할 수 있다.

assembly는 Solidity 문법만으로는 할 수 없는 것들을 할 수 없는 것을 가능하게 해준다. (e.g. pointing to a specific memory slot.)

### Gas cost reduction
assembly를 사용해서 연산에 필요한 가스를 줄일 수 있다.

### Enhanced features
Solidity 언어로 구성할 수 없는 것을 assembly로 구성할 수 있는 것들이 있다.

1. inline assembly는 string과 bytes와 같은 data type의 값을 단 하나의 operation으로 32 bytes(1 word)만큼 읽어올 수 있다. 이러한 점을 이용해 32 bytes 단위로 값을 비교 할 수 있다. 그렇게 하지 않으면 byte 단위로 이를 비교해야 한다. e.g. [solidity-stringutils](https://github.com/Arachnid/solidity-stringutils/blob/master/src/strings.sol#L198)
2. 몇몇 operation은 Solidity 언어에 구현되어 있지 않다. 예를 들어 `sha3` opcode는 memory의 byte range를 이용해 hash를 한다. 반면 solidity는 이를 위해 항상 string을 사용한다. 따라서 string을 hash하기 위해서는 비싼 string copying operation이 필요하다. inline assembly를 사용하면 string을 pass해서 바로 hash하면 된다.
3. inline assembly가 없이는 안되는 것들이 존재한다. Solidity는 external function으로부터 dynamic array인 bytes 또는 string의 length를 return하는 값을 가져오는 것을 지원하지 않는다. 하지만 length를 알고 있다면 inline assembly를 이용해 가져올 수 있다.

~~~javascript
contract AssemblyArrays {

    bytes testArray;

    function getLength() public view returns (uint256) {
        bytes memory memoryTestArray = testArray;
        uint256 result;
        assembly {
            result := mload(memoryTestArray)
        }
        return result;
    }

    function getElement(uint256 index) public view returns (bytes1) {
        return testArray[index];
    }
}
~~~
1. `bytes memory memoryTestArray = testArray;` : storage에 있던 memoryTestArray 값을 memory로 복사한다.
2. `result := mload(memoryTestArray)` : mload를 하면 memory에 있는 값을 stack에 얹는다. 이 값은 복사된 memoryTestArray 값이 메모리에 위치한 곳이다.
3. 메모리에 올라와 있는 데이터는 다음과 같다.
    ![data](https://i.imgur.com/viZvbJY.png)
4. **memory에서 bytes array의 첫 32 bytes는 length를 나타낸다.**

## Reference 
- [https://jeancvllr.medium.com/solidity-tutorial-all-about-assembly-5acdfefde05c](https://jeancvllr.medium.com/solidity-tutorial-all-about-assembly-5acdfefde05c)
- [https://ethereum.stackexchange.com/a/3174](https://ethereum.stackexchange.com/a/3174)
- [https://www.programmersought.com/article/56416827061/](https://www.programmersought.com/article/56416827061/)