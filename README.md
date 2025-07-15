# DeFi Credit Scoring System

> **TL;DR**: We built a credit scoring system for DeFi wallets that combines good old-fashioned financial analysis with machine learning. It analyzes Aave transactions to give wallets a credit score from 0-1000. Think FICO score, but for your crypto wallet.

## What This Thing Actually Does

Ever wonder if that wallet asking for a loan is trustworthy? This system digs through DeFi transaction history and spits out a credit score. We're talking about real on-chain behavior - how often they repay loans, whether they get liquidated, if they're running bot operations, or abusing flash loans.

The magic happens in two phases:
1. **Traditional scoring** using financial rules that actually make sense
2. **ML enhancement** that finds patterns we didn't think of

## Why We Built It This Way

Most credit scoring in DeFi is either completely manual or uses basic transaction counts. That's like judging someone's driving by how many miles they've driven. We wanted something that actually looks at *how* they use DeFi protocols.

The hybrid approach came from a simple realization: domain experts know what matters in DeFi (liquidations = bad, consistent repayments = good), but ML is better at finding complex relationships between dozens of variables.

Here's the kicker - we use our rule-based system to train the ML models. The traditional scorer becomes the teacher, and the ML models learn to replicate and improve on that knowledge.

## How The Scoring Actually Works

### The Core Algorithm (Traditional Approach)

Every wallet starts at 500 points. From there:

**Reliability (40% of score)**
- Repayment rate: Do they actually pay back loans? 
- Consistency: Are they regular users or just gambling?

**Activity (30% of score)**  
- Transaction volume: More skin in the game = better
- Frequency: Active users understand the protocols
- Asset diversity: Using multiple tokens shows sophistication

**Risk Management (20% of score)**
- Liquidation history: Getting liquidated hurts, a lot
- Bot detection: Automated trading patterns are penalized
- Flash loan abuse: Exploiting protocols for quick profits

**Engagement (10% of score)**
- Time active: Long-term users are more reliable
- Transaction count: More experience = better understanding

### The Penalty System

Some behaviors are automatic red flags:

| What You Did | Points Lost | Why It Hurts |
|--------------|-------------|--------------|
| Got liquidated | -200 each | Shows poor risk management |
| Bot-like behavior | -100 | Automated trading lacks real intent |
| Flash loan abuse | -150 | Exploiting protocols isn't trustworthy |
| Erratic patterns | -50 | Unpredictable behavior |

### Feature Engineering (Where It Gets Interesting)

We extract 25+ features from transaction history. Some obvious ones:

```python
# Financial health
repay_rate = total_repayments / total_borrows
utilization_rate = borrowed_amount / deposited_amount
net_position = deposits - borrows

# Risk indicators  
liquidation_rate = liquidations / total_transactions
consecutive_liquidations = count_liquidation_streaks()

# Behavioral patterns
regularity_score = 1 / (1 + coefficient_of_variation)
asset_diversity = unique_assets / total_transactions
```

Some not-so-obvious ones that ML helped us discover:

```python
# Temporal patterns
weekend_trading_ratio = weekend_txs / total_txs
hour_diversity = unique_hours_active
activity_clustering = temporal_transaction_patterns()

# Advanced risk metrics
recovery_ability = deposits_after_liquidation / liquidation_events
large_transaction_exposure = big_trades / total_trades
volume_volatility = std(transaction_sizes)
```

## The ML Enhancement Layer

Once we have features and scores from the traditional method, we train multiple ML models:

- **Random Forest**: Great for finding feature interactions
- **Gradient Boosting**: Excellent at handling complex patterns  
- **Linear/Ridge/Lasso**: Good baselines and interpretability
- **SVR**: Handles non-linear relationships well

The system automatically picks the best performer based on cross-validation R² scores. Usually Random Forest or Gradient Boosting win.

### Why Multiple Models?

Different algorithms catch different things. Random Forest might excel at finding feature interactions, while Gradient Boosting captures subtle non-linear patterns. We let them compete and pick the winner.

## Pattern Detection Deep Dive

### Bot Detection Algorithm

```python
def looks_like_bot(transactions):
    # Same amounts repeated frequently
    if any_amount_appears_more_than_5_times():
        return True
    
    # Suspiciously regular timing
    time_gaps = calculate_time_differences()
    if coefficient_of_variation(time_gaps) < 0.1:
        return True
        
    # Repeated action sequences
    if find_repeated_3_action_sequences():
        return True
        
    return False
```

### Flash Loan Abuse Detection

```python
def detect_flash_abuse(transactions):
    for i in range(len(transactions) - 1):
        current = transactions[i]
        next_tx = transactions[i + 1]
        
        # Large deposit → immediate withdrawal pattern
        if (current.action == 'deposit' and 
            next_tx.action == 'withdraw' and
            time_diff < 5_minutes and
            amount > $10k):
            return True
            
    return False
```

### Recovery Analysis

After liquidations, do they come back? Users who deposit again within 7 days show resilience:

