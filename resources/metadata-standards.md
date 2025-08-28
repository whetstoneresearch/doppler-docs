---
description: Best practices for ecosystem support and compatability
icon: scale-balanced
---

# Metadata standards

Tokens created on Doppler should attempt to adhere to ecosystem best practices to ensure compatibility across as many supported interfaces as possible. We aim to make it as simple, and flexible, as possible, in order to support a wide range of applications and use cases.&#x20;

#### Requirements to be supported within the Doppler Indexer

1. Tokens must be created with a URI stored on IPFS that contains an image field
2. The image field in the URI must also contain another pointer to a valid IPFS CID&#x20;

{% hint style="info" %}
Have suggestions? File an issue on the documentation, or get in touch with the team!
{% endhint %}

### Examples

#### Example token created on Pure Markets

```
{
  "name": "8453",
  "symbol": "8453",
  "image": "ipfs://QmZEFVUKoQDxsUbsRTgrpdwhwxDM95HbMzc7kjUxjynLDk",
  "x": "",
  "telegram": "",
  "farcaster": "",
  "discord": ""
}
```

* tokenURI: [ipfs://QmTM59ZqHcJgv1EQVWs6e96Hchd3diHW9F9evLWmX2PQCU](ipfs://QmTM59ZqHcJgv1EQVWs6e96Hchd3diHW9F9evLWmX2PQCU)
* Image: [ipfs://QmZEFVUKoQDxsUbsRTgrpdwhwxDM95HbMzc7kjUxjynLDk](ipfs://QmZEFVUKoQDxsUbsRTgrpdwhwxDM95HbMzc7kjUxjynLDk)&#x20;
* :link: [https://ipfs.io/ipfs/QmTM59ZqHcJgv1EQVWs6e96Hchd3diHW9F9evLWmX2PQCU](https://ipfs.io/ipfs/QmTM59ZqHcJgv1EQVWs6e96Hchd3diHW9F9evLWmX2PQCU)&#x20;

| Field     | Example                                               |
| --------- | ----------------------------------------------------- |
| name      | 8453                                                  |
| symbol    | 8453                                                  |
| image     | ipfs://QmZEFVUKoQDxsUbsRTgrpdwhwxDM95HbMzc7kjUxjynLDk |
| x         |                                                       |
| telegram  |                                                       |
| farcaster |                                                       |
| discord   |                                                       |

Note: fields can be left empty. Pure.st will not attempt to load social links or icons for empty inputs.&#x20;

#### Example token created on Zora

<pre><code>{
  "name": "balajis",
  "ticker": "balajis",
  "image": "ipfs://bafybeibeeqdppfm2iizeaqiobdvn533xyuunjpdspdirju7xf4ykwyindq",
  "content": {
    "uri": "ipfs://bafybeibeeqdppfm2iizeaqiobdvn533xyuunjpdspdirju7xf4ykwyindq"
  }

<strong>
</strong></code></pre>

* tokenURI: [ipfs://bafybeiexri3tfd6fgjfume3tovyx6xdppgcapmvbjrqtraxec66pbtrus4](ipfs://bafybeiexri3tfd6fgjfume3tovyx6xdppgcapmvbjrqtraxec66pbtrus4)
* Image: [ipfs://bafybeibeeqdppfm2iizeaqiobdvn533xyuunjpdspdirju7xf4ykwyindq](ipfs://bafybeibeeqdppfm2iizeaqiobdvn533xyuunjpdspdirju7xf4ykwyindq)
* :link: [https://ipfs.io/ipfs/QmTM59ZqHcJgv1EQVWs6e96Hchd3diHW9F9evLWmX2PQCU](https://ipfs.io/ipfs/QmTM59ZqHcJgv1EQVWs6e96Hchd3diHW9F9evLWmX2PQCU)&#x20;

| Field   | Example                                                            |
| ------- | ------------------------------------------------------------------ |
| name    | balajis                                                            |
| ticker  | balajis                                                            |
| image   | ipfs://bafybeibeeqdppfm2iizeaqiobdvn533xyuunjpdspdirju7xf4ykwyindq |
| content | ipfs://bafybeibeeqdppfm2iizeaqiobdvn533xyuunjpdspdirju7xf4ykwyindq |

