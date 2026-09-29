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
   vectorized in NumPy.
4. Checked where the price that *actually* showed up during the holdout
   window landed within that simulated distribution; its percentile rank.
5. Since I cared about whether the model was well-calibrated in general, not
   just lucky on one stock, I looked at whether those 25 percentiles came
   out roughly uniform, which is what you'd expect from a well-calibrated
   model.

## Conclusion

It wasn't well-calibrated, and the shape of the miscalibration turned out
to be the more interesting part.

16 of the 25 stocks (64%) landed above the 50th percentile, and none landed
below the 10th or above the 90th. A formal test against a uniform
distribution (Kolmogorov–Smirnov) didn't reject the null (p = 0.39), but
with only 25 stocks that test doesn't have much power to catch a real
effect anyway; the skew in the histogram tells you more than the p-value
does here.

So I checked the obvious suspect: was volatility overestimated? Pretty
clearly, yes. For 96% of the stocks, the volatility realized during the
holdout window came in lower than what the training window implied (median
ratio around 0.80). That explains why the simulated ranges were too wide;
but a too-wide, correctly-centered distribution should produce clustering
near the middle, not a skew toward one end. My guess is the drift estimate
was also a bit conservative relative to what these stocks actually did,
though I haven't directly tested that part yet.

## Caveats

- GBM assumes constant volatility and normally distributed returns. Real
  markets have fat tails and volatility clustering that this model doesn't see.
- The 25 stocks are all large, currently-successful companies, so there's
  some survivorship bias baked into the list itself.

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
throughout, so you should get the same numbers I did.
