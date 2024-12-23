NBA Over/Under Sports Betting Model project, plus an additional section on how you might integrate Solana and Grin token into your workflow. The goal is to keep things simple, explain the core ideas, and then outline a basic approach to crypto integration. Enjoy!

NBA Over/Under Sports Betting Model (Beginner-Friendly)
Overview
Author: Graham Pinsent
LinkedIn | GitHub

Tools Used:

Python
Scrapy (web scraping)
Pandas, NumPy (data handling)
scikit-learn (machine learning)
XGBoost (advanced machine learning)
Matplotlib, Seaborn (data visualization)
Goal: Predict the total points (Over/Under) of an NBA game based on past team performance.

“Over/Under” means betting on whether the total points of a game (sum of both teams’ scores) will be over or under a line provided by a sports betting site.
We do not expect to beat the oddsmakers consistently. Instead, this is a data-science exercise to practice regression analysis.
Mean Absolute Error (MAE) will be our main measure of accuracy. The lower the MAE, the better.
Quick Takeaway (TL;DR)
You can’t just say “They scored 130 points last game, so they’ll do it again!” because the data is very noisy.
Even the professional betting lines are off by about 13 points (MAE) on average.
Our best model (XGBoost with hyperparameter tuning) is still about 15% worse than the betting lines—but is 12% better than just blindly guessing a static average.
1. Data Collection & Cleaning
Sources:

Historical NBA game data from Kaggle (by Nathan Lauga).
Betting line history scraped from SDQL using Scrapy.
After combining these datasets, I made sure to:

Fix season year formats (some data had extra digits, e.g. 22016).
Only use data from 2011 onward.
Create a new column point_total = PTS_home + PTS_away.
Example:

python
Copy code
gamesdata["point_total"] = gamesdata["PTS_home"] + gamesdata["PTS_away"]
gamesdata = gamesdata.loc[gamesdata.SEASON >= 2011]
I also engineered features like “average points in the last 50 home or away games” for each team, then averaged them to get a “meanpointtotal” per matchup.

2. Betting Line Scraping & Baseline Calculation
To compare my model’s accuracy, I scraped actual betting lines from SDQL:

python
Copy code
class tableSpider(scrapy.Spider):
    name= 'table'
    start_url = ["https://sportsdatabase.com/nba/..."]
    def parse(self, response):
        for row in response.xpath(...):
            if row.xpath('td[3]//text()').extract_first() == "home":
                yield {
                    'date' : row.xpath('td[1]//text()').extract_first(),
                    'team' : ...
                    'total' : ...
                    'points' : ...
                    'o:points' : ...
                }
Then I calculated the Mean Absolute Error (MAE) of these lines:

python
Copy code
data["real_total"] = data["points"] + data["o:points"]
data["error"] = (data["real_total"] - data["total"]).abs()
mae_lines = data["error"].mean()
print(mae_lines)  # ~13.43
So if the betting lines average an MAE of ~13.43, that sets a baseline for what we’re up against.

3. Preprocessing & Exploratory Data Analysis (EDA)
One-Hot Encoding for home & away teams.
Feature Scaling of the SEASON column (e.g. SEASON - 2010 so 2011 => 1, 2012 => 2, etc.).
Dropping Unnecessary Columns (like direct game scores, which would cause “data leakage”).
Example:

python
Copy code
from sklearn import preprocessing

ohe = preprocessing.OneHotEncoder()
ohe_home = ohe.fit_transform(X_full[["HOME_TEAM_ID"]]).toarray()
ohe_away = ohe.transform(X_full[["VISITOR_TEAM_ID"]]).toarray()

X_full["SEASON"] = X_full["SEASON"] - 2010
X_full = X_full.drop(["HOME_TEAM_ID","VISITOR_TEAM_ID","PTS_home","PTS_away"], axis=1)
Interesting Features:

meanpointtotal = average points from both teams’ last 50 games.
diff = difference between the home team’s average points in their last 50 games and the away team’s. Sometimes we also use the absolute value of diff to see if large mismatches matter.
4. Model Training & Validation
4.1 Train/Test Split
No random shuffle—because we want to simulate real predictive scenarios (we train on older seasons, predict on newer seasons).
90% of data is used for training, 10% of the newest data is used for validation.
python
Copy code
y = X_full["point_total"]
X_full = X_full.drop(columns=['point_total'])

test_size = int(len(X_full) * 0.1)
X_train = X_full.iloc[:-test_size]
X_valid = X_full.iloc[-test_size:]
y_train = y.iloc[:-test_size]
y_valid = y.iloc[-test_size:]
4.2 Models & Mean Absolute Error
I tried various regression models:

