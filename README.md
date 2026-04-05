# NFT-English-Auction
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

import "@openzeppelin/contracts/token/ERC721/IERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract BaseEnglishAuction is Ownable {
    IERC721 public nft;
    uint256 public nftId;
    uint256 public endTime;
    address public highestBidder;
    uint256 public highestBid;
    bool public ended;

    event BidPlaced(address bidder, uint256 amount);
    event AuctionEnded(address winner, uint256 amount);

    constructor(address _nft, uint256 _nftId, uint256 _duration) {
        nft = IERC721(_nft);
        nftId = _nftId;
        endTime = block.timestamp + _duration;
    }

    function bid() external payable {
        require(block.timestamp < endTime, "Auction ended");
        require(msg.value > highestBid, "Bid too low");

        if (highestBidder != address(0)) {
            payable(highestBidder).transfer(highestBid);
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
        emit BidPlaced(msg.sender, msg.value);
    }

    function endAuction() external {
        require(block.timestamp >= endTime, "Not finished");
        require(!ended, "Already ended");

        ended = true;
        nft.transferFrom(address(this), highestBidder, nftId);
        emit AuctionEnded(highestBidder, highestBid);
    }
}
