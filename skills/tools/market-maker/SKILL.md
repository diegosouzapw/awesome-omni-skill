---
name: market-maker
description: Create a market maker bot for Turbine's BTC 15-minute prediction markets. Use when building trading bots for Turbine.
disable-model-invocation: true
argument-hint: "[algorithm-type]"
---

# Turbine Market Maker Bot Generator

You are helping a programmer create a market maker bot for Turbine's Bitcoin 15-minute prediction markets.

## Step 0: Environment Context Detection

**CRITICAL**: Before writing ANY Python code, you MUST detect the user's environment to ensure correct syntax and compatibility.

Run these commands to gather environment context:

```bash
# Get Python version
python3 --version

# Check if in virtualenv
echo "VIRTUAL_ENV: $VIRTUAL_ENV"

# Get platform info
uname -s

# Check if pyproject.toml exists for project Python requirements
cat pyproject.toml 2>/dev/null | grep -E "(requires-python|python_version)" || echo "No pyproject.toml found"
```

**Environment Rules:**
- If Python version is 3.9+: Use modern syntax (type hints with `list[str]` instead of `List[str]`, `dict[str, int]` instead of `Dict[str, int]`, `X | None` instead of `Optional[X]`)
- If Python version is 3.8 or below: Use `from typing import List, Dict, Optional` and older syntax
- Always match the project's `requires-python` if specified in pyproject.toml
- Use `async`/`await` syntax (supported in all Python 3.9+ environments)
- For dataclasses, use `@dataclass` decorator (available in Python 3.7+)

Store the detected Python version mentally and use it for ALL generated code in this session.

## Step 1: Environment Setup Check

First, check if the user has the required setup:

1. Check if `turbine_client` is importable by looking at the project structure
2. Check if `.env` file exists with the required credentials
3. If `.env` doesn't exist, guide them through creating it

## Step 2: Private Key Setup

Check if .env file exists with TURBINE_PRIVATE_KEY set. If not:

1. Use AskUserQuestion to ask for their Ethereum wallet private key
2. Explain security best practices:
   - Use a dedicated trading wallet with limited funds
   - Never share your private key
   - Get it from MetaMask: Settings > Security & Privacy > Export Private Key
3. Once they provide it, CREATE the .env file directly using the Write tool

Do NOT just tell them to create the file - actually create it for them!

## Step 3: API Key Auto-Registration

The bot should automatically register for API credentials on first run AND save them to the .env file automatically. Use this pattern:

```python
import os
import re
from pathlib import Path
from turbine_client import TurbineClient

def get_or_create_api_credentials(env_path: Path = None):
    """Get existing credentials or register new ones and save to .env."""
    if env_path is None:
        env_path = Path(__file__).parent / ".env"

    api_key_id = os.environ.get("TURBINE_API_KEY_ID")
    api_private_key = os.environ.get("TURBINE_API_PRIVATE_KEY")

    if api_key_id and api_private_key:
        print("Using existing API credentials")
        return api_key_id, api_private_key

    # Register new credentials
    private_key = os.environ.get("TURBINE_PRIVATE_KEY")
    if not private_key:
        raise ValueError("TURBINE_PRIVATE_KEY not set in environment")

    print("Registering new API credentials...")
    credentials = TurbineClient.request_api_credentials(
        host="https://api.turbinefi.com",
        private_key=private_key,
    )

    api_key_id = credentials["api_key_id"]
    api_private_key = credentials["api_private_key"]

    # Auto-save credentials to .env file
    _save_credentials_to_env(env_path, api_key_id, api_private_key)

    # Update current environment so bot can use them immediately
    os.environ["TURBINE_API_KEY_ID"] = api_key_id
    os.environ["TURBINE_API_PRIVATE_KEY"] = api_private_key

    print(f"API credentials registered and saved to {env_path}")
    return api_key_id, api_private_key


def _save_credentials_to_env(env_path: Path, api_key_id: str, api_private_key: str):
    """Save API credentials to .env file."""
    env_path = Path(env_path)

    if env_path.exists():
        content = env_path.read_text()
        # Update or append TURBINE_API_KEY_ID
        if "TURBINE_API_KEY_ID=" in content:
            content = re.sub(r'^TURBINE_API_KEY_ID=.*$', f'TURBINE_API_KEY_ID={api_key_id}', content, flags=re.MULTILINE)
        else:
            content = content.rstrip() + f"\nTURBINE_API_KEY_ID={api_key_id}"
        # Update or append TURBINE_API_PRIVATE_KEY
        if "TURBINE_API_PRIVATE_KEY=" in content:
            content = re.sub(r'^TURBINE_API_PRIVATE_KEY=.*$', f'TURBINE_API_PRIVATE_KEY={api_private_key}', content, flags=re.MULTILINE)
        else:
            content = content.rstrip() + f"\nTURBINE_API_PRIVATE_KEY={api_private_key}"
        env_path.write_text(content + "\n")
    else:
        # Create new .env file
        content = f"""# Turbine Trading Bot Configuration
TURBINE_PRIVATE_KEY={os.environ.get('TURBINE_PRIVATE_KEY', '0x...')}
TURBINE_API_KEY_ID={api_key_id}
TURBINE_API_PRIVATE_KEY={api_private_key}
"""
        env_path.write_text(content)
```

