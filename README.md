# TSA TDI Double Divergence Strategy (TradingView)

Pine Script **strategy** variant using **double divergence** logic on the TSA TDI indicator (without level-based filters in the strategy name).

## Files

| File | Description |
|------|-------------|
| `TSA_TDI_Double_Divergence_Strategy.pine` | Full Pine Script source |
| `README.md` | Documentation (this file) |

## Strategy summary

- Built on TDI RSI / bands / fast MA
- Trades regular divergences with configurable pivot and lookback windows
- Stop loss & take profit in pips
- Overlay on chart with optional MA 50/200 display

## Import into TradingView

1. Copy `TSA_TDI_Double_Divergence_Strategy.pine` into the Pine Editor.
2. Save and add to chart as a **strategy**.
3. Adjust inputs under strategy settings.

## Comparison

| Repo | Focus |
|------|--------|
| [Strategy-1-TSA-TDI-Divergence-](https://github.com/Tahir-581/Strategy-1-TSA-TDI-Divergence-) | Single divergence + candle confirmation |
| **This repo** | Double divergence variant |

## Disclaimer

Not financial advice. Past backtests do not guarantee future performance.

## License

Private — personal use.
