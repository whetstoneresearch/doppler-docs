---
icon: qrcode-read
---

# SVM SDK Examples

These examples show how to create and trade Doppler launches on Solana.

## Fee settings

Each Solana launch chooses a standard trading fee when it is created. The current allowed range is `0.10%-5.00%`. The fee is paid in the token being sold: buys generate fees in SOL or USDC, while sells generate fees in the launched token.

Doppler receives `7.50%` of each collected trading fee, and the remainder goes to the beneficiaries selected for the launch. This is a share of the fee, not an additional fee on the trade. For example, with a `1%` trading fee, Doppler receives `0.075%` of the trade.

A launch's fee settings are saved when it is created, so later changes to the network settings apply only to new launches. A launch may also use a fee that decreases over time. It can begin anywhere from `0%` to `100%`, including above the standard fee range, but it can never reduce the fee below the launch's standard fee. See [Dynamic fee launch](dynamic-fee-launch.md).

## Cosigning

Cosigning is an optional transaction approval layer for a launch. While it is enabled, each swap must include a signature from an approved cosigner before Doppler accepts it. The cosigner does not take custody of the user's tokens or submit the trade for them; it only confirms that the transaction is allowed under the launch's trading policy.

A cosigning gate can end at a chosen time or remain active indefinitely, and it works with either standard or time-based fees. Once an expiring gate ends, trading continues normally without approval. Creating a launch and claiming fees do not require cosigning.

The standard SDK selects a cosigner already approved by Doppler; supplying an address does not register a new cosigner. Teams interested in cosigning, running their own approval service, or discussing a custom setup can contact [contact@whetstone.cc](mailto:contact@whetstone.cc).

* [Launch](launch.md) - create a standard Solana launch.
* [Dynamic fee launch](dynamic-fee-launch.md) - create a launch with a decaying swap fee schedule, optionally combined with cosigner gating.
* [Swap](swap.md) - swap against a Solana CPMM pool.
* [Launch, monitor, and e2e](launch-monitor-and-e2e.md) - create, monitor, trade, and migrate a launch end to end.