## Step 4: Algorithm Selection

Present the user with these trading algorithm options for prediction markets:

**Option 1: Price Action Trader (Recommended)**
- Uses real-time BTC price from Pyth Network (same oracle Turbine uses)
- Compares current price to the market's strike price
- If BTC is above strike price → buy YES (bet it stays above)
- If BTC is below strike price → buy NO (bet it stays below)
- Adjusts confidence based on how far price is from strike
- Best for: Beginners, following price momentum
- Risk: Medium - follows current price action

**Option 2: Simple Spread Market Maker**
- Places bid and ask orders around the mid-price with a fixed spread
- Best for: Learning the basics, stable markets
- Risk: Medium - can accumulate inventory in trending markets

**Option 3: Inventory-Aware Market Maker**
- Adjusts quotes based on current position to reduce inventory risk
- Skews prices to encourage trades that reduce position
- Best for: Balanced exposure, risk management
- Risk: Lower - actively manages inventory

**Option 4: Momentum-Following Trader**
- Detects price direction from recent trades
- Buys when momentum is up, sells when momentum is down
- Best for: Trending markets, breakouts
- Risk: Higher - can be wrong on reversals

**Option 5: Mean Reversion Trader**
- Fades large moves expecting price to revert
- Buys after dips, sells after spikes
- Best for: Range-bound markets, overreactions
- Risk: Higher - can fight strong trends

**Option 6: Probability-Weighted Trader**
- Uses distance from 50% as a signal
- Bets on extremes reverting toward uncertainty
- Best for: Markets with overconfident pricing
- Risk: Medium - based on market efficiency assumptions

## Step 5: Generate the Bot Code

Based on the user's algorithm choice, generate a complete bot file. The bot should:

1. Load credentials from environment variables
2. Auto-register API keys if needed
3. Connect to the BTC 15-minute quick market
4. Implement the chosen algorithm
5. Include proper error handling
6. Cancel orders on shutdown
7. **Automatically detect new BTC markets and switch liquidity/trades to them**
8. Handle market expiration gracefully with seamless transitions
9. **Sign USDC permits for gasless order execution** (no separate approval transaction needed)
10. **Track traded markets and automatically claim winnings when they resolve**

Use this template structure for all bots:

