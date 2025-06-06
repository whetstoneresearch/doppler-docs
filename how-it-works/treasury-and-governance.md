---
icon: building-columns
---

# Treasury & governance

The Doppler Airlock comes out of the box with optional support for community managed treasuries, via the GovernanceFactory. These are currently implemented using industry standard contracts, specifically, [Open Zeppelin's Governor](https://docs.openzeppelin.com/contracts/4.x/api/governance), which was directly inspired by Compound Finance's Governor Bravo. Treasuries can be customized at the time of creation, including how much of a token gets allocated into the Treasury, how long it takes for a Treasury to unlock, or what the voting requirements are to move assets or take actions on behalf of the Treasury.&#x20;

{% hint style="info" %}
Check out the [OpenZeppelin smart contract Wizard](https://wizard.openzeppelin.com/#governor) to view what customizations are possible within Governor. 
{% endhint %}

### Example usage

Tokens created on Pure Markets have a community managed treasury that unlocks after 90 days. After the 90 days, any community member can propose an action to the contracts directly. Due to the usage of standard solutions like Open Zeppelin's Governor, third party websites and services like [Tally.xyz](https://tally.xyz/) automatically support the token holder community's ability to vote on the proposed action. Community members would vote with weight proportionate to their token holdings.

{% hint style="info" %}
Refer to [GitHub](https://github.com/whetstoneresearch/doppler/blob/main/src/Governance.sol) to view the open source Doppler Treasury & Governance implementation
{% endhint %}
