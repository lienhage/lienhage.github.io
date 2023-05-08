---
layout: post
title: Tornado.cash-anonymous-mining
date: '2023-04-27 19:23:35 +0800'
# categories: [ZK, ]
tags: [zero-knowledge-proofs]
math: true
---


## Mechanism Design
### AMM of AP (anonymity points) and Governance Token (TORN)

1. Define pool's virtual reserve of TORN as:

- Within TORN release session (1 year): 
  
    $$ reserve = initialLiquidity + \frac{schedualedReleaseLiquidity * passedTime} {duration} - soldTokens $$

- After expired:
  
  $$ reserve = balanceOf(address(swapContract)) $$

2. AMM mechanism (swap APs for Torn tokens):
   
$$ reserveAfter = reserveBefore * e ^ {-\frac{APamount} {poolweight} } $$

$$ receivedAmount = reserveBefore - reserveAfter $$

### AP Calculation:

AP can be obtained by depositing a certain amount of liquidity into tornado for a period of time:

$$ APAmount = depositAmount * rewardRatePerBlock * passedBlocks $$

Each tornado instance has a certain denomination, so we can define rate as:

$$rate = denomination * rewardRatePerBlock$$

Then the formula turns into:

$$ APAmount = rate * passedBlocks $$

### Anonymity:
1. The duration related to a certain amount of deposit is shielded through Zero-Knowledge.
   
2. One thing can be derived is the AP amount of a reward swap since $reserveAfter$ and $reserveBefore$ can be tracked through the history state, but the AP of an account is likely to have been accumulated from different deposits, i.e.,:
   
   $$ AP=\sum ^{n}_{i=0} rates[i]*durations[i] $$

    There are many possibilities of duration given a range of rates and AP, and one may not spend his AP at once but rather for several times and let the tokens his received a normal amount, making the fund source hard to be identified.
    Furthermore,  liquidity withdraw and reward withdraw are separated operations, thus a realistic user could leave the reward in the pool long enough before he performs a reward withdraw, this will expand the possible time range and make it harder to track.

3. Like the tornado pool, users can entrust the *withdraw reward*  transaction to a relayer as long as they pay for the gas and service fee.

## Implementation

### Architecture Design

1. Key types:
   
    Despite the tornado-core already has a merkle tree recording user’s deposit commitment hash and withdrawal nullifier hash, we need more information including user’s deposit/withdrawal block number and the instance’s denomination since AP is calculated according to duration the funds stay and how many tokens they deposited/withdrawn. The bad news is that tornado-v2 is immutable, thus we have to maintain two more trees: deposit tree and withdrawal tree. The leaves data are as follows:

    ```
    1. Deposit tree
    leaf: hash(instance, depositCommitment, blockNumber)

    1. Withdrawal tree
    leaf: hash(instance, withdrawNullifierHash, blockNumber)

    1. Account tree
    leaf: hash(amount, secret, nullfier)

    ```

    A user could provide a pair of valid deposit & withdraw commitment  at any time to get his AP to a virtual account. So there is another account tree whose leaf data contains account’s AP amount, secret and the nullifier. Also the account’s AP amount can be accumulated from multiple deposit/withdraw commitment pair and transferred to another account.



### Interactions and code details

1. User deposit/withdraw through Proxy contract, which will register user’s mining deposit/withdrawal commitment in Miner.

    ``` 

    function deposit(
        ITornadoInstance _tornado,
        bytes32 _commitment,
        bytes calldata _encryptedNote
    ) public payable virtual {
        Instance memory instance = instances[_tornado];
        require(instance.state != InstanceState.DISABLED, "The instance is not supported");

        if (instance.isERC20) {
            instance.token.safeTransferFrom(msg.sender, address(this), _tornado.denomination());
        }
        _tornado.deposit{ value: msg.value }(_commitment);

            // if the tornado instance is mineble
        if (instance.state == InstanceState.MINEABLE) {
            tornadoTrees.registerDeposit(address(_tornado), _commitment);
        }
        emit EncryptedNote(msg.sender, _encryptedNote);
    }

    /// @dev Queue a new deposit data to be inserted into a merkle tree
    /// leaf: keccak256(abi.encode(_instance, _commitment, blockNumber()))
    /// store the deposit leaf in a mapping
    function registerDeposit(address _instance, bytes32 _commitment) public onlyTornadoProxy {
        uint256 _depositsLength = depositsLength;
        deposits[_depositsLength] = keccak256(abi.encode(_instance, _commitment, blockNumber()));
        emit DepositData(_instance, _commitment, blockNumber(), _depositsLength);
        depositsLength = _depositsLength + 1;
    }

    ```

2. The community or the other Joe call the TornadoTrees contract to update deposit tree.

![Desktop View](/assets/blogImg/20230427/tornado-anonymous-mining-architecture.png){: width="972" height="589" }
_tornado.cash anonymous mining interactions flow_