Linear Models: Lasso, Ridge, ElasticNet, TheilSen, LinearRegression
Neural Network: MLPRegressor
XGBoost (both gblinear and gbtree variants)
Hyperparameter Tuning (using the hpsklearn library + hyperopt)
Example:

python
Copy code
from sklearn.linear_model import Lasso
ls = Lasso().fit(X_train, y_train)
pred_ls = ls.predict(X_valid)
mae_ls = mean_absolute_error(pred_ls, y_valid)
print(mae_ls)  # ~15.94
After tuning, my best model was an XGBoost Regressor with an MAE of around 15.71.

5. Results
Here’s a sorted list of my final MAE scores:

Model	MAE Score
Betting lines	13.428539
Hyperopt XGBoost	15.707999
Lasso	15.939184
ElasticNet	15.979897
XGBoost (basic)	16.022314
TheilSen	16.033545
LinearRegression	16.052512
Ridge	16.059752
MLPRegression	16.590032
Guessing the mean	17.865045
Key Observations
Betting lines (13.43 MAE) are still much better than our best approach (15.71).
Even so, we are ~12% better than just guessing the same average every game.
Predictions are noisy because total points vary a lot from game to game.
6. Visualizations
Plots of model predictions vs. actual outcomes show the model’s estimates hover around the 200–220 range, while real outcomes can spike up or down more dramatically.
Overtime games often inflate real totals, pushing the model to under-predict.
7. Future Improvements
Focus on a single team (e.g., only the Lakers) to see if specialized features help.
Remove or adjust overtime games so they don’t skew results.
Try more advanced time-series approaches (e.g., “leave-future-out” cross-validation).
8. Integrating Solana & Grin Token
If you’re interested in using this model with a blockchain-based payment or rewards system (e.g., you want users to stake Grin or Solana tokens on predictions), here’s a very high-level outline:

8.1 Why Integrate Crypto?
Trustless: Results (predictions) and payouts can be tracked on-chain, so there’s a public record of all wagers and outcomes.
Micro-Payments: Crypto makes small bets/fees easier to manage programmatically.
Global Access: No typical restrictions of bank-based transactions (although always check your local laws about gambling).
8.2 Basic Steps
Set Up a Wallet

For Solana, you can use the Solana Python SDK.
For Grin, you’d typically run a Grin node or use a Grin wallet service.
Smart Contract / Program

On Solana, you can write a “Program” (like a smart contract) in Rust (or sometimes C/C++). This program can:
Track open bets (with an ID or reference to the game).
Escrow user funds (in SOL or an SPL token).
Release payouts to winners after the game result is known.
Off-Chain Model & Oracles

Your Python code runs the NBA Over/Under model off-chain.
After the game finishes, you use an oracle (like a trusted API or data feed) to push final scores on-chain so the contract knows who won.
Workflow Example

User calls your Solana Program to “place a bet” on Over/Under.
The Program locks their tokens in an escrow account.
After the game ends:
Your Python script verifies the final score.
An “oracle transaction” is sent to the Solana Program with the final total points.
The Program compares final points to the Over/Under line, then automatically releases tokens to winners.
Using Grin

Because Grin is a privacy-focused coin (uses Mimblewimble), it does not have “smart contracts” in the same sense as Ethereum or Solana.
You’d typically handle bet logic off-chain (in your Python server or app), then just accept or send Grin payments upon bet settlement.
The steps might look like:
Front-end: Player chooses Over/Under, sends X Grin to your address.
Back-end:
Wait for transaction confirmation.
Once the game ends, use your Over/Under model to see if user bet was correct.
If user wins, pay them from your Grin wallet.
8.3 Technical Pointers
Solana Python: Check out solana-py for building transactions, sending tokens, and interacting with Solana programs.
Grin: Look at the official Grin documentation for setting up a node and wallet.
Security: Always ensure your private keys are properly secured, and your oracle data feed is trustworthy.
Legal: Make sure you’re complying with local regulations regarding sports betting.
9. Closing Thoughts
Predicting NBA Over/Under is tough—there’s a lot of random variation (injuries, game pace, overtime, etc.).
Even with advanced models, the house’s lines are hard to beat.
If you integrate Solana or Grin tokens, you can automate payouts and potentially offer a novel betting experience—but do so carefully and legally.
Thanks for reading! If you have any questions or feedback, feel free to reach out on my GitHub or LinkedIn. Special thanks to everyone who helped along the way.

End of Beginner-Friendly Guide












ChatGPT can make mistakes. Check important info.
