import math
import matplotlib.pyplot as plt
import numpy as np


x_axis_data_label = [
'Mar 25',
'Apr 25',
'May 25',
'Jun 25',
'Jul 25',
'Aug 25',
'Sep 25',
'Oct 25',
'Nov 25',
'Dec 25',
'Jan 26',
'Feb 26',
'Mar 26'
]

# -- Incoming/new 
intake = [
120, # Mar 2025 Q1
120, # Apr 2025 Q2
120, # May 2055 Q2
150, # Jun 2025 Q2
150, # Jul 2025 Q3
150, # Aug 2025 Q3
150, # Sep 2025 Q3
150, # Oct 2025 Q4
150, # Nov 2025 Q4
160, # Dec 2025 Q4
160, # Jan 2026 Q1
160, # Feb 2026 Q1
160 # Mar 2026 Q1
]

# -- Team Size
# -- Number of techs (fractional amounts = not fully trained yet, etc)
team_size = [
6.0, # Mar 2025 Q1 
6.5, # Apr 2025 Q2
7.0, # May 2055 Q2
7.0, # Jun 2025 Q2
8.0, # Jul 2025 Q3
8.0, # Aug 2025 Q3
8.0, # Sep 2025 Q3
9.0, # Oct 2025 Q4
9.0, # Nov 2025 Q4
9.0, # Dec 2025 Q4
9.0, # Jan 2026 Q1
9.0, # Feb 2026 Q1
9.0 # Mar 2026 Q1
]

# -- Pool size is the starting amount of nodes that need to be fixed; Start ref = 972 Mar 2025
# -- Pool size for future months is calculated (see calculation section)
pool_size = [
972 # Mar 2025 Q1
]

# -- List to keep track of month gross burndown (repaired nodes not counting intake)
gross_burndown = []


tech_nodes_per_week = 7.0  # -- Avg nodes per technician per week
weeks_per_month = 4.0

# -- Calculate month by month
for idx in range(0, len(x_axis_data_label)):
    # Gross burndown is raw number of nodes fixed by techs
    gross_burndown_val = team_size[idx] * tech_nodes_per_week * weeks_per_month
    gross_burndown_val = math.floor(gross_burndown_val) # round to closest integer
    gross_burndown.append(gross_burndown_val) # store value in list for graphing later

    # Net burndown gross burndown but include incoming new nodes("intake")
    net_burndown = gross_burndown_val - intake[idx]

    # Pool size decreases each month by net_burndown
    new_pool_size = (pool_size[idx] - net_burndown)
    if new_pool_size < 0:
        new_pool_size = 0;
    pool_size.append(new_pool_size) # Store value for graphing later

    print(x_axis_data_label[idx])
    print("Starting pool size: ", pool_size[idx])
    print("Team Size: ", team_size[idx])
    print("Gross burndown: ", gross_burndown[idx])
    print("Intake: ", intake[idx])
    print("Net Burndown: ", net_burndown)
    print("End of month pool size now:", new_pool_size)
    print()


last_item = pool_size.pop()  # Removes last value (so graphing data all the same length)

print(len(x_axis_data_label))
print(len(pool_size))
print(len(gross_burndown))
print(len(intake))

# ------------------------------------------------------------------- GRAPH IT

# Create a numeric x-axis based on the length of the labels
x = np.arange(len(x_axis_data_label))

# Create subplots with custom height ratios: 80% for the top and 20% for the bottom.
fig, (ax1, ax2) = plt.subplots(
    2, 1,
    figsize=(12.8, 9.0),
    dpi=100,
    gridspec_kw={'height_ratios': [4, 1]}
)
fig.tight_layout(pad=4.0)  # Adjust layout spacing

# ---------------------------
# Top Plot: Node Burndown Targets
# ---------------------------
ax1.set_axisbelow(True)
ax1.grid(axis='y', color='lightgray')

# Bar plot for pool_size on the top subplot (solid blue bars)
bars = ax1.bar(x, pool_size, width=0.8, label="Pool Size")

# Annotate each bar with the current value, change from the previous bar, and percent change from the previous bar
for i, bar in enumerate(bars):
    height = bar.get_height()
    if i == 0:
        annotation = f'{height}'
    else:
        prev_height = pool_size[i - 1]
        delta = height - prev_height
        if prev_height != 0:
            pct_change = (delta) / prev_height * 100
            annotation = f'{height}\n({delta:+}, {pct_change:+.1f}%)'
        else:
            annotation = f'{height}\n({delta:+}, N/A)'
    ax1.text(
        bar.get_x() + bar.get_width() / 2, 
        height, 
        annotation, 
        ha='center', 
        va='bottom'
    )

# Line plots for gross_burndown and intake
#ax1.plot(x, gross_burndown, color='green', marker='o', label="Gross Burndown")
ax1.plot(x, gross_burndown, color='#00bb00', marker='o', label="Gross Burndown")
ax1.plot(x, intake, color='red', marker='o', label="Intake")

ax1.set_ylim(0, 1000)
ax1.set_ylabel('Node Pool')
ax1.legend()

# Set x-ticks and labels for the top plot
ax1.set_xticks(x)
ax1.set_xticklabels(x_axis_data_label)

# ---------------------------
# Bottom Plot: Team Size
# ---------------------------
ax2.set_axisbelow(True)
ax2.grid(axis='y', color='lightgray')

# Bar plot for team_size on the bottom subplot
bars2 = ax2.bar(x, team_size, width=0.8, label="Team Size", color='skyblue')

# Annotate each bar with its value above the bar
for bar in bars2:
    height = bar.get_height()
    ax2.text(
        bar.get_x() + bar.get_width() / 2,
        height,
        f'{height}',
        ha='center',
        va='bottom'
    )

ax2.set_ylim(0, 14)
ax2.set_ylabel('Team Size')
ax2.set_title("Team Size")

# Set x-ticks and labels for the bottom plot
ax2.set_xticks(x)
ax2.set_xticklabels(x_axis_data_label)

# Set the overall title of the figure using the tech_nodes_per_week variable
fig.suptitle(f"Node Burndown Targets, Rate = {tech_nodes_per_week} nodes/week per tech", fontsize=16)

# Save the combined plot as an image file
plt.savefig('my_combined_plot.png')

# Display the plot
plt.show()
