# solvent-recommender-version-1-
Rule-based solvent recommendation system for organic reactions.
# Solvent Recommendation System (Version 1)
Overview

This project is a rule-based Python system that recommends organic solvents based on reaction type and simple chemical properties such as polarity, reaction environment, and toxicity.
The goal of this project was not to build a production tool, but to learn how to structure a small computational system around a chemistry concept and improve my understanding of Python programming through a real use case.
How it works
The program follows a simple pipeline:
Load data
Solvents are stored in a CSV file (solution.csv)
Each solvent has properties: name, polarity, type, toxicity
Define reaction rules
Reactions (e.g., SN2, E1) are mapped to required solvent conditions:
polarity
solvent type (protic / aprotic)
User input
The user enters a reaction type
Filtering
The system filters solvents that match the reaction requirements
Toxicity conversion
Toxicity labels (low, medium, high) are converted into numerical values for comparison
Sorting
Recommended solvents are sorted by toxicity (lowest to highest)
Output
The final ranked solvent list is displayed
What I learned

This project helped me understand:

How to structure Python programs using functions
How to read and process CSV data
How dictionaries can be used to model real-world rules
How to pass data between functions instead of relying on global variables
Basic filtering and sorting logic in Python
The importance of breaking a problem into stages (pipeline thinking)
⚠️ Limitations (Version 1)
This is an early-stage model with simplifications:
Only a small set of reactions (SN2, E1)
Simple rule-based matching (no chemical kinetics or thermodynamics)
Toxicity is simplified into 3 levels
No user interface yet (terminal-based only)
CSV file is manually created within the script
🚀 Future Improvements (Version 2+)
Planned upgrades include:

Expanding reaction database (SN1, E2, etc.)
Adding scoring systems for solvent selection instead of simple filtering
Introducing weighted chemical properties (not just polarity + type)
Improving data structure design
Possibly adding a user interface (e.g., Streamlit)
Making the system more chemically realistic using external references
