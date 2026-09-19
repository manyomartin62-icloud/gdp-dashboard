"""
Starter Paper-Trading / Backtesting Bot
---------------------------------------
This bot DOES NOT place real trades.

It generates a simple moving-average crossover strategy:
- Fast moving average: 10 periods
- Slow moving average: 30 periods
- Buy when the fast MA crosses above the slow MA
- Sell when the fast MA crosses below the slow MA

The script creates its own example price data, so you can run it
without connecting to a broker or providing an API key.
"""

from __future__ import annotations

import random


FAST_PERIOD = 10
SLOW_PERIOD = 30
STARTING_CASH = 10_000.0


def moving_average(values: list[float], period: int) -> float | None:
    """Return the simple moving average of the latest values."""
    if len(values) < period:
        return None
    return sum(values[-period:]) / period


def generate_demo_prices(days: int = 180) -> list[float]:
    """Generate fictional prices for testing the bot."""
    random.seed(42)
    prices = []
    price = 100.0

    for _ in range(days):
        price += random.uniform(-2.0, 2.0)

        # Keep the demo price positive.
        price = max(price, 1.0)
        prices.append(round(price, 2))

    return prices


def run_backtest(prices: list[float]) -> None:
    """Run the moving-average strategy on the supplied prices."""
    cash = STARTING_CASH
    units = 0.0
    previous_fast = None
    previous_slow = None
    trades = []

    for day, price in enumerate(prices, start=1):
        fast_ma = moving_average(prices[:day], FAST_PERIOD)
        slow_ma = moving_average(prices[:day], SLOW_PERIOD)

        if fast_ma is None or slow_ma is None:
            continue

        # Buy only when the fast MA crosses from below to above the slow MA.
        if (
            units == 0
            and previous_fast is not None
            and previous_slow is not None
            and previous_fast <= previous_slow
            and fast_ma > slow_ma
        ):
            units = cash / price
            cash = 0.0
            trades.append((day, "BUY", price))
            print(f"Day {day:3d}: BUY  at ${price:8.2f}")

        # Sell only when the fast MA crosses from above to below the slow MA.
        elif (
            units > 0
            and previous_fast is not None
            and previous_slow is not None
            and previous_fast >= previous_slow
            and fast_ma < slow_ma
        ):
            cash = units * price
            units = 0.0
            trades.append((day, "SELL", price))
            print(f"Day {day:3d}: SELL at ${price:8.2f}")

        previous_fast = fast_ma
        previous_slow = slow_ma

    # Value any open position at the final price.
    final_price = prices[-1]
    final_value = cash + units * final_price

    print("\n--- Backtest Results ---")
    print(f"Starting cash: ${STARTING_CASH:,.2f}")
    print(f"Final value:   ${final_value:,.2f}")
    print(f"Profit/loss:   ${final_value - STARTING_CASH:,.2f}")
    print(f"Trades:        {len(trades)}")


def main() -> None:
    print("Starter Paper-Trading Bot")
    print("No real trades will be placed.\n")

    prices = generate_demo_prices()
    run_backtest(prices)


if __name__ == "__main__":
    main()