```python
def calculate_recovery_rate(liquidation_events):
    recoveries = 0
    for liquidation in liquidation_events:
        future_deposits = find_deposits_within_week(liquidation.timestamp)
        if future_deposits:
            recoveries += 1
    return recoveries / len(liquidation_events)
```

## Data Flow Architecture

```
Raw Transactions (JSON)
         ↓
Data Preprocessing
- Handle token decimals
- Convert to USD values  
- Clean timestamps
         ↓
Feature Extraction  
- Financial metrics
- Risk indicators
- Behavioral patterns
         ↓
Traditional Scoring
- Apply weighted algorithm
- Generate base scores
         ↓
ML Training Data Prep
- Use traditional scores as targets
- Feature engineering
- Train/test split
         ↓
Model Training & Selection
- Train 6 different models
- Cross-validation
- Hyperparameter tuning
- Pick best performer
         ↓
Final Predictions
- Generate enhanced scores
- Feature importance analysis
- Performance visualization
```

## Real-World Usage

### Basic Scoring
```python
scorer = CreditScorer()
df = scorer.load_data("transactions.json")
features = scorer.analyze_wallets(df)
scores = scorer.calculate_scores(features)

# Get top performers
top_wallets = scores.head(10)
```

### ML-Enhanced Analysis
```python
predictor = DeFiCreditMLPredictor()
df = predictor.load_data("transactions.json")
features = predictor.extract_ml_features(df)
targets = predictor.calculate_target_scores(features)

X, y = predictor.prepare_ml_data(features, targets)
results = predictor.train_models(X, y)

# See which model performed best
print(f"Best model: {predictor.best_model_name}")
print(f"R² score: {results[predictor.best_model_name]['cv_r2_mean']:.4f}")
```

### Production Deployment
```python
# Save the trained model
predictor.save_model("production_model.pkl")

# Later, load and score new wallets
predictor.load_model("production_model.pkl")
new_scores = predictor.predict_scores(new_wallet_features)
```

## What The Output Looks Like

### Credit Score Categories
- **800-1000**: Excellent (DeFi natives, consistent repayers)
- **600-799**: Good (Solid users, minor issues)
- **400-599**: Fair (Average behavior, some red flags)
- **200-399**: Poor (Multiple liquidations, risky patterns)
- **0-199**: Very Poor (Frequent liquidations, clear abuse)

### Performance Metrics
After training, you get comprehensive model performance:
- R² scores showing how well models explain variance
- RMSE indicating prediction accuracy
- Feature importance revealing what matters most
- Cross-validation scores ensuring generalization

## Feature Importance Insights

Usually the top predictors are:
1. **Repayment rate** - Obvious but crucial
2. **Total volume** - More activity = more data = better predictions
3. **Liquidation count** - Red flag that heavily impacts score
4. **Regularity score** - Consistent users are predictable
5. **Asset diversity** - Sophisticated users score better

Surprising findings:
- **Weekend trading ratio** often matters more than expected
- **Recovery ability** is a strong positive signal
- **Transaction size volatility** correlates with risk-taking

## Technical Requirements

### Dependencies
```bash
pip install pandas numpy scikit-learn matplotlib seaborn joblib
```

### Data Format
Input JSON should have these fields per transaction:
```json
{
  "userWallet": "0x...",
  "timestamp": 1640995200,
  "action": "deposit",
  "actionData": {
    "amount": "1000000000000000000",
    "assetSymbol": "USDC", 
    "assetPriceUSD": "1.0"
  }
}
```

### Performance Expectations
- **Training time**: ~30 seconds for 1000 wallets
- **Prediction time**: <1 second for new wallets
- **Memory usage**: ~100MB for model storage
- **Accuracy**: Typically 85-95% R² on validation data

## Known Limitations

**Data dependency**: Quality in = quality out. Garbage transaction data gives garbage scores.

**Protocol specific**: Currently tuned for Aave. Other protocols might have different risk patterns.

**Historical bias**: Based on past behavior, doesn't predict future black swan events.

**Gaming resistance**: Sophisticated users could potentially game the system if they understand the algorithm.

**Market conditions**: Doesn't account for broader market volatility affecting liquidation rates.

## Future Improvements

**Real-time scoring**: Currently batch-based, could move to streaming

**Multi-protocol**: Expand beyond Aave to Compound, MakerDAO, etc.

**Cross-chain**: Support for Polygon, Arbitrum, other L2s

**Governance signals**: Include DAO voting behavior as trust signal

**MEV detection**: Identify and potentially penalize MEV strategies

**Deep learning**: Neural networks might catch even more subtle patterns

## Files Generated

Running the system produces:
- `wallet_scores.csv` - Basic scores for all wallets
- `detailed_wallet_analysis.csv` - Full feature breakdown
- `credit_score_analysis.png` - Score distribution charts
- `top_bottom_analysis.png` - Best vs worst performer comparison  
- `feature_importance.png` - ML insights on what matters most
- `defi_credit_model.pkl` - Trained model for production use

## Installation & Setup

```bash
git clone [your-repo]
cd defi-credit-scoring
pip install -r requirements.txt

# Run basic scoring
python credit_scorer.py

# Run ML-enhanced version  
python ml_credit_predictor.py
```
