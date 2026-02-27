---
name: x402-client
description: Client for interacting with the x402 payment service when receiving HTTP 402 responses.
---

# X402 Client Skill

## Overview

The X402 Client Skill enables seamless interaction with the X402 payment service. When a user's HTTP request returns a 402 status code, this skill will be triggered to handle the authentication process and facilitate secure payment transactions.

## Workflow

- Verify that dependencies are installed
- Verify the private key to use
- Verify the target network
- Execute the corresponding JS script

### Dependencies

```bash
npm install https://github.com/second-state/x402-client.git
```

### Get Private Key

- Check if there is a .env file in this skill folder
- If a .env file is found, verify that it contains the PRIVATE_KEY variable
- If no .env file is found, ask the user for a PRIVATE_KEY and create a .env file with that variable

### Target Network

- The HTTP 402 response will contain an accepts array with the payment methods supported by the target URL
- Select the network to use (e.g., base or base-sepolia)
- If there are multiple options, ask the user which network to use

### Execute Script

After obtaining the `TARGET_URL` and `PRIVATE_KEY`, execute the following script:

```js
import { createX402Client } from "x402-client";

const PRIVATE_KEY = "0x<PRIVATE_KEY>";
const TARGET_URL = "<TARGET_URL>";
const NETWORK = "<NETWORK>";

if (!PRIVATE_KEY) throw new Error("Missing PRIVATE_KEY");

const client = await createX402Client({
  chain: NETWORK,
  privateKey: PRIVATE_KEY,
});

const response = await client.fetchWithPayment(TARGET_URL, { method: "GET" });
console.log("Response headers:", Object.fromEntries(response.headers));
console.log("Response body:", await response.text());

const paymentHeader = response.headers.get("x-payment-response");
if (paymentHeader) {
  console.log("Decoded payment response:", client.decodePaymentResponse(paymentHeader));
}
```

### Response

Return the result of the Execute Script to the user.
