# Algorithm Driven Smart Supply Chain and Logistics Optimizer
Problem Statement:  
Retail businesses managing multiple warehouses and shops often face challenges such as 
inaccurate stock tracking, inefficient allocation of new inventory, delays in fulfilling shop 
demands, lack of real-time visibility, and difficulty suggesting purchases within a shop’s 
budget. Manual tracking systems are prone to errors, leading to frequent stockouts, excess 
inventory, missed sales opportunities, and higher operational costs. Moreover, inefficient 
search and poor route planning between warehouses and shops further worsen operational 
delays and customer dissatisfaction. 
Objectives: 
1. Develop an automated system to centralize inventory data from three warehouses 
and five shops. 
2. Provide real‑time visibility of stock levels, expiry dates, and pricing information. 
3. Implement route optimization for fulfilling shop orders from the nearest warehouse. 
4. Offer budget-based purchase suggestions to shops using fractional knapsack 
methods. 
5. Enhance decision‑making through low‑stock alerts, expiry management, data 
visualization, and efficient search. 
Project Features  
● Add Stock: 
Sort incoming inventory using Merge Sort and distribute it optimally across 
warehouses with Fractional Knapsack. 
● Route Optimization: 
Apply the Floyd–Warshall algorithm to find shortest paths between warehouses and 
shops for faster delivery. 
● Search Product: 
Use KMP string matching for quick and efficient product searches across multiple 
locations. 
● Budget Suggestion: 
Recommend products within a given budget using Fractional Knapsack to maximize 
value. 
● Password-Protected View: 
Protect sensitive inventory data and reports with password authentication. 
● Advanced Features: 
● Sales tracking module (planned) 
● Low-stock alert notifications 
● Supplier restocking suggestions 
● Data visualization via ASCII charts and tables 
● Expiry alerts for nearing-expiry products 
● Out-of-stock item ordering support 
● Pattern-based inventory cleanup using finite automata 
Algorithm Implementation Analysis 
● Floyd-Warshall (Graph Algorithms) 
➢ Implementation: The code uses Floyd-Warshall algorithm (floydWarshall 
function) to compute shortest paths between all pairs of nodes (warehouses and 
shops). 
➢ Use Case: Optimal path selection for deliveries and finding shortest warehouse 
routes to shops. 
➢ Location: floydWarshall() function called during initialization to precompute all 
distances. 
● Knapsack (Dynamic Programming) 
➢ Implementation: The fractional knapsack algorithm is implemented in 
fractionalKnapsack and enhanced in enhancedBudgetSuggestion. 
➢ Use Case: Load optimization for delivery trucks to maximize deliveries within 
weight limits and budget constraints. 
➢ Location: Used in both stock addition and budget suggestion features. 
● Greedy Algorithm 
➢ Implementation: The fractional knapsack solution uses a greedy approach to select 
items with best value/quantity ratio. 
➢ Use Case: Last-mile delivery optimization by selecting the best items to purchase 
within a budget. 
➢ Location: fractionalKnapsack function and budget suggestion features. 
● KMP (String Matching) 
➢ Implementation: The Knuth-Morris-Pratt algorithm is implemented in kmpSearch and 
buildLPS functions. 
➢ Use Case: Fast search for inventory items during product searches and password 
verification. 
➢ Location: Used in product search functionality and password checking. 
● Merge Sort (Divide & Conquer) 
➢ Implementation: The mergeSort function implements the standard merge sort 
algorithm. 
➢ Use Case: Sorting inventory based on expiration dates for better stock management. 
➢ Location: Used when adding new stock and checking expiring items. 
● Finite Automaton (String Matching) 
➢ Implementation: The FiniteAutomaton class implements finite automata based string 
matching. 
➢ Use Case: Efficient pattern matching for item removal functionality. 
➢ Location: Used in the removeItems function. 
● Additional Notable Implementations 
➢ Dynamic Programming: Used in various forms throughout the system, particularly in 
the budget optimization and route planning. 
➢ Matrix Operations: Implicitly used in the Floyd-Warshall implementation for distance 
matrix manipulation.
