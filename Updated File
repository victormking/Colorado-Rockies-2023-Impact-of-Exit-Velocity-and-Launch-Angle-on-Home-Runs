from IPython.core.interactiveshell import InteractiveShell
InteractiveShell.ast_node_interactivity = "all"

import pybaseball as pyball
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Set up global plotting styles
sns.set(style="whitegrid")
plt.style.use('seaborn')

# Colorado Rockies Game Records for 2023
def fetch_game_records(team, year):
    df = pyball.schedule_and_record(year, team)
    df['Date'] = df['Date'].str.replace(r'\w+day, ', '', regex=True) + f' {year}'
    df['Date'] = pd.to_datetime(df['Date'], errors='coerce')
    return df

COL_game_records_2023 = fetch_game_records("COL", 2023)

# Function for player lookup
def get_player_ids(players):
    player_ids = {}
    for last, first in players:
        result = pyball.playerid_lookup(last, first)
        if not result.empty:
            player_ids[last] = result.iloc[0]['key_mlbam']
    return player_ids

# List of (last, first) tuples for Rockies players
rockies_players = [
    ('mcmahon', 'ryan'), ('hilliard', 'sam'), ('bouchard', 'sean'),
    ('jones', 'nolan'), ('toglia', 'michael'), ('bryant', 'kris'),
    ('stallings', 'jacob'), ('goodman', 'hunter'), ('tovar', 'ezequiel'),
    ('diaz', 'elias'), ('montero', 'elehuris'), ('blackmon', 'charlie'),
    ('doyle', 'brenton'), ('rodgers', 'brendan')
]

player_ids = get_player_ids(rockies_players)

# Fetch Statcast batter data for each player
def fetch_statcast_batter_data(player_ids):
    all_data = []
    for player, player_id in player_ids.items():
        data = pyball.statcast_batter('2023-03-30', '2023-11-03', player_id=player_id)
        if data is not None:
            data['player_name'] = player
            all_data.append(data)
    return pd.concat(all_data, ignore_index=True)

COL_batting_records_2023 = fetch_statcast_batter_data(player_ids)

# Combine with game records
COL_batting_records_2023.rename(columns={'game_date': 'Date'}, inplace=True)
combined_stats_col_2023 = pd.merge(COL_batting_records_2023, COL_game_records_2023, on=["Date"], how="left")
combined_stats_col_2023['Games_at_Coors'] = combined_stats_col_2023['Home_Away'].map({'@': '', 'Home': 'Coors'})

# Drop unused columns
unused_columns = [
    'Date_formatted_x', 'Date_formatted_y', 'Home_Away', 'Orig. Scheduled',
    'spin_dir', 'break_angle_deprecated', 'spin_rate_deprecated', 'break_length_deprecated',
    'hit_location'
]
combined_stats_col_2023.drop(unused_columns, axis=1, inplace=True)

# Rename columns for consistency
combined_stats_col_2023.rename(columns={'launch_speed': 'Exit Velocity', 'launch_angle': 'Launch Angle'}, inplace=True)

# Convert to proper data types
combined_stats_col_2023 = combined_stats_col_2023.astype({'Games_at_Coors': 'bool'})

# Save combined stats to CSV
combined_stats_col_2023.to_csv("Statcast practice/combined_stats_col_2023.csv", index=False)

# Generate grouped average stats
avg_stats = combined_stats_col_2023[['Games_at_Coors', 'Exit Velocity', 'Launch Angle']].groupby(['Games_at_Coors']).mean()
avg_stats_visual = avg_stats.copy()
avg_stats_visual['Exit Velocity'] = avg_stats_visual['Exit Velocity'].round(2).astype(str) + ' mph'
avg_stats_visual['Launch Angle'] = avg_stats_visual['Launch Angle'].round(2).astype(str) + '°'

# Visualize average stats
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 6))
avg_stats[['Exit Velocity']].plot(kind='bar', ax=ax1, color='red')
ax1.set_title("Average Exit Velocity")
ax1.set_ylabel("MPH")
ax1.set_xlabel("Field")

avg_stats[['Launch Angle']].plot(kind='bar', ax=ax2, color='blue')
ax2.set_title("Average Launch Angle")
ax2.set_ylabel("Degrees")
ax2.set_xlabel("Field")
plt.tight_layout()
plt.show()

# Density plots for Exit Velocity and Launch Angle
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 6))
combined_stats_col_2023[['Exit Velocity']].plot(kind='density', ax=ax1, color='red')
ax1.set_title("Density Plot of Exit Velocity")
ax1.set_xlabel("MPH")
ax1.set_ylabel("Density")

combined_stats_col_2023[['Launch Angle']].plot(kind='density', ax=ax2, color='blue')
ax2.set_title("Density Plot of Launch Angle")
ax2.set_xlabel("Degrees")
ax2.set_ylabel("Density")
plt.tight_layout()
plt.show()