```python
"""
Turbine Market Maker Bot - {ALGORITHM_NAME}
Generated for Turbine

Algorithm: {ALGORITHM_DESCRIPTION}
"""

import asyncio
import os
import re
import time
from pathlib import Path
from dotenv import load_dotenv
import httpx  # For Price Action Trader - fetching BTC price from Pyth Network

from turbine_client import TurbineClient, TurbineWSClient, Outcome, Side
from turbine_client.exceptions import TurbineApiError, WebSocketError

# Load environment variables
load_dotenv()

# ============================================================
# CONFIGURATION - Adjust these parameters for your strategy
# ============================================================
ORDER_SIZE = 1_000_000  # 1 share (6 decimals)
MAX_POSITION = 5_000_000  # Maximum position size (5 shares)
QUOTE_REFRESH_SECONDS = 30  # How often to refresh quotes
# Algorithm-specific parameters added here...

def get_or_create_api_credentials(env_path: Path = None):
    """Get existing credentials or register new ones and save to .env."""
    if env_path is None:
        env_path = Path(__file__).parent / ".env"

    api_key_id = os.environ.get("TURBINE_API_KEY_ID")
    api_private_key = os.environ.get("TURBINE_API_PRIVATE_KEY")

    if api_key_id and api_private_key:
        print("Using existing API credentials")
        return api_key_id, api_private_key

    private_key = os.environ.get("TURBINE_PRIVATE_KEY")
    if not private_key:
        raise ValueError("Set TURBINE_PRIVATE_KEY in your .env file")

    print("Registering new API credentials...")
    credentials = TurbineClient.request_api_credentials(
        host="https://api.turbinefi.com",
        private_key=private_key,
    )

    api_key_id = credentials["api_key_id"]
    api_private_key = credentials["api_private_key"]

    # Auto-save to .env
    _save_credentials_to_env(env_path, api_key_id, api_private_key)
    os.environ["TURBINE_API_KEY_ID"] = api_key_id
    os.environ["TURBINE_API_PRIVATE_KEY"] = api_private_key

    print(f"API credentials saved to {env_path}")
    return api_key_id, api_private_key


def _save_credentials_to_env(env_path: Path, api_key_id: str, api_private_key: str):
    """Save API credentials to .env file."""
    env_path = Path(env_path)

    if env_path.exists():
        content = env_path.read_text()
        # Update or append each credential
        if "TURBINE_API_KEY_ID=" in content:
            content = re.sub(r'^TURBINE_API_KEY_ID=.*$', f'TURBINE_API_KEY_ID={api_key_id}', content, flags=re.MULTILINE)
        else:
            content = content.rstrip() + f"\nTURBINE_API_KEY_ID={api_key_id}"
        if "TURBINE_API_PRIVATE_KEY=" in content:
            content = re.sub(r'^TURBINE_API_PRIVATE_KEY=.*$', f'TURBINE_API_PRIVATE_KEY={api_private_key}', content, flags=re.MULTILINE)
        else:
            content = content.rstrip() + f"\nTURBINE_API_PRIVATE_KEY={api_private_key}"
        env_path.write_text(content + "\n")
    else:
        content = f"# Turbine Bot Config\nTURBINE_PRIVATE_KEY={os.environ.get('TURBINE_PRIVATE_KEY', '')}\nTURBINE_API_KEY_ID={api_key_id}\nTURBINE_API_PRIVATE_KEY={api_private_key}\n"
        env_path.write_text(content)


class MarketMakerBot:
    """Market maker bot implementation with automatic market switching and winnings claiming."""

    def __init__(self, client: TurbineClient):
        self.client = client
        self.market_id: str | None = None
        self.settlement_address: str | None = None  # For USDC permits
        self.contract_address: str | None = None  # For claiming winnings
        self.strike_price: int = 0  # BTC price when market created (8 decimals) - used by Price Action Trader
        self.current_position = 0
        self.active_orders: dict[str, str] = {}  # order_hash -> side
        self.running = True
        # Track markets we've traded in for claiming winnings
        self.traded_markets: dict[str, str] = {}  # market_id -> contract_address
        # Algorithm state...

    async def get_active_market(self) -> tuple[str, int, int]:
        """
        Get the currently active BTC quick market.
        Returns (market_id, end_time, start_price) tuple.
        """
        quick_market = self.client.get_quick_market("BTC")
        return quick_market.market_id, quick_market.end_time, quick_market.start_price

    async def cancel_all_orders(self, market_id: str) -> None:
        """Cancel all active orders on a market before switching."""
        if not self.active_orders:
            return

        print(f"Cancelling {len(self.active_orders)} orders on market {market_id[:8]}...")
        for order_id in list(self.active_orders.keys()):
            try:
                self.client.cancel_order(market_id=market_id, order_id=order_id)
                del self.active_orders[order_id]
            except TurbineApiError as e:
                print(f"Failed to cancel order {order_id}: {e}")

    async def switch_to_new_market(self, new_market_id: str, start_price: int = 0) -> None:
        """
        Switch liquidity and trading to a new market.
        Called when a new BTC 15-minute market becomes active.

        Args:
            new_market_id: The new market ID to switch to.
            start_price: The BTC price when market was created (8 decimals).
                         Used by Price Action Trader to compare against current price.
        """
        old_market_id = self.market_id

        # Track old market for claiming winnings later
        if old_market_id and self.contract_address:
            self.traded_markets[old_market_id] = self.contract_address
            print(f"Tracking market {old_market_id[:8]}... for winnings claim")

        if old_market_id:
            print(f"\n{'='*50}")
            print(f"MARKET TRANSITION DETECTED")
            print(f"Old market: {old_market_id[:8]}...")
            print(f"New market: {new_market_id[:8]}...")
            print(f"{'='*50}\n")

            # Cancel all orders on the old market
            await self.cancel_all_orders(old_market_id)

        # Update to new market
        self.market_id = new_market_id
        self.strike_price = start_price  # Store for Price Action Trader
        self.active_orders = {}

        # Fetch settlement and contract addresses from markets list
        try:
            markets = self.client.get_markets()
            for market in markets:
                if market.id == new_market_id:
                    self.settlement_address = market.settlement_address
                    self.contract_address = market.contract_address
                    print(f"Settlement: {self.settlement_address[:16]}...")
                    print(f"Contract: {self.contract_address[:16]}...")
                    break
        except Exception as e:
            print(f"Warning: Could not fetch market addresses: {e}")

        strike_usd = start_price / 1e8 if start_price else 0
        print(f"Now trading on market: {new_market_id[:8]}...")
        if strike_usd > 0:
            print(f"Strike price: ${strike_usd:,.2f}")

    async def monitor_market_transitions(self) -> None:
        """
        Background task that polls for new markets and triggers transitions.
        Runs continuously while the bot is active.
        """
        POLL_INTERVAL = 5  # Check every 5 seconds

        while self.running:
            try:
                new_market_id, end_time, start_price = await self.get_active_market()

                # Check if market has changed
                if new_market_id != self.market_id:
                    await self.switch_to_new_market(new_market_id, start_price)

                # Log time remaining periodically
                time_remaining = end_time - int(time.time())
                if time_remaining <= 60 and time_remaining > 0:
                    print(f"Market expires in {time_remaining}s - preparing for transition...")

            except Exception as e:
                print(f"Market monitor error: {e}")

            await asyncio.sleep(POLL_INTERVAL)

    # ... Algorithm implementation ...


async def main():
    # Get credentials
    private_key = os.environ.get("TURBINE_PRIVATE_KEY")
    if not private_key:
        print("Error: Set TURBINE_PRIVATE_KEY in your .env file")
        return

    api_key_id, api_private_key = get_or_create_api_credentials()

    # Create client
    client = TurbineClient(
        host="https://api.turbinefi.com",
        chain_id=137,  # Polygon mainnet
        private_key=private_key,
        api_key_id=api_key_id,
        api_private_key=api_private_key,
    )

    print(f"Bot wallet address: {client.address}")

    # Get the initial active BTC 15-minute market
    quick_market = client.get_quick_market("BTC")
    print(f"Initial market: BTC @ ${quick_market.start_price / 1e8:,.2f}")
    print(f"Market expires at: {quick_market.end_time}")

    # Note gasless features
    print("Orders will include USDC permit signatures for gasless trading")
    print("Automatic winnings claim enabled for resolved markets")
    print()

    # Run the bot with automatic market switching and winnings claiming
    bot = MarketMakerBot(client)

    try:
        # Initialize with the current market (pass start_price for Price Action Trader)
        await bot.switch_to_new_market(quick_market.market_id, quick_market.start_price)

        # Run the main trading loop (starts background tasks internally)
        await bot.run("https://api.turbinefi.com")
    except KeyboardInterrupt:
        print("\nShutting down...")
    finally:
        bot.running = False
        # Cancel any remaining orders before exit
        if bot.market_id:
            await bot.cancel_all_orders(bot.market_id)
        client.close()
        print("Bot stopped cleanly.")


if __name__ == "__main__":
    asyncio.run(main())
```

