
import itertools  # For generating combinations
import random     
import matplotlib.pyplot as plt  # For plotting
import numpy as np   
import pandas as pd  
import os            

all_transactions = []

# -------------------- Utility Functions --------------------

# Generate a dictionary of 10 random transactions
def generate_transactions():
    transactions = {}
    for i in range(1, 11):
        transactions[f'Tx{i}'] = {
            'gas': random.randint(5, 15),  # Gas limit per tx (used in block selection)
            'tip': random.randint(1, 7)    # Tip value per tx (used for profit)
        }
    return transactions

# Get all subsets of txs that fit under a given gas limit
def possible_combinations(tx_list, transactions, gas_limit):
    combos = []
    for r in range(1, len(tx_list)+1):  # For all sizes from 1 to full set
        for subset in itertools.combinations(tx_list, r):
            total_gas = sum(transactions[tx]['gas'] for tx in subset)
            if total_gas <= gas_limit:
                total_tip = sum(transactions[tx]['tip'] for tx in subset)
                combos.append((subset, total_gas, total_tip))
    return combos

# -------------------- Simulation Parameters --------------------
block_limit = 30  # Max block gas limit allowed

# -------------------- Exploitation Simulation --------------------

# Choose best follower response given remaining gas and capital limit
def best_response_exploitation(follower_tx, transactions, remaining_gas, delay_factor=0, capital_limit=None):
    combos = possible_combinations(follower_tx, transactions, remaining_gas)
    if not combos:
        return None, 0  # No valid combos
    if capital_limit is not None:
        combos = [combo for combo in combos if combo[2] <= capital_limit]  # Filter by capital
    if not combos:
        return None, 0  # No combos left after applying capital filter
    if random.random() < delay_factor:
        return random.choice(combos)[0], random.choice(combos)[2]  # Randomized (delayed) choice
    else:
        best = max(combos, key=lambda x: x[2])  # Optimal tip choice
        return best[0], best[2]

# Simulate one round of auction with leader strategy and follower response
def simulate_auction_exploitation(delay_factor=0.3, capital_limit=None):
    transactions = generate_transactions()
    all_transactions.append(transactions)

    # Split txs: leader gets first 6, follower gets last 4
    leader_tx = list(transactions.keys())[:6]
    follower_tx = list(transactions.keys())[6:]

    # Generate all valid leader move sets under block gas constraint
    leader_moves = possible_combinations(leader_tx, transactions, block_limit)
    results = []

    # For each leader move, simulate the best follower response
    for move, used_gas, tip in leader_moves:
        remaining_gas = block_limit - used_gas
        follower_move, follower_tip = best_response_exploitation(
            follower_tx, transactions, remaining_gas, delay_factor, capital_limit)
        results.append({
            'Leader Move': move,
            'Leader Tip': tip,
            'Follower Move': follower_move,
            'Follower Tip': follower_tip,
            'Total Tip': tip + follower_tip
        })

    # Choose leader move with highest tip (greedy strategy)
    best_leader_strategy = max(results, key=lambda x: x['Leader Tip'])
    return best_leader_strategy['Leader Tip'], best_leader_strategy['Follower Tip']

# Run multiple rounds and collect leader and follower payoffs
def run_multiple_auctions_exploitation(rounds=100, delay_factor=0.3, capital_limit=None):
    leader_earnings, follower_earnings = [], []
    for _ in range(rounds):
        l, f = simulate_auction_exploitation(delay_factor, capital_limit)
        leader_earnings.append(l)
        follower_earnings.append(f)
    return leader_earnings, follower_earnings

# -------------------- Latency Simulation (No Capital Constraint) --------------------

# Follower logic (simplified for latency comparison)
def best_response_latency(follower_tx, transactions, remaining_gas, delay_factor=0):
    combos = possible_combinations(follower_tx, transactions, remaining_gas)
    if not combos:
        return None, 0
    if random.random() < delay_factor:
        return random.choice(combos)[0], random.choice(combos)[2]
    else:
        best = max(combos, key=lambda x: x[2])
        return best[0], best[2]

# Simulate one auction round for latency comparison
def simulate_auction_latency(delay_factor=0.3):
    transactions = generate_transactions()
    all_transactions.append(transactions)

    leader_tx = list(transactions.keys())[:6]
    follower_tx = list(transactions.keys())[6:]
    leader_moves = possible_combinations(leader_tx, transactions, block_limit)
    results = []

    for move, used_gas, tip in leader_moves:
        remaining_gas = block_limit - used_gas
        follower_move, follower_tip = best_response_latency(follower_tx, transactions, remaining_gas, delay_factor)
        results.append({
            'Leader Move': move,
            'Leader Tip': tip,
            'Follower Move': follower_move,
            'Follower Tip': follower_tip,
            'Total Tip': tip + follower_tip
        })

    best_leader_strategy = max(results, key=lambda x: x['Leader Tip'])
    return best_leader_strategy['Leader Tip']

# Run latency-only simulation
def run_multiple_auctions_latency(rounds=100, delay_factor=0.3):
    earnings = []
    for _ in range(rounds):
        earnings.append(simulate_auction_latency(delay_factor))
    return earnings

# -------------------- Main Execution --------------------

