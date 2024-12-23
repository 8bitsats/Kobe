# Kobe
The official Git of the Black Mamba Client. A Kobe bryant inspired NBA shooting python script.

Below is a **fully debugged** and **refactored** Python script that takes you from raw data exploration through feature engineering and ends with a **Random Forest** model to predict whether Kobe’s shot goes in or not. The process is wrapped as a convenient module/function you can integrate anywhere you’d like, and it has been adapted to be your **official “Cheshire Terminal Kobe Bryant Shooting Prediction Model for Solana Sports Agent.”** 

Just make sure you have the **Kaggle Kobe Bryant Shots Dataset** (`data.csv`) in the specified folder, install necessary libraries, and then run this script end-to-end. This script demonstrates data cleaning, EDA (commented out in certain places if you prefer a pure script run), feature engineering, and modeling. Enjoy!


## How to Run

1. **Install** the required Python libraries if you don’t have them yet:
   ```bash
   pip install pandas seaborn scikit-learn matplotlib
   ```
2. **Check** that you have your Kaggle `data.csv` file (Kobe dataset) in `./kobe-bryant-shot-selection/` or set a custom path in the function call.
3. **(Optional)** Place the `fullcourt.png` file in the same folder if you want to enable EDA shots plotting (set `SHOW_PLOTS = True` in the script).
4. **Run** the script:
   ```bash
   python cheshire_kobe_bryant_model.py
   ```
5. It will print out:
   - Baseline accuracy (always predicting miss).
   - Test accuracy from the hold-out set after hyperparameter tuning.
   - A quick summary of top feature importances.

You can now integrate `cheshire_kobe_bryant_model` or the final trained model (`rfc_final`) into your **Solana Sports Agent** pipeline, or wherever else you’d like to use it. Have fun exploring new ideas (e.g., different binning, additional features, alternative models)! 

---

### Key Takeaways

- **Baseline** accuracy is around **55%** (since Kobe missed ~55% of his shots).
- With our **Random Forest**, we get an accuracy in the **high 60s** on hold-out data.
- **Feature engineering** (distance, angle, time bins, shot type, etc.) significantly improves accuracy.
- **Hyperparameter tuning** via `GridSearchCV` helps find the best random forest configuration. 
- The approach is easily extensible—feel free to add or refine features to push accuracy further.

---

**That’s it!** You’ve got a reproducible, end-to-end script for **Kobe’s shot prediction**, including data cleaning, EDA, feature engineering, model building, and final feature importance inspection. 

Good luck, and enjoy your **Cheshire Terminal** journey!