## Step 6: Create the .env File and Install Dependencies

IMPORTANT: Actually create the .env file for the user using the Write tool. Do NOT just tell them to copy a template.

Ask the user for their Ethereum private key using AskUserQuestion, then:

1. Create the `.env` file directly with their private key:
```
# Turbine Trading Bot Configuration
TURBINE_PRIVATE_KEY=0x...user's_actual_key...
TURBINE_API_KEY_ID=
TURBINE_API_PRIVATE_KEY=
```

2. Install dependencies by running:
```bash
pip install -e . python-dotenv httpx
```

Note: `httpx` is used by the Price Action Trader to fetch real-time BTC prices from Pyth Network.

## Step 7: Explain How to Run

Tell the user:
```
Your bot is ready! To run it:

  python {bot_filename}.py

The bot will:
- Automatically register API credentials on first run (saved to .env)
- Connect to the current BTC 15-minute market
- Start trading based on your chosen algorithm
- Sign USDC permits for gasless order execution (no approval TX needed)
- Automatically switch to new markets when they start
- Track traded markets and claim winnings when they resolve

To stop the bot, press Ctrl+C.
```

## Core Bot Run Method

Every generated bot must include this `run()` method that handles WebSocket streaming with automatic market switching and winnings claiming:

```python
async def run(self, host: str) -> None:
    """
    Main trading loop with WebSocket streaming, automatic market switching, and winnings claiming.
    """
    ws = TurbineWSClient(host)

    # Start background tasks
    monitor_task = asyncio.create_task(self.monitor_market_transitions())
    claim_task = asyncio.create_task(self.claim_resolved_markets())

    while self.running:
        try:
            # Ensure we have a current market
            if not self.market_id:
                market_id, _ = await self.get_active_market()
                await self.switch_to_new_market(market_id)

            current_market = self.market_id

            async with ws.connect() as stream:
                # Subscribe to the current market
                await stream.subscribe(current_market)
                print(f"Subscribed to market {current_market[:8]}...")

                # Place initial quotes (with USDC permits)
                await self.place_quotes()

                async for message in stream:
                    # Check if market has changed (set by monitor task)
                    if self.market_id != current_market:
                        print("Market changed, reconnecting to new market...")
                        break  # Exit inner loop to reconnect

                    if message.type == "orderbook":
                        await self.on_orderbook_update(message.orderbook)
                    elif message.type == "trade":
                        await self.on_trade(message.trade)
                    elif message.type == "order_cancelled":
                        self.on_order_cancelled(message.data)

        except WebSocketError as e:
            print(f"WebSocket error: {e}, reconnecting...")
            await asyncio.sleep(1)
        except Exception as e:
            print(f"Unexpected error: {e}")
            await asyncio.sleep(5)

    # Cleanup background tasks
    monitor_task.cancel()
    claim_task.cancel()
```