# Run exploitation model with latency
leader_with_latency, follower_with_latency = run_multiple_auctions_exploitation(delay_factor=0.3)
avantage_with_latency = np.array(leader_with_latency) - np.array(follower_with_latency)

# Run with added capital constraint
leader_with_capital_limit, follower_with_capital_limit = run_multiple_auctions_exploitation(delay_factor=0.3, capital_limit=10)
avantage_with_capital_limit = np.array(leader_with_capital_limit) - np.array(follower_with_capital_limit)

# Compare latency effect only
leader_earnings_with_latency = run_multiple_auctions_latency(delay_factor=0.3)
leader_earnings_without_latency = run_multiple_auctions_latency(delay_factor=0.0)

# -------------------- Plotting --------------------

fig, axs = plt.subplots(7, 1, figsize=(16, 42), gridspec_kw={'hspace': 0.5})

# 1. Raw Earnings: Leader vs Follower
axs[0].plot(leader_with_latency, label="Leader (Latency)", alpha=0.7)
axs[0].plot(follower_with_latency, label="Follower (Latency)", alpha=0.7)
axs[0].set_title("Leader vs Follower Earnings (With Latency)")
axs[0].legend()
axs[0].grid(True)

# 2. Advantage Plot
axs[1].plot(avantage_with_latency, label="Leader Exploitation (Latency)", color='green')
axs[1].set_title("Leader Exploitation Over Follower (With Latency)")
axs[1].legend()
axs[1].grid(True)

# 3. Histogram of Advantage
axs[2].hist(avantage_with_latency, bins=10, alpha=0.8, color='purple')
axs[2].set_title("Distribution of Leader Exploitation (Latency)")
axs[2].grid(True)

# 4. Capital Constraint Effect
axs[3].plot(avantage_with_capital_limit, label="Leader Exploitation (Latency + Capital Limit)", color='red')
axs[3].set_title("Leader Exploitation with Capital Constraint")
axs[3].legend()
axs[3].grid(True)

# 5. Raw Leader Earnings (Latency vs No Latency)
axs[4].plot(leader_earnings_with_latency, label="Leader With Latency (30%)", alpha=0.7)
axs[4].plot(leader_earnings_without_latency, label="Leader Without Latency", alpha=0.7)
axs[4].set_title("Leader Earnings Over Time")
axs[4].legend()
axs[4].grid(True)

# 6. Rolling Average Earnings
window = 10
axs[5].plot(pd.Series(leader_earnings_with_latency).rolling(window).mean(), label=f"Latency (Moving Avg {window})")
axs[5].plot(pd.Series(leader_earnings_without_latency).rolling(window).mean(), label=f"No Latency (Moving Avg {window})")
axs[5].set_title("Moving Average of Leader Earnings")
axs[5].legend()
axs[5].grid(True)

# 7. Histogram Comparison of Leader Earnings
axs[6].hist(leader_earnings_with_latency, bins=10, alpha=0.7, label="With Latency")
axs[6].hist(leader_earnings_without_latency, bins=10, alpha=0.7, label="Without Latency")
axs[6].set_title("Histogram of Leader Earnings")
axs[6].legend()
axs[6].grid(True)

plt.show()

# -------------------- Summary Tables --------------------

summary_exploitation = pd.DataFrame({
    "Latency Only": [
        np.min(avantage_with_latency),
        np.max(avantage_with_latency),
        np.mean(avantage_with_latency),
        np.median(avantage_with_latency)
    ],
    "Latency + Capital Limit": [
        np.min(avantage_with_capital_limit),
        np.max(avantage_with_capital_limit),
        np.mean(avantage_with_capital_limit),
        np.median(avantage_with_capital_limit)
    ]
}, index=["Min", "Max", "Mean", "Median"])

print("\nLeader Exploitation Summary:")
print(summary_exploitation.round(2))

summary_latency = pd.DataFrame({
    "With Latency": [
        np.min(leader_earnings_with_latency),
        np.max(leader_earnings_with_latency),
        np.mean(leader_earnings_with_latency),
        np.median(leader_earnings_with_latency)
    ],
    "Without Latency": [
        np.min(leader_earnings_without_latency),
        np.max(leader_earnings_without_latency),
        np.mean(leader_earnings_without_latency),
        np.median(leader_earnings_without_latency)
    ]
}, index=["Min", "Max", "Mean", "Median"])

print("\nLeader Earnings Summary:")
print(summary_latency.round(2))

# -------------------- Save Transactions to files --------------------

# Create output folder if not exists
os.makedirs("auction_data", exist_ok=True)

# Flatten all transactions to one DataFrame
final_tx_data = pd.concat([
    pd.DataFrame.from_dict(tx, orient='index').assign(Round=i+1)
    for i, tx in enumerate(all_transactions)
])

# Add combined transaction identifier
final_tx_data['R_Transaction'] = final_tx_data.index

# Reorder columns
final_tx_data = final_tx_data[['Round', 'R_Transaction', 'gas', 'tip']]

# Save to files
final_tx_data.to_csv("auction_data/auction_transactions.csv", index=False)
final_tx_data.to_excel("auction_data/auction_transactions.xlsx", index=False)

print("\nAll transaction data saved to 'auction_data/' folder, now showing which round each transaction is from.")
