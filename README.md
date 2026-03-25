# Automated-Market-Maker-with-Fees
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/ERC20/IERC20.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/security/ReentrancyGuard.sol";

contract FeeAMM is ReentrancyGuard {
    IERC20 public tokenA;
    IERC20 public tokenB;
    uint256 public reserveA;
    uint256 public reserveB;
    uint256 public fee = 30; // 0.3%

    event Swap(address indexed user, address tokenIn, uint256 amountIn, uint256 amountOut);

    constructor(address _tokenA, address _tokenB) {
        tokenA = IERC20(_tokenA);
        tokenB = IERC20(_tokenB);
    }

    function swap(address tokenIn, uint256 amountIn) public nonReentrant returns (uint256 amountOut) {
        bool isAToB = tokenIn == address(tokenA);
        IERC20 input = isAToB ? tokenA : tokenB;
        IERC20 output = isAToB ? tokenB : tokenA;

        uint256 inputRes = isAToB ? reserveA : reserveB;
        uint256 outputRes = isAToB ? reserveB : reserveA;

        input.transferFrom(msg.sender, address(this), amountIn);

        uint256 amountInWithFee = amountIn * (10000 - fee) / 10000;
        amountOut = (outputRes * amountInWithFee) / (inputRes + amountInWithFee);

        output.transfer(msg.sender, amountOut);

        if (isAToB) {
            reserveA += amountIn;
            reserveB -= amountOut;
        } else {
            reserveB += amountIn;
            reserveA -= amountOut;
        }

        emit Swap(msg.sender, tokenIn, amountIn, amountOut);
    }
}