## Algorithm Implementation Details

When generating bots, use these implementations:

### Price Action Trader (Recommended)

This algorithm fetches the current BTC price from **Pyth Network** (the same oracle Turbine uses) and compares it to the market's strike price to make trading decisions.

```python
import httpx

# Pyth Network Hermes API - same price source Turbine uses
PYTH_HERMES_URL = "https://hermes.pyth.network/v2/updates/price/latest"
PYTH_BTC_FEED_ID = "0xe62df6c8b4a85fe1a67db44dc12de5db330f7ac66b72dc658afedf0f4a415b43"

# Configuration
PRICE_THRESHOLD_BPS = 50  # 0.5% threshold before taking action
MIN_CONFIDENCE = 0.6  # Minimum confidence to place a trade
MAX_CONFIDENCE = 0.9  # Cap confidence at 90%
PRICE_POLL_SECONDS = 10  # How often to check price

class PriceActionBot:
    def __init__(self, client: TurbineClient):
        self.client = client
        self.market_id: str | None = None
        self.strike_price: int = 0  # BTC price when market created (8 decimals)
        self.current_position = 0
        self.active_orders: dict[str, str] = {}
        self.running = True
        self.traded_markets: dict[str, str] = {}
        self.settlement_address: str | None = None
        self.contract_address: str | None = None

    def get_current_btc_price(self) -> float:
        """Fetch current BTC price from Pyth Network (same source as Turbine)."""
        try:
            response = httpx.get(
                PYTH_HERMES_URL,
                params={"ids[]": PYTH_BTC_FEED_ID},
                timeout=5.0,
            )
            response.raise_for_status()
            data = response.json()

            if not data.get("parsed"):
                print("No price data from Pyth")
                return 0.0

            price_data = data["parsed"][0]["price"]
            price_int = int(price_data["price"])
            expo = price_data["expo"]  # Usually -8 for BTC

            # Convert Pyth price to USD: price * 10^expo
            return price_int * (10 ** expo)

        except Exception as e:
            print(f"Failed to fetch BTC price from Pyth: {e}")
            return 0.0

    def calculate_signal(self) -> tuple[str, float]:
        """
        Calculate trading signal based on current price vs strike price.

        Returns:
            (action, confidence) where action is "BUY_YES", "BUY_NO", or "HOLD"
        """
        current_price = self.get_current_btc_price()
        if current_price <= 0:
            return "HOLD", 0.0

        # Convert strike price from 8 decimals to USD
        strike_usd = self.strike_price / 1e8

        # Calculate percentage difference
        price_diff_pct = ((current_price - strike_usd) / strike_usd) * 100

        # Threshold check (0.5% = 50 bps)
        threshold_pct = PRICE_THRESHOLD_BPS / 100

        if abs(price_diff_pct) < threshold_pct:
            # Price too close to strike, hold
            return "HOLD", 0.0

        # Calculate confidence based on distance from strike
        # Further from strike = higher confidence (capped)
        raw_confidence = min(abs(price_diff_pct) / 2, MAX_CONFIDENCE)
        confidence = max(raw_confidence, MIN_CONFIDENCE) if abs(price_diff_pct) >= threshold_pct else 0.0

        if price_diff_pct > 0:
            # BTC is above strike → bet YES (will end above)
            print(f"BTC ${current_price:,.2f} is {price_diff_pct:+.2f}% above strike ${strike_usd:,.2f}")
            return "BUY_YES", confidence
        else:
            # BTC is below strike → bet NO (will end below)
            print(f"BTC ${current_price:,.2f} is {price_diff_pct:+.2f}% below strike ${strike_usd:,.2f}")
            return "BUY_NO", confidence

    async def execute_signal(self, action: str, confidence: float) -> None:
        """Execute the trading signal."""
        if action == "HOLD" or confidence < MIN_CONFIDENCE:
            return

        # Check position limits
        if abs(self.current_position) >= MAX_POSITION:
            print("Position limit reached")
            return

        # Get orderbook to determine price
        orderbook = self.client.get_orderbook(self.market_id)

        if action == "BUY_YES":
            # Buy YES outcome
            if not orderbook.asks:
                return
            # Pay slightly above best ask to ensure fill
            price = min(orderbook.asks[0].price + 5000, 999000)
            outcome = Outcome.YES
        else:
            # Buy NO outcome
            if not orderbook.asks:
                return
            price = min(orderbook.asks[0].price + 5000, 999000)
            outcome = Outcome.NO

        try:
            order = self.client.create_limit_buy(
                market_id=self.market_id,
                outcome=outcome,
                price=price,
                size=ORDER_SIZE,
                expiration=int(time.time()) + 300,
                settlement_address=self.settlement_address,
            )

            # Sign USDC permit for gasless execution
            buyer_cost = (ORDER_SIZE * price) // 1_000_000
            permit_amount = (buyer_cost * 120) // 100  # 20% margin
            permit = self.client.sign_usdc_permit(
                value=permit_amount,
                settlement_address=self.settlement_address,
            )
            order.permit_signature = permit

            result = self.client.post_order(order)
            outcome_str = "YES" if outcome == Outcome.YES else "NO"
            print(f"Placed {outcome_str} order @ {price / 10000:.1f}% (confidence: {confidence:.0%})")

            # Track position
            self.current_position += ORDER_SIZE if outcome == Outcome.YES else -ORDER_SIZE
            self.active_orders[order.order_hash] = action

        except TurbineApiError as e:
            print(f"Order failed: {e}")

    async def price_action_loop(self) -> None:
        """Main loop that monitors price and executes trades."""
        while self.running and self.market_id:
            try:
                action, confidence = self.calculate_signal()
                if action != "HOLD":
                    await self.execute_signal(action, confidence)
                await asyncio.sleep(PRICE_POLL_SECONDS)
            except Exception as e:
                print(f"Price action error: {e}")
                await asyncio.sleep(PRICE_POLL_SECONDS)
```

