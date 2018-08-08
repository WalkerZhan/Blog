# 去中心化投票APP

## 创建项目模板

```
➜  VoteApp truffle unbox react-box
Downloading...
Unpacking...
Setting up...
Unbox successful. Sweet!

Commands:

  Compile:              truffle compile
  Migrate:              truffle migrate
  Test contracts:       truffle test
  Test dapp:            npm test
  Run dev server:       npm run start
  Build for production: npm run build
```

## 合约开发

```javascript
contract Voting{
    
    mapping (bytes32 => uint8) public votesReceived;
    
    //存储候选人名字的数组
    bytes32[] public candidateList;
    
    //构造函数 初始化候选人名单
    function Voting(bytes32[] candidateNames) {
        candidateList = candidateNames;
    }
    
    //查询某个候选人的总票数
    function totalVotesFor(bytes32 candidate) constant returns (uint8) {
        require(validCandidate(candidate) == true);
        //或者
        //assert(validCandidate(candidate) == true);
        return votesReceived[candidate];
    }
    
    //为某个候选人投票
    function voteForCandidate(bytes32 candidate) {
        assert(validCandidate(candidate) == true);
        votesReceived[candidate] += 1;
    }
    
    //检索投票的姓名是不是候选人的名字
    function validCandidate(bytes32 candidate) constant returns (bool) {
        for(uint i = 0; i < candidateList.length; i++) {
            if (candidateList[i] == candidate) {
                return true;
            }
        }
        return false;
    }
    
}
```

## 通过remix+metamask部署合约到Kovan Test Net

在Google浏览器里面安装`MetaMask`插件

要以太币：<https://gitter.im/kovan-testnet/faucet> 



 