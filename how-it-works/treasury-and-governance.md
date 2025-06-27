---
icon: building-columns
---

# Treasury & governance

The Doppler Airlock comes out of the box with optional support for community managed treasuries, via the GovernanceFactory. These are currently implemented using industry standard contracts, specifically, [Open Zeppelin's Governor](https://docs.openzeppelin.com/contracts/4.x/api/governance), which was directly inspired by Compound Finance's Governor Bravo. Treasuries can be customized at the time of creation, including how much of a token gets allocated into the Treasury, how long it takes for a Treasury to unlock, or what the voting requirements are to move assets or take actions on behalf of the Treasury.

{% hint style="info" %}
Check out the [OpenZeppelin smart contract Wizard](https://wizard.openzeppelin.com/#governor) to view what customizations are possible within Governor.
{% endhint %}

### Example usage

Tokens created on Pure Markets have a community managed treasury that unlocks after 90 days. After the 90 days, any community member can propose an action to the contracts directly. Due to the usage of standard solutions like Open Zeppelin's Governor, third party websites and services like [Tally.xyz](https://tally.xyz/) automatically support the token holder community's ability to vote on the proposed action. Community members would vote with weight proportionate to their token holdings.

{% hint style="info" %}
Refer to [GitHub](https://github.com/whetstoneresearch/doppler/blob/main/src/Governance.sol) to view the open source Doppler Treasury & Governance implementation
{% endhint %}

## Disabling or opting-out of governance

For projects that prefer to opt out of onchain token based governance mechanisms, Doppler offers a "no-op governance" option that maintains project control while enabling perpetual fee streaming.&#x20;

### How It Works

Doppler supports a "No-op" governance pattern that permanently locks 100% of liquidity and streams all trading fees to predefined beneficiaries.&#x20;

## Benefits of NoOpGovernanceFactory

1. **Gas Savings**: \~30-40% reduction in deployment costs
2. **Simplicity**: No governance overhead to manage
3. **Security**: Governance address set to `0xdead`, preventing any governance actions
4. **V4 Compatible**: Works seamlessly with V4 migration features

## When to Use Each Option

### Use NoOpGovernanceFactory when:

* Governance is not required for your token
* Gas efficiency is a priority
* You want a simpler deployment process
* Community governance will be handled off-chain

### Use Standard Governance when:

* On-chain governance is required
* Token holders need voting capabilities
* Protocol parameters may need updates
* Systems equire governance mechanisms



{% hint style="warning" %}
**Important**: This is a permanent decision that cannot be reversed or transferred.
{% endhint %}

See the [v3](../v3-sdk/governance-options.md) or [v4](../v4-sdk/governance-options.md) governance options for more information.