**Key points for Price Action Trader:**
- Uses Pyth Network Hermes API (same oracle Turbine uses) to get real-time BTC price
- Compares current price to strike price (stored in `quick_market.start_price`)
- If BTC > strike by threshold → buy YES
- If BTC < strike by threshold → buy NO
- Confidence scales with distance from strike (capped at 90%)
- Polls price every 10 seconds by default

### Simple Spread Market Maker
```python
SPREAD_BPS = 200  # 2% total spread (1% each side)

def calculate_quotes(self, mid_price):
    """Calculate bid/ask around mid price."""
    half_spread = (mid_price * SPREAD_BPS) // 20000
    bid = max(1, mid_price - half_spread)
    ask = min(999999, mid_price + half_spread)
    return bid, ask
```

### Inventory-Aware Market Maker
```python
SPREAD_BPS = 200
SKEW_FACTOR = 50  # BPS skew per share of inventory

def calculate_quotes(self, mid_price):
    """Skew quotes based on inventory."""
    half_spread = (mid_price * SPREAD_BPS) // 20000

    # Skew to reduce inventory
    inventory_shares = self.current_position / 1_000_000
    skew = int(inventory_shares * SKEW_FACTOR)

    bid = max(1, mid_price - half_spread - skew)
    ask = min(999999, mid_price + half_spread - skew)
    return bid, ask
```

### Momentum Following
```python
MOMENTUM_WINDOW = 10  # Number of trades to consider
MOMENTUM_THRESHOLD = 0.6  # 60% same direction = trend

def detect_momentum(self, recent_trades):
    """Detect market momentum from recent trades."""
    if len(recent_trades) < MOMENTUM_WINDOW:
        return None

    buys = sum(1 for t in recent_trades[-MOMENTUM_WINDOW:] if t["side"] == "BUY")
    buy_ratio = buys / MOMENTUM_WINDOW

    if buy_ratio > MOMENTUM_THRESHOLD:
        return "UP"
    elif buy_ratio < (1 - MOMENTUM_THRESHOLD):
        return "DOWN"
    return None
```

### Mean Reversion
```python
REVERSION_THRESHOLD = 50000  # 5% move triggers fade
LOOKBACK_TRADES = 20

def should_fade(self, current_price, recent_trades):
    """Check if price moved enough to fade."""
    if len(recent_trades) < LOOKBACK_TRADES:
        return None

    avg_price = sum(t["price"] for t in recent_trades) / len(recent_trades)
    deviation = current_price - avg_price

    if deviation > REVERSION_THRESHOLD:
        return "SELL"  # Fade the up move
    elif deviation < -REVERSION_THRESHOLD:
        return "BUY"  # Fade the down move
    return None
```

