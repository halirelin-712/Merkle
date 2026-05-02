# Merkle
Merkle.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

/**
 * @title BAST Token - Base Chain Airdrop
 * @dev BAST 토큰 (총 공급량 10억 개)
 * @author Grok (xAI)
 */
contract BASTToken {
    
    string public name = "BAST Token";
    string public symbol = "BAST";
    uint8 public decimals = 18;
    uint256 public totalSupply;
    
    address public owner;
    
    mapping(address => uint256) private _balances;
    mapping(address => bool) public hasClaimed;
    
    uint256 public constant INITIAL_AIRDROP = 10000 * 10**18;
    uint256 public constant TOTAL_SUPPLY = 1000000000 * 10**18;
    
    event Transfer(address indexed from, address indexed to, uint256 value);
    event AirdropDistributed(address indexed to, uint256 amount);

    constructor() {
        owner = msg.sender;
        totalSupply = TOTAL_SUPPLY;
        _balances[address(this)] = TOTAL_SUPPLY;
        emit Transfer(address(0), address(this), TOTAL_SUPPLY);
    }

    modifier onlyOwner() {
        require(msg.sender == owner, unicode"Owner만 호출할 수 있습니다");
        _;
    }

    function balanceOf(address account) public view returns (uint256) {
        return _balances[account];
    }

    /**
     * @dev Owner 
     */
    function distributeInitialAirdrop(address[] calldata recipients) public onlyOwner {
        for (uint256 i = 0; i < recipients.length; i++) {
            address to = recipients[i];
            if (to != address(0) && !hasClaimed[to]) {
                _balances[address(this)] -= INITIAL_AIRDROP;
                _balances[to] += INITIAL_AIRDROP;
                hasClaimed[to] = true;
                emit Transfer(address(this), to, INITIAL_AIRDROP);
                emit AirdropDistributed(to, INITIAL_AIRDROP);
            }
        }
    }

    /**
     * @dev 
     */
    function claimAirdrop() public {
        require(!hasClaimed[msg.sender], unicode"이미 claim했습니다");
        
        uint256 ethBalance = msg.sender.balance;
        uint256 claimAmount = ethBalance * 10;
        
        require(claimAmount > 0, unicode"ETH 잔고가 없습니다");
        require(_balances[address(this)] >= claimAmount, unicode"남은 토큰 부족");
        
        hasClaimed[msg.sender] = true;
        _balances[address(this)] -= claimAmount;
        _balances[msg.sender] += claimAmount;
        
        emit Transfer(address(this), msg.sender, claimAmount);
    }

    function withdrawRemaining() public onlyOwner {
        uint256 remaining = _balances[address(this)];
        if (remaining > 0) {
            _balances[address(this)] -= remaining;
            _balances[owner] += remaining;
            emit Transfer(address(this), owner, remaining);
        }
    }
}
0x323A2b2735a9a9D7CFDd6CE79D6B7b1b6301d083



