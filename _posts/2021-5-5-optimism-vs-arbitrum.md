---
layout: post
title: Optimism vs Arbitrum
tags: [layer-2]
---

1. 공통점: 1/ 👉 둘다 롤업이다. 즉 모든 l2 tx를 l1에 제출한다. 2/ 👉 둘다 옵티미스틱하다. 즉 fraud proof를 사용한다. 3/ 👉 instant "finality"를 위해 시퀀서를 사용한다. 4/ 👉 토큰 브릿지를 만들 수 있도록 하는 generic cross-chain messaging을 가진다.
2. 차이점: fraud proof의 매커니즘의 차이를 가진다.
Optimism은 fraud proof의 single round를 사용한다. state root를 검증하기 위해 on-chain에서 whole L2 transaction을 L1에서 실행한다. 이렇게 하면 fraud proof가 즉각적으로 된다는 장점이 있다.
3. Optimism의 방법에는 문제점도 존재함: 1/ 👉 tx 실행을 지켜봐야 하기 때문에 OVM이 필요하다. 2/ 👉 L2 tx 가스는 L1 block gas limit만큼의 제한이 있다. 3/ 👉 TX별로 온체인에 state root들을 필요로 한다. (비용이 많이 들게 됨.) 4/ 👉 잠재적인 보안 이슈
4. Arbitrum 특징은 multi-round fraud proof다. 두 party간의 binary search를 하면서 서로 간에 disagree하는 첫번째 opcode를 발견한다. on-chain에서 발견된 이 특정 opcode를 실행한다.
5. Arbitrum은 다음의 장점을 가진다. 1/ 👉 단 하나의 state proof만 on-chain에 등록하면 된다. 2/ 👉 L1 block gas limit은 중요하지 않다. 왜냐하면 L2 tx 전체가 L1에서 실행되는 것이 아니기 때문이다.
6. 단점은 다음과 같다. 1/ 👉 EVM → AVM translation이 필요하다(이는 자동으로 이루어지긴 한다.). 2/ 👉 느리다. fraud proof를 하는데 최대 2주까지 걸릴 수 있다. 현실적으로는 1주일만을 사용할 것이다. 3/ 👉 original claimer가 항상 online이면서 협력적이어야 한다.
7. 다르게 생각해봤을 때, Optimism은 containerization이고 Arbitrum은 virtualization이다.
8. 옵티미즘에는 큰 단점이 존재한다. 이더리움 하드포크로 인해서 컨센서스 룰이 변경될 때, opcode들 중에 하나가 제거되거나 가격이 변하거나 수정될 수 있다.
9. 그렇기 때문에 이전 버전의 tx를 하드포크가 발생한 L1에서 실행하면 다른 final state가 만들어질 수 있다. 옵티미즘 팀은 이에 대한 해결책을 어떻게 마련할 것인지 모르겠지만 언젠가 이에 대한 해결방법을 찾을 수 있다고 생각한다. Arbirtrum은 AVM spec에 대한 통제권을 가지기 때문에 이러한 문제를 가지지 않는다.
10. 두 프로젝트 모두 이더리움과 관련된 툴(solidity, hardhat, waffle etc)들을 사용할 수 있지만, 사용하는 것이 그렇게 쉽진 않다.
11. 옵티미즘은 OVM bytecode를 만들어내기 위한 optimistic용 컴파일러를 사용한다. 이 컴파일러는 Solidity에서만 동작하고 특정 버전에서만 사용할 수 있다. 반면에 L2 node는 geth를 수정한 것으로 compatibility가 좋다.
12. Arbitrum은 EVM/JSON RPC 스펙과 완전히 호환되지만, Arbitrum의 노드는 custom implementation이다. 이 노드는 EVM → AVM transpilation을 해서 fraud proof를 지원한다. EVM → AVM translation은 low level에서 이루어지기 때문에 EVM language(solidity, vyper, YUL + etc)는 모두 지원한다.
13. Optimism은 weth를 사용하지만 Arbitrum은 native eth를 지원한다.
14. Arbitrum은 통합된 하나의 permissionless bridge를 사용해서 token을 L2로 옮길 수 있게 하는 반면, Optimism은 토큰 별로 자신만의 brdige를 만들도록 권한다.

## Reference
- [https://twitter.com/krzKaczor/status/1395812308451004419](https://twitter.com/krzKaczor/status/1395812308451004419)
