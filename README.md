# Monte Carlo Simulation of Equity Price Dynamics

I wanted to test something fairly simple: if you calibrate a Geometric
Brownian Motion model on a stock's own historical returns, does it actually
give you a realistic picture of where the price could go over the next six
months? And if it doesn't, why not?

## Methodology

For 25 stocks spread across a handful of sectors, I:

1. Pulled daily closing prices over three years [from January 1, 2021 to January 1, 2024] and split the history into a training window
   and a holdout window of the most recent ~126 trading days (about six
   months).
2. Estimated drift (μ) and volatility (σ) for each stock from its training
   window's daily log-returns.
3. Simulated 5,000+ forward price paths per stock using those parameters,
   vectorized in NumPy. Each daily step multiplies the price by
   exp(μ + σZ), where μ is the mean daily *log* return, so the Itô
   correction is already built in.
4. Checked where the price that *actually* showed up during the holdout
   window landed within that simulated distribution; its percentile rank.
5. Since I cared about whether the model was well-calibrated in general, not
   just lucky on one stock, I looked at whether those 25 percentiles came
   out roughly uniform, which is what you'd expect from a well-calibrated
   model.

## Conclusion

It wasn't well-calibrated, but the problem was width, not direction.

The realized prices bunched up in the middle of the simulated
distributions. 24 of the 25 stocks landed between the 10th and 90th
percentiles, where a well-calibrated model would put about 20; only one
(PFE) fell below the 10th and none above the 90th. If the stocks were
independent, 24 or more inside that band would happen only about 3% of the
time. A formal test against a uniform distribution (Kolmogorov-Smirnov)
didn't reject the null (p = 0.31), but with only 25 stocks that test
doesn't have much power, and it isn't designed to pick up this particular
pattern.

The cause is volatility. For 96% of the stocks, the volatility realized
during the holdout window came in lower than what the training window
implied (median ratio around 0.80). The training window includes 2022, a
volatile bear market, while the second half of 2023 was much calmer, so the
simulated ranges were too wide and the outcomes clustered toward the
middle.

Direction looks roughly balanced: 14 of 25 stocks landed above the median,
which is consistent with chance.

**A correction.** An earlier version of the notebook subtracted the σ²/2
term from a drift that was already a mean log return, so the correction was
applied twice. That shifted every simulated distribution down and made the
outcomes look skewed upward (16 of 25 above the median). Fixing it removed
most of that skew.

## Caveats

- GBM assumes constant volatility and normally distributed returns. Real
  markets have fat tails and volatility clustering that this model doesn't see.
- The 25 stocks are all large, currently-successful companies, so there's
  some survivorship bias baked into the list itself.
- All 25 stocks share the same six-month holdout window, so their outcomes
  are correlated through the overall market and are not 25 independent
  tests. Repeating the test over many non-overlapping windows would be more
  convincing.

## Running this

You'll need:
```
numpy
pandas
matplotlib
scipy
yfinance
```

```
pip install numpy pandas matplotlib scipy yfinance
```

Open the notebook and run it top to bottom. There's a fixed random seed
throughout, so the simulations are reproducible. The price data is
downloaded live from Yahoo Finance, though, and adjusted prices get revised
over time, so a rerun may give slightly different numbers from mine.
