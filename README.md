# Airdrop-FMV-Utility
Two scripts to help with your airdrop taxes (USA) 1. To help aggregate your digital asset txns associated with wallet addresses 2. To attempt a FMV of the token based on available price and liquidity data from Coingeckoterminal and DefiLama

# Airdrop FMV Utility

Open-source tools for cryptocurrency tax reporting (USA). Aggregates multi-chain airdrop transactions and calculates Fair Market Value (FMV) using GeckoTerminal and DeFiLlama APIs with liquidity-based validation.

## ⚠️ IMPORTANT DISCLAIMERS

**NOT TAX, LEGAL, OR FINANCIAL ADVICE**: This software is provided for informational and educational purposes only. It does not constitute tax, legal, or financial advice. Cryptocurrency tax treatment varies significantly by jurisdiction and individual circumstances.

**NO WARRANTY**: This software is provided "AS IS" without warranty of any kind, express or implied. The authors make no guarantees about the accuracy, reliability, or completeness of the data provided by this tool.

**NO LIABILITY**: The authors and contributors are not responsible for any losses, damages, tax penalties, or legal issues arising from the use or misuse of this software. Use at your own risk.

**CONSULT A PROFESSIONAL**: Always consult with a qualified tax professional, CPA, or attorney familiar with cryptocurrency taxation before making any tax reporting decisions. This tool should be used only as a starting point for your own research and verification.

**VERIFY ALL DATA**: All outputs from this tool should be independently verified using blockchain explorers (Solscan, Etherscan, etc.) and cross-referenced with your actual transaction history before use in tax filings.

## What This Tool Does

This project contains two Python scripts designed to assist with cryptocurrency tax reporting:

### 1. Airdrop Aggregator (`airdrop_aggregator.py`)
- Scans wallet addresses across multiple blockchain networks
- Identifies potential airdrop and token receipt transactions
- Filters out spam/scam tokens using liquidity thresholds
- Supports Solana (via Helius API) and EVM chains (via Blockscout API)
- Outputs transaction data in CSV format for further analysis

### 2. FMV Calculator (`fmv_calculator.py`)
- Calculates Fair Market Value (FMV) for cryptocurrency transactions
- Uses multiple data sources: GeckoTerminal, DeFiLlama, Moralis, The Graph
- Implements fallback mechanisms and confidence scoring
- Provides liquidity validation to identify potential scam tokens
- Generates audit-ready reports with source attribution

## What This Tool Does NOT Do

- **Does not provide tax advice** - You must determine the tax treatment of each transaction
- **Does not guarantee accuracy** - API data may be incomplete, outdated, or incorrect
- **Does not detect all scam tokens** - Novel scam techniques may bypass filters
- **Does not replace professional review** - Complex transactions require expert analysis
- **Does not handle all edge cases** - DEX routing, wrapped tokens, and complex DeFi may need manual review

## Known Limitations & Issues

Based on real-world testing, be aware of these issues:

### Scam/Spam Token Detection
- Some scam tokens create fake liquidity pools that appear legitimate
- Tokens with manipulated on-chain data may pass liquidity filters
- Always verify suspicious tokens manually before including in tax reports

### DEX Aggregator Artifacts
- Jupiter (Solana) and other DEX aggregators create intermediate token movements
- These routing artifacts should NOT be included as taxable events
- The tool may flag these - verify transaction details on blockchain explorers

### API Data Quality Issues
- **GeckoTerminal**: Historical data limited to ~180 days, may return wrong pool data
- **DeFiLlama**: Gaps in coverage for newer or low-volume tokens
- **Price mismatches**: Stablecoin pools may return the wrong side of the pair (e.g., TRUMP price instead of USDC price)

### Rate Limiting
- Free API tiers have strict rate limits
- Processing large wallets may require paid API subscriptions
- Built-in delays help but don't eliminate rate limit errors

## Supported Blockchains

### Currently Supported
- **Solana** (via Helius API)
- **Ethereum** (via Blockscout API)
- **Polygon** (via Blockscout API)
- **Arbitrum** (via Blockscout API)
- **Optimism** (via Blockscout API)
- **Base** (via Blockscout API)

### Coming Soon
- Avalanche, BNB Chain, and other EVM chains

## Prerequisites

### Required API Keys
You'll need free API keys from:

1. **Helius** (for Solana): https://www.helius.dev/
2. **Blockscout** (for EVM chains): https://www.blockscout.com/
3. **GeckoTerminal** (optional, for enhanced pricing): https://www.geckoterminal.com/
4. **Moralis** (optional, backup pricing): https://moralis.io/

### Python Requirements
- Python 3.8 or higher
- Required packages (install via `requirements.txt`):
  - requests
  - pandas
  - python-dotenv
  - (add others as needed)

## Installation