### Probability-Weighted
```python
EDGE_THRESHOLD = 200000  # 20% from 50% = extreme

def find_edge(self, best_bid, best_ask):
    """Look for mispriced extremes."""
    mid = (best_bid + best_ask) // 2
    distance_from_fair = abs(mid - 500000)

    if distance_from_fair > EDGE_THRESHOLD:
        if mid > 500000:
            return "SELL"  # Market too bullish
        else:
            return "BUY"  # Market too bearish
    return None
```

## Automatic Market Transition

**All generated bots automatically handle market transitions.** When a BTC 15-minute market expires:

1. **Detection**: The bot polls every 5 seconds for new markets
2. **Order Cleanup**: All active orders on the expiring market are cancelled
3. **Seamless Switch**: The bot automatically connects to the new market
4. **Continued Trading**: Trading resumes on the new market without manual intervention

**How it works:**
- A background task (`monitor_market_transitions`) runs continuously
- It compares the current market ID with the active market from the API
- When a new market is detected, `switch_to_new_market()` handles the transition
- Positions carry over (they're wallet-based), but orders must be re-placed

**Warning before expiration:**
- When less than 60 seconds remain, the bot logs a warning
- Orders are cancelled proactively to avoid stuck orders on expired markets

## USDC Permit Signatures (Gasless Trading)

**Every order must include a USDC permit signature** for gasless execution. Without this, orders will fail with "ERC20: transfer amount exceeds allowance".

The `TurbineClient` provides `sign_usdc_permit()` to create EIP-2612 permit signatures:

```python
async def place_quotes(self) -> None:
    """Place bid and ask orders with USDC permit signatures."""
    bid_price, ask_price = self.calculate_quotes()

    # Place bid (buy YES)
    bid_order = self.client.create_limit_buy(
        market_id=self.market_id,
        outcome=Outcome.YES,
        price=bid_price,
        size=ORDER_SIZE,
        expiration=int(time.time()) + QUOTE_REFRESH_SECONDS + 60,
        settlement_address=self.settlement_address,
    )

    # Calculate permit amount for BUY orders:
    # (size * price / 1e6) + 1% fee + 10% safety margin
    buyer_cost = (ORDER_SIZE * bid_price) // 1_000_000
    total_fee = ORDER_SIZE // 100  # 1% fee
    permit_amount = ((buyer_cost + total_fee) * 110) // 100

    # Sign and attach USDC permit
    permit = self.client.sign_usdc_permit(
        value=permit_amount,
        settlement_address=self.settlement_address,
    )
    bid_order.permit_signature = permit

    result = self.client.post_order(bid_order)

    # Place ask (sell YES)
    ask_order = self.client.create_limit_sell(
        market_id=self.market_id,
        outcome=Outcome.YES,
        price=ask_price,
        size=ORDER_SIZE,
        expiration=int(time.time()) + QUOTE_REFRESH_SECONDS + 60,
        settlement_address=self.settlement_address,
    )

    # Calculate permit amount for SELL orders: size + 10% margin
    permit_amount = (ORDER_SIZE * 110) // 100

    permit = self.client.sign_usdc_permit(
        value=permit_amount,
        settlement_address=self.settlement_address,
    )
    ask_order.permit_signature = permit

    result = self.client.post_order(ask_order)
```

**Key points:**
- BUY orders need permit for: `(size * price / 1e6) + fee`
- SELL orders need permit for: `size` (for JIT token splitting)
- Always add a 10% safety margin to permit amounts
- Permits are signed per-order with the settlement address as spender

## Automatic Winnings Claiming

**Bots must track markets they've traded in and automatically claim winnings when markets resolve.**

### Implementation Pattern

Add these fields to your bot class:

```python
class MarketMakerBot:
    def __init__(self, client: TurbineClient):
        self.client = client
        self.market_id: str | None = None
        self.settlement_address: str | None = None
        self.contract_address: str | None = None  # Current market contract
        self.current_position = 0
        self.active_orders: dict[str, str] = {}
        self.running = True
        # Track markets we've traded in for claiming winnings
        # market_id -> contract_address
        self.traded_markets: dict[str, str] = {}
```

### Track Markets When Switching

When switching to a new market, save the old market for later claiming:

```python
async def switch_to_new_market(self, new_market_id: str, start_price: int = 0) -> None:
    """Switch liquidity to a new market.

    Args:
        new_market_id: The new market ID.
        start_price: BTC strike price (8 decimals) - used by Price Action Trader.
    """
    old_market_id = self.market_id

    # Track old market for claiming winnings later
    if old_market_id and self.contract_address:
        self.traded_markets[old_market_id] = self.contract_address
        print(f"Tracking market {old_market_id[:16]}... for winnings claim")

    if old_market_id:
        await self.cancel_all_orders()

    self.market_id = new_market_id
    self.strike_price = start_price  # Store for Price Action Trader
    self.active_orders = {}

    # Fetch settlement and contract addresses
    markets = self.client.get_markets()
    for market in markets:
        if market.id == new_market_id:
            self.settlement_address = market.settlement_address
            self.contract_address = market.contract_address
            break

    if start_price:
        print(f"Strike price: ${start_price / 1e8:,.2f}")
```

### Background Task for Claiming

Add a background task that checks for resolved markets and claims winnings:

```python
async def claim_resolved_markets(self) -> None:
    """Background task to claim winnings from resolved markets."""
    while self.running:
        try:
            if not self.traded_markets:
                await asyncio.sleep(30)
                continue

            markets_to_remove = []
            for market_id, contract_address in list(self.traded_markets.items()):
                try:
                    # Check if market is resolved
                    markets = self.client.get_markets()
                    market_resolved = False
                    for market in markets:
                        if market.id == market_id and market.resolved:
                            market_resolved = True
                            break

                    if market_resolved:
                        print(f"\nMarket {market_id[:16]}... has resolved!")
                        print(f"Attempting to claim winnings...")
                        try:
                            result = self.client.claim_winnings(contract_address)
                            tx_hash = result.get("txHash", result.get("tx_hash", "unknown"))
                            print(f"Winnings claimed! TX: {tx_hash}")
                            markets_to_remove.append(market_id)
                        except TurbineApiError as e:
                            if "no winnings" in str(e).lower() or "no position" in str(e).lower():
                                print(f"No winnings to claim for {market_id[:16]}...")
                                markets_to_remove.append(market_id)
                            else:
                                print(f"Failed to claim winnings: {e}")
                except Exception as e:
                    print(f"Error checking market {market_id[:16]}...: {e}")

            # Remove claimed markets from tracking
            for market_id in markets_to_remove:
                self.traded_markets.pop(market_id, None)

        except Exception as e:
            print(f"Claim monitor error: {e}")

        await asyncio.sleep(30)  # Check every 30 seconds
```

### Start the Claim Task

In the `run()` method, start the claim task alongside other background tasks:

```python
async def run(self, host: str) -> None:
    """Main trading loop with automatic market switching and winnings claiming."""
    ws = TurbineWSClient(host=host)

    # Start background tasks
    monitor_task = asyncio.create_task(self.monitor_market_transitions())
    claim_task = asyncio.create_task(self.claim_resolved_markets())

    try:
        # ... main trading loop ...
    finally:
        monitor_task.cancel()
        claim_task.cancel()
```

**Key points:**
- `claim_winnings(contract_address)` uses gasless EIP-712 permits
- The API handles all on-chain redemption via a relayer
- Markets are removed from tracking after successful claim or if no position exists
- Check every 30 seconds to catch resolutions promptly

## Important Notes for Users

- **Risk Warning**: Trading involves risk. Start with small sizes.
- **Testnet First**: Consider testing on Base Sepolia (chain_id=84532) first.
- **Monitor Positions**: Always monitor your bot and have stop-loss logic.
- **Market Expiration**: BTC 15-minute markets expire quickly. Bots handle this automatically!
- **Gas/Fees**: Trading on Polygon has minimal gas costs but watch for fees.
- **Continuous Operation**: Bots are designed to run 24/7, switching between markets automatically.

## Quick Reference

**Price Scaling**: Prices are 0-1,000,000 representing 0-100%
- 500000 = 50% probability
- 250000 = 25% probability

**Size Scaling**: Sizes use 6 decimals
- 1_000_000 = 1 share
- 500_000 = 0.5 shares

**Outcome Values**:
- Outcome.YES (0) = BTC ends ABOVE strike price
- Outcome.NO (1) = BTC ends BELOW strike price

**Strike Price (for Price Action Trader)**:
- Available via `quick_market.start_price` (8 decimals)
- Example: 9500000000000 = $95,000.00
- Current BTC price fetched from Pyth Network (same oracle Turbine uses):
  - URL: `https://hermes.pyth.network/v2/updates/price/latest?ids[]=0xe62df6c8b4a85fe1a67db44dc12de5db330f7ac66b72dc658afedf0f4a415b43`
  - BTC Feed ID: `0xe62df6c8b4a85fe1a67db44dc12de5db330f7ac66b72dc658afedf0f4a415b43`
- If current > strike → buy YES, if current < strike → buy NO