1. Clone this repository:
```bash
git clone https://github.com/yourusername/airdrop-fmv-utility.git
cd airdrop-fmv-utility
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Create a `.env` file in the project root with your API keys:
```
HELIUS_API_KEY=your_helius_key_here
BLOCKSCOUT_API_KEY=your_blockscout_key_here
GECKOTERMINAL_API_KEY=your_gecko_key_here
MORALIS_API_KEY=your_moralis_key_here
```

## Usage

### Step 1: Aggregate Airdrops

Edit the script to add your wallet addresses, then run:

```bash
python airdrop_aggregator.py
```

This will generate a CSV file with all potential airdrop transactions.

### Step 2: Calculate FMV

Feed the CSV from Step 1 into the FMV calculator:

```bash
python fmv_calculator.py --input airdrops.csv --output fmv_report.csv
```

### Step 3: Manual Review

**CRITICAL**: Review the output CSV for:
- Tokens with impossibly high liquidity (scam indicators)
- Stablecoins priced incorrectly (should be ~$1.00)
- DEX routing artifacts (intermediate tokens you never controlled)
- Low confidence scores (may need additional research)

## Configuration Options

### Liquidity Thresholds

In `airdrop_aggregator.py`, you can adjust the minimum liquidity filter:

```python
MIN_LIQUIDITY_USD = 10000  # Default: $10,000
```

Lower values will include more tokens (including potential spam).
Higher values will be more conservative but may exclude legitimate small-cap tokens.

### Confidence Scoring

The FMV calculator assigns confidence scores based on:
- **100%**: Price from API matching transaction timestamp
- **80%**: Price from recent data within 7 days
- **60%**: Genesis fallback or >30 days old
- **40%**: Estimated from on-chain liquidity only

Transactions below 60% confidence should be manually verified.

## Output Files

### Airdrop Aggregator Output
- `airdrops_[chain]_[timestamp].csv`
- Contains: date, token address, symbol, quantity, liquidity, source

### FMV Calculator Output
- `fmv_report_[timestamp].csv`
- Contains: all input data plus FMV price, total value, confidence score, data source, flags

## Common Issues & Troubleshooting

### "Rate limit exceeded" errors
- Add delays between API calls (built-in but may need adjustment)
- Upgrade to paid API tiers for higher limits
- Process wallets in smaller batches

### Stablecoins showing wrong prices
- Check if the API returned data from a trading pair pool
- Manually override USDC/USDT/DAI to $1.00 if needed
- This is a known issue being addressed in future versions

### Missing tokens in output
- Check if token liquidity is below the threshold
- Verify the token has active trading pools
- Some tokens may not be indexed by the APIs used

### Scam tokens appearing legitimate
- Cross-reference holder count and trading volume
- Check token age vs. liquidity (new tokens with huge liquidity = red flag)
- Search the token on social media for scam reports

## Recommended Workflow

1. **Run airdrop aggregator** for all your addresses
2. **Review the output** - remove obvious spam before FMV calculation
3. **Run FMV calculator** on the cleaned data
4. **Flag low-confidence transactions** for manual research
5. **Verify suspicious tokens** using blockchain explorers
6. **Cross-reference with your wallet history** to confirm you actually received these tokens
7. **Consult your tax professional** with the final report

## Tax Reporting Considerations (USA)

### Airdrop Tax Treatment
- **Taxable as income** at FMV on the date of receipt (in most cases)
- **Cost basis** equals the FMV at receipt for future sale calculations
- **Scam/spam tokens** may be reported at $0 FMV or excluded (consult CPA)
- **DEX routing artifacts** are NOT taxable events (you never had control)

### Documentation Requirements
- Date and time of receipt
- Fair market value at time of receipt
- Source of FMV determination
- Blockchain transaction hash for verification

This tool helps gather this information but **does not make tax determinations for you**.

## Contributing

Contributions are welcome! Areas for improvement:

- Additional blockchain support
- Better scam token detection algorithms
- Enhanced DEX routing detection
- Alternative API integrations
- UI/frontend development

Please open an issue before submitting major changes.

## Roadmap

- [ ] Frontend interface for non-technical users
- [ ] Automated DEX routing detection (Jupiter, 1inch, etc.)
- [ ] Known scam token database integration
- [ ] Enhanced pool-side verification for stablecoin pairs
- [ ] Support for additional blockchains (Avalanche, BNB, etc.)
- [ ] Integration with popular tax software (CoinTracker, Koinly, etc.)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Built using APIs from GeckoTerminal, DeFiLlama, Helius, Blockscout, and Moralis
- Inspired by the need for transparent, verifiable cryptocurrency tax reporting tools
- Thanks to the open-source crypto development community

## Support & Contact

- **Issues**: Please use the GitHub issue tracker
- **Questions**: Open a discussion in the GitHub Discussions tab
- **Not a support channel for tax advice**: For tax questions, consult a qualified professional

---

**Final Reminder**: This tool is a starting point, not a complete solution. Cryptocurrency taxation is complex and evolving. The data provided by APIs can be incomplete or incorrect. Always verify outputs independently and consult with qualified tax professionals before filing your taxes.

**Your tax situation is your responsibility.** Use this tool wisely and at your own risk.
