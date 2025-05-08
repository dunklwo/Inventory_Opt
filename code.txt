#include <bits/stdc++.h>
using namespace std;
const int WARE_COUNT = 3;
const int SHOP_COUNT = 5;
const string DEFAULT_PASSWORD = "Warehouse123";
enum MenuOption { ADD_STOCK = 1, ROUTE_OPT, SEARCH_PRODUCT, BUDGET_SUGGESTION, VIEW_STOCK, ADVANCED_FEATURES, EXIT_APP };

struct Item {
    string name;
    int quantity;
    int expiry;
    int price;


    int warehouseSource = 0;

};

vector<Item> loadItems(const string& filename) {

    ifstream fin(filename);
    vector<Item> items;
    string line;
    while (getline(fin, line)) {
        if (line.empty()) continue;
        stringstream ss(line);
        Item it;
        string q, e, p;
        getline(ss, it.name, ',');
        getline(ss, q, ',');
        getline(ss, e, ',');
        getline(ss, p, ',');
        it.quantity = stoi(q);
        it.expiry   = stoi(e);
        it.price    = stoi(p);
        items.push_back(it);
    }

    return items;
}

void saveItems(const string& filename, const vector<Item>& items) {
    ofstream fout(filename, ios::app);
    for (auto& it : items) {
        fout << it.name << "," << it.quantity << "," << it.expiry << "," << it.price << "\n";
    }
}

void merge(vector<Item>& arr, int l, int m, int r) {
    int n1 = m - l + 1;
    int n2 = r - m;
    vector<Item> L(arr.begin() + l, arr.begin() + m + 1);
    vector<Item> R(arr.begin() + m + 1, arr.begin() + r + 1);
    int i = 0, j = 0, k = l;
    while (i < n1 && j < n2) {
        if (L[i].expiry > R[j].expiry) arr[k++] = L[i++];
        else                            arr[k++] = R[j++];
    }
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}

void mergeSort(vector<Item>& arr, int l, int r) {
    if (l < r) {
        int m = l + (r - l) / 2;
        mergeSort(arr, l, m);
        mergeSort(arr, m + 1, r);
        merge(arr, l, m, r);
    }
}

vector<Item> fractionalKnapsack(vector<Item> items, int budget) {
    sort(items.begin(), items.end(), [](const Item& a, const Item& b) {
        double r1 = (double)a.price / a.quantity;
        double r2 = (double)b.price / b.quantity;
        return r1 < r2;
    });
    vector<Item> chosen;
    for (auto& it : items) {
        if (it.quantity == 0) continue;
        int unitPrice = it.price / it.quantity;
        if (unitPrice <= 0) continue;
        int maxQty = min(it.quantity, budget / unitPrice);
        if (maxQty > 0) {
            chosen.push_back({it.name, maxQty, it.expiry, maxQty * unitPrice});
            budget -= maxQty * unitPrice;
        }
        if (budget <= 0) break;
    }
    return chosen;
}

const int INF = 1e9;
void floydWarshall(int n, vector<vector<int>>& dist) {
    for (int k = 0; k < n; ++k) {
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                if (dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
}

vector<int> buildLPS(const string& pat) {
    int m = pat.size();
    vector<int> lps(m, 0);
    int len = 0;
    int i = 1;
    while (i < m) {
        if (pat[i] == pat[len]) {
            len++;
            lps[i] = len;
            i++;
        } else {
            if (len != 0) {
                len = lps[len - 1];
            } else {
                lps[i] = 0;
                i++;
            }
        }
    }
    return lps;
}

bool kmpSearch(const string& text, const string& pattern) {
    auto lps = buildLPS(pattern);
    int i = 0, j = 0;
    int n = text.size(), m = pattern.size();
    while (i < n) {
        if (text[i] == pattern[j]) {
            i++; j++;
            if (j == m) {
                return true;
            }
        } else {
            if (j != 0) j = lps[j - 1];
            else        i++;
        }
    }
    return false;
}

void displayWarehouseDetails(int idx, const vector<string>& files) {
    string header = "----- Warehouse " + to_string(idx + 1) + " Stock Details -----";
    cout << "\n" << header << "\n";
    auto items = loadItems(files[idx]);
    if (items.empty()) {
        cout << "[No items found in this warehouse]" << "\n";
    }
    for (auto& it : items) {
        cout << "Name: " << setw(20) << left << it.name
             << " | Qty: " << setw(4) << right << it.quantity
             << " | Expiry: " << setw(4) << it.expiry
             << " | Price: " << setw(6) << it.price << "\n";
    }
    cout << string(header.size(), '-') << "\n";
}

void trackSales() {
    cout << "[Tracking sales feature coming soon...]" << "\n";
}

void alertLowStock(const vector<string>& files) {
    cout << "------- Low Stock Alerts -------\n";
    bool alertShown = false;
    for (int i = 0; i < files.size(); ++i) {
        auto items = loadItems(files[i]);
        for (auto& it : items) {
            if (it.quantity <= 5) {
                if (!alertShown) alertShown = true;
                cout << "[ALERT] Warehouse " << (i+1)
                     << " | Item: " << it.name
                     << " | Qty: " << it.quantity << "\n";
            }
        }
    }
    if (!alertShown) {
        cout << "All stock levels are sufficient.\n";
    }
    cout << "--------------------------------\n";
}


void restockSuppliers(const vector<string>& files) {
    cout << "------- Restock Existing Items -------\n";
    for (int i = 0; i < files.size(); ++i) {
        auto items = loadItems(files[i]);
        bool updated = false;
        cout << "Warehouse " << (i + 1) << " items:\n";
        for (int j = 0; j < items.size(); ++j) {
            cout << j+1 << ". " << items[j].name << " (Current Qty: " << items[j].quantity << ")\n";
        }
        cout << "Enter item number to update quantity (0 to skip): ";
        int choice; cin >> choice;
        if (choice > 0 && choice <= items.size()) {
            cout << "Enter new quantity for " << items[choice - 1].name << ": ";
            int newQty; cin >> newQty;
            items[choice - 1].quantity = newQty;
            updated = true;
        }
        if (updated) {
            saveItems(files[i], items);
            cout << "Stock updated for Warehouse " << (i + 1) << "\n";
        }
    }
    cout << "--------------------------------------\n";
}



void manageUserRoles() {
    cout << "[User role management coming soon...]" << "\n";
}
void expiryManagement(const vector<string>& files, int daysThreshold) {
    cout << "------- Items Expiring in " << daysThreshold << " Days -------\n";
    bool anyFound = false;
    for (int i = 0; i < files.size(); ++i) {
        auto items = loadItems(files[i]);
        mergeSort(items, 0, items.size() - 1);
        for (auto& it : items) {
            if (it.expiry <= daysThreshold) {
                if (!anyFound) anyFound = true;
                cout << "Warehouse " << (i+1)
                     << " | Item: " << it.name
                     << " | Expiry: " << it.expiry
                     << " days\n";
            }
        }
    }
    if (!anyFound) {
        cout << "No items are expiring within " << daysThreshold << " days.\n";
    }
    cout << "-----------------------------------------------------------\n";
}
void orderFromNearestWarehouse(const vector<string>& files, vector<vector<int>>& dist, int shopIdx) {
    cout << "------- Place Order from Nearest Warehouse -------\n";
    cout << "Enter item name to order: ";
    string itemName; cin >> ws; getline(cin, itemName);

    int nearestWarehouse = -1;
    int minDist = INF;
    int itemIndex = -1;
    for (int i = 0; i < files.size(); ++i) {
        auto items = loadItems(files[i]);
        for (int j = 0; j < items.size(); ++j) {
            if (kmpSearch(items[j].name, itemName) && items[j].quantity > 0 && dist[i][shopIdx] < minDist) {
                minDist = dist[i][shopIdx];
                nearestWarehouse = i;
                itemIndex = j;
            }
        }
    }

    if (nearestWarehouse == -1) {
        cout << "No stock available in any warehouse for this item.\n";
    } else {
        auto items = loadItems(files[nearestWarehouse]);
        int availableQty = items[itemIndex].quantity;
        cout << "Item found in Warehouse " << (nearestWarehouse+1) << " at distance " << minDist << "\n";
        cout << "Available quantity: " << availableQty << "\n";
        cout << "Enter quantity to order: ";
        int orderQty; cin >> orderQty;

        if (orderQty > availableQty) {
            cout << "Only " << availableQty << " units available. Reducing order to available quantity.\n";
            orderQty = availableQty;
        }
        items[itemIndex].quantity -= orderQty;
        saveItems(files[nearestWarehouse], items);
        cout << "Order placed successfully! " << orderQty << " units deducted from Warehouse " << (nearestWarehouse+1) << ".\n";
    }

    cout << "--------------------------------------------------\n";
}

void orderItems(const vector<string>& files) {
    cout << "------- Order Out-of-Stock Items -------\n";
    for (int i = 0; i < files.size(); ++i) {
        auto items = loadItems(files[i]);
        bool updated = false;
        for (auto& it : items) {
            if (it.quantity == 0) {
                cout << "Warehouse " << (i+1) << " | Item: " << it.name << " is out of stock.\n";
                cout << "Enter quantity to order: ";
                int qty; cin >> qty;
                it.quantity = qty;
                updated = true;
            }
        }
        if (updated) {
            saveItems(files[i], items);
            cout << "Order placed and stock updated for Warehouse " << (i+1) << "\n";
        }
    }
    cout << "----------------------------------------\n";
}
void visualizeData() {
    vector<string> warehouseFiles = {"warehouse1.txt", "warehouse2.txt", "warehouse3.txt"};

    cout << "\n===============================================\n";
    cout << "          WAREHOUSE INVENTORY VISUALIZATION     \n";
    cout << "===============================================\n\n";
    vector<vector<Item>> allWarehouses;
    int maxItems = 0;
    int maxQuantity = 0;

    for (const auto& file : warehouseFiles) {
        vector<Item> items = loadItems(file);
        allWarehouses.push_back(items);
        maxItems = max(maxItems, (int)items.size());
        for (const auto& item : items) {
            maxQuantity = max(maxQuantity, item.quantity);
        }
    }
    cout << "\n----- Detailed Inventory Report -----\n\n";

    cout << setw(20) << left << "Product Name" << " | ";
    for (int w = 0; w < warehouseFiles.size(); w++) {
        cout << "WH" << (w+1) << setw(9) << " ";
    }
    cout << "| Total" << endl;

    cout << string(20, '-') << "-+-";
    for (int w = 0; w < warehouseFiles.size(); w++) {
        cout << string(11, '-');
    }
    cout << "-+-------" << endl;
    map<string, vector<int>> itemMap;

    for (int w = 0; w < allWarehouses.size(); w++) {
        for (const auto& item : allWarehouses[w]) {
            if (itemMap.find(item.name) == itemMap.end()) {
                itemMap[item.name] = vector<int>(allWarehouses.size(), 0);
            }
            itemMap[item.name][w] = item.quantity;
        }
    }

    for (const auto& pair : itemMap) {
        cout << setw(20) << left << pair.first << " | ";
        int total = 0;
        for (int qty : pair.second) {
            cout << setw(10) << right << qty << " ";
            total += qty;
        }
        cout << "| " << setw(5) << right << total << endl;
    }
    cout << "\n\n----- Inventory Level Visualization -----\n\n";

    const int maxBarWidth = 50;

    for (int w = 0; w < allWarehouses.size(); w++) {
        cout << "Warehouse " << (w+1) << " Stock Levels:\n";
        cout << string(60, '-') << endl;

        for (const auto& item : allWarehouses[w]) {
            string expiryWarning = "";
            if (item.expiry <= 5) {
                expiryWarning = " [!]";
            } else if (item.expiry <= 15) {
                expiryWarning = " [*]";
            }
            float unitPrice = item.quantity > 0 ?
                             static_cast<float>(item.price) / item.quantity : 0;

            cout << setw(20) << left << item.name<< " " << item.quantity<< expiryWarning<< " ($" << fixed << setprecision(2) << unitPrice << "/unit)"<< endl;
        }
        cout << string(60, '-') << endl << endl;
    }

    cout << "\n----- Expiry Summary -----\n\n";

    vector<tuple<string, int, int>> expiryList;

    for (int w = 0; w < allWarehouses.size(); w++) {
        for (const auto& item : allWarehouses[w]) {
            if (item.quantity > 0) {
                expiryList.push_back(make_tuple(item.name, w, item.expiry));
            }
        }
    }

    sort(expiryList.begin(), expiryList.end(),
        [](const auto& a, const auto& b) { return get<2>(a) < get<2>(b); });

    if (!expiryList.empty()) {
        cout << "Items nearing expiry (sorted):\n";
        for (int i = 0; i < min(10, (int)expiryList.size()); i++) {
            cout << setw(20) << left << get<0>(expiryList[i])
                 << " | Warehouse " << (get<1>(expiryList[i]) + 1)
                 << " | " << get<2>(expiryList[i]) << " days remaining" << endl;
        }
    } else {
        cout << "No items with expiry information found.\n";
    }

    cout << "\n===============================================\n";
    cout << "Press Enter to return to the main menu...";
    cin.ignore();
    string dummy;
    getline(cin, dummy);
}
struct FiniteAutomaton {
    int m;
    int states;
    vector<vector<int>> transition;
    FiniteAutomaton(const string& pattern) {
        m = pattern.size();
        states = m + 1;
        set<char> alphabet;
        for (char c : pattern) alphabet.insert(c);
        transition.resize(states, vector<int>(256, 0));

        for (int i = 0; i < 256; i++) {
            transition[0][i] = 0;
        }

        for (int state = 0; state <= m; state++) {
            for (char c : alphabet) {
                string prefix = pattern.substr(0, state) + c;
                int nextState = state + 1;

                while (nextState > 0) {
                    if (pattern.substr(0, nextState) == prefix.substr(prefix.size() - nextState)) {
                        break;
                    }
                    nextState--;
                }

                transition[state][c] = nextState;
            }
        }
    }

    bool search(const string& text) {
        int state = 0;
        for (int i = 0; i < text.size(); i++) {
            state = transition[state][text[i]];
            if (state == m) {
                return true;
            }
        }
        return false;
    }
};
void removeItems(const vector<string>& files) {
    cout << "------- Remove Items from Warehouses -------\n";
    cout << "Enter item name pattern to remove: ";
    string pattern;
    cin >> ws;
    getline(cin, pattern);
    FiniteAutomaton fa(pattern);
    bool itemsRemoved = false;

    for (int i = 0; i < files.size(); ++i) {
        auto items = loadItems(files[i]);
        vector<Item> updatedItems;
        bool warehouseModified = false;

        cout << "Warehouse " << (i+1) << " items:\n";
        for (int j = 0; j < items.size(); ++j) {
            if (fa.search(items[j].name)) {
                cout << "Found matching item: " << items[j].name << "\n";
                cout << "Do you want to remove this item? (1=Yes, 0=No): ";
                int removeChoice;
                cin >> removeChoice;

                if (removeChoice == 1) {
                    cout << "Item '" << items[j].name << "' will be removed.\n";
                    warehouseModified = true;
                    itemsRemoved = true;
                } else {
                    updatedItems.push_back(items[j]);
                }
            } else {
                updatedItems.push_back(items[j]);
            }
        }

        if (warehouseModified) {
            ofstream fout(files[i], ios::trunc);
            fout.close();

            if (!updatedItems.empty()) {
                saveItems(files[i], updatedItems);
            }
            cout << "Warehouse " << (i+1) << " updated after item removal.\n";
        } else {
            cout << "No items removed from Warehouse " << (i+1) << ".\n";
        }
    }

    if (!itemsRemoved) {
        cout << "No items matching '" << pattern << "' were found in any warehouse.\n";
    }
    cout << "-------------------------------------------\n";
}
vector<Item> enhancedBudgetSuggestion(const vector<string>& warehouseFiles, int budget,
                                     bool prioritizeExpiry = false, int currentWarehouse = -1) {
    vector<Item> allItems;
    vector<int> itemSources;
    for (int w = 0; w < warehouseFiles.size(); ++w) {
        if (currentWarehouse != -1 && w != currentWarehouse) continue;

        auto items = loadItems(warehouseFiles[w]);
        for (const auto& item : items) {
            if (item.quantity > 0) {
                allItems.push_back(item);
                itemSources.push_back(w);
            }
        }
    }

    if (prioritizeExpiry) {
        sort(allItems.begin(), allItems.end(),
            [&itemSources](const Item& a, const Item& b) {
                if (a.expiry == b.expiry) {
                    double valueA = static_cast<double>(a.price) / a.quantity;
                    double valueB = static_cast<double>(b.price) / b.quantity;
                    return valueA > valueB;
                }
                return a.expiry < b.expiry;
            });
    } else {
        sort(allItems.begin(), allItems.end(),
            [](const Item& a, const Item& b) {
                double valueA = static_cast<double>(a.price) / a.quantity;
                double valueB = static_cast<double>(b.price) / b.quantity;
                return valueA > valueB;
            });
    }
    vector<Item> chosen;
    vector<int> chosenSources;
    int remainingBudget = budget;
    int totalValue = 0;

    for (size_t i = 0; i < allItems.size(); ++i) {
        const auto& item = allItems[i];
        int unitPrice = item.price / item.quantity;

        if (unitPrice <= 0) continue;
        int maxQty = min(item.quantity, remainingBudget / unitPrice);

        if (maxQty > 0) {
            Item selectedItem = {
                item.name,
                maxQty,
                item.expiry,
                maxQty * unitPrice
            };
            chosen.push_back(selectedItem);
            chosenSources.push_back(itemSources[i]);
            remainingBudget -= maxQty * unitPrice;
            totalValue += maxQty * unitPrice;
        }

        if (remainingBudget <= 0) break;
    }
    for (size_t i = 0; i < chosen.size(); ++i) {
        chosen[i].warehouseSource = chosenSources[i] + 1;
    }

    return chosen;
}

void displayBudgetSuggestion(const vector<Item>& chosen, int originalBudget) {
    cout << "\n===== Budget Suggestion Results =====\n";

    if (chosen.empty()) {
        cout << "No items fit within the budget of $" << originalBudget << ".\n";
        return;
    }

    int totalCost = 0;
    int totalItems = 0;
    cout << setw(25) << left << "Item Name"
         << setw(10) << right << "Quantity"
         << setw(12) << right << "Unit Price"
         << setw(12) << right << "Total"
         << setw(10) << right << "Expiry"
         << setw(12) << right << "Warehouse" << endl;
    cout << string(81, '-') << endl;
    for (const auto& item : chosen) {
        float unitPrice = static_cast<float>(item.price) / item.quantity;

        cout << setw(25) << left << item.name
             << setw(10) << right << item.quantity
             << setw(12) << right << fixed << setprecision(2) << unitPrice
             << setw(12) << right << item.price
             << setw(10) << right << item.expiry << " days";

        if (item.warehouseSource > 0) {
            cout << setw(12) << right << item.warehouseSource;
        } else {
            cout << setw(12) << right << "N/A";
        }
        cout << endl;

        totalCost += item.price;
        totalItems += item.quantity;
    }

    cout << string(81, '-') << endl;
    cout << "Total Items: " << totalItems << " | Total Cost: $" << totalCost
         << " | Remaining Budget: $" << (originalBudget - totalCost) << endl;
    double utilizationPercentage = (static_cast<double>(totalCost) / originalBudget) * 100;
    cout << "Budget Utilization: " << fixed << setprecision(1) << utilizationPercentage << "%" << endl;
    for (const auto& item : chosen) {
        if (item.expiry <= 7) {
            cout << "\n[WARNING] Some selected items expire within 7 days!\n";
            break;
        }
    }

    cout << "====================================\n";
}

void enhancedBudgetSuggestionMenu(const vector<string>& warehouseFiles, int currentWarehouse) {
    cout << "\n===== Enhanced Budget Suggestion =====\n";

    int budget;
    cout << "Enter your budget ($): ";
    cin >> budget;
    cout << "\nAdvanced Options:\n";
    cout << "1. Standard (maximize value)\n";
    cout << "2. Prioritize soon-to-expire items\n";
    cout << "3. Select specific warehouse\n";
    cout << "4. All warehouses\n";
    cout << "Select strategy (1-4): ";

    int strategy;
    cin >> strategy;

    bool prioritizeExpiry = (strategy == 2);
    int warehouseToUse = -1;
    if (strategy == 3) {
        cout << "\nAvailable warehouses:\n";
        for (int i = 0; i < warehouseFiles.size(); i++) {
            cout << (i + 1) << ". Warehouse " << (i + 1);
            if (i == currentWarehouse) {
                cout << " (current)";
            }
            cout << endl;
        }

        cout << "Select warehouse (1-" << warehouseFiles.size() << "): ";
        int selectedWarehouse;
        cin >> selectedWarehouse;

        if (selectedWarehouse >= 1 && selectedWarehouse <= warehouseFiles.size()) {
            warehouseToUse = selectedWarehouse - 1;  // Convert to 0-based index
            cout << "Using Warehouse " << selectedWarehouse << " for budget suggestions.\n";
        } else {
            cout << "Invalid selection. Using all warehouses.\n";
        }
    }

    vector<Item> suggestions = enhancedBudgetSuggestion(
        warehouseFiles, budget, prioritizeExpiry, warehouseToUse);
    displayBudgetSuggestion(suggestions, budget);

    cout << "=======================================\n";
}

int main() {
    vector<string> warehouseFiles = {"warehouse1.txt", "warehouse2.txt", "warehouse3.txt"};
    vector<vector<int>> dist = {
        {0,10,20,30,40,20,60,70}, {10,0,15,35,45,55,65,75},
        {20,15,0,25,35,15,55,5}, {30,35,25,0,12,22,32,42},
        {40,45,35,12,0,18,28,38}, {50,55,45,22,18,0,14,24},
        {60,65,55,32,28,14,0,16}, {70,75,65,42,38,24,16,0}
    };
    floydWarshall(WARE_COUNT + SHOP_COUNT, dist);

    int currentWarehouse = 0;
    int currentShop = WARE_COUNT;
    bool running = true;

    while (running) {
        cout << "\n========================================" << "\n";
        cout << " Warehouse Inventory Management Menu " << "\n";
        cout << "========================================" << "\n";
        cout << "1. Add Stock to Warehouse" << "\n";
        cout << "2. Route Optimization to Shop" << "\n";
        cout << "3. Search Product in Warehouses" << "\n";
        cout << "4. Budget-based Suggestion" << "\n";
        cout << "5. View Warehouse Stock (Password)" << "\n";
        cout << "6. Advanced Features Menu" << "\n";
        cout << "7. Exit Application" << "\n";
        cout << "Enter choice (1-7): ";
        int choice;
        cin >> choice;
        cout << "----------------------------------------" << "\n";

        switch (choice) {
            case ADD_STOCK: {
                cout << "[ADD_STOCK] Enter warehouse number (1-3): ";
                int widx; cin >> widx;
                widx--;
                cout << "[ADD_STOCK] Number of new items to add: ";
                int n; cin >> n;
                vector<Item> items(n);
                for (int i = 0; i < n; ++i) {
                    cout << " Item " << (i+1) << " name: "; cin >> ws; getline(cin, items[i].name);
                    cout << " Item " << (i+1) << " quantity: "; cin >> items[i].quantity;
                    cout << " Item " << (i+1) << " expiry days: "; cin >> items[i].expiry;
                    cout << " Item " << (i+1) << " total price: "; cin >> items[i].price;
                }
                mergeSort(items, 0, items.size() - 1);
                auto selected = fractionalKnapsack(items,100);
                saveItems(warehouseFiles[widx], selected);
                cout << "[ADD_STOCK] Items successfully added to warehouse.\n";
                break;
            }
            case ROUTE_OPT: {
                cout << "[ROUTE_OPT] Enter shop number (1-5): ";
                int shop; cin >> shop;
                currentShop = WARE_COUNT + (shop - 1);
                int bestIdx = 0, bestDist = INF;
                for (int w = 0; w < WARE_COUNT; ++w) {
                    if (dist[w][currentShop] < bestDist) {
                        bestDist = dist[w][currentShop];
                        bestIdx = w;
                    }
                }
                currentWarehouse = bestIdx;
                cout << "[ROUTE_OPT] Nearest warehouse: " << (bestIdx+1)
                     << " (Distance=" << bestDist << ")\n";
                break;
            }
            case SEARCH_PRODUCT: {

                cout << "[SEARCH_PRODUCT] Enter keyword to search: ";
                string keyword; cin >> keyword;
                bool found = false;
                auto items = loadItems(warehouseFiles[currentWarehouse]);
                cout << "Searching in current warehouse (#" << (currentWarehouse+1) << ")...\n";
                for (auto& it : items) {
                    if (kmpSearch(it.name, keyword)) {
                        cout << " Found: " << it.name
                             << " | Qty:" << it.quantity
                             << " | Exp:" << it.expiry
                             << " | Price:" << it.price << "\n";
                        found = true;
                    }
                }
                if (!found) {

                    cout << "Not in current. Checking all warehouses...\n";
                    for (int w = 0; w < WARE_COUNT; ++w) {
                        if (w == currentWarehouse) continue;
                        auto items2 = loadItems(warehouseFiles[w]);
                        for (auto& it : items2) {
                            if (kmpSearch(it.name, keyword)) {
                                cout << " Found in W" << (w+1)
                                     << " at Dist=" << dist[w][currentShop] << "\n";
                                found = true;
                            }
                        }
                    }
                }
                if (!found) {
                    cout << "Item not found in any warehouse.\n";
                }
                break;
            }
          case BUDGET_SUGGESTION: {

    cout << "[BUDGET_SUGGESTION] Would you like to use:\n";
    cout << "1. Basic suggestion (current warehouse only)\n";
    cout << "2. Advanced suggestion with more options\n";
    cout << "Enter choice (1-2): ";

    int suggestionType;
    cin >> suggestionType;

    if (suggestionType == 1) {
        cout << "[BUDGET_SUGGESTION] Enter budget for shop: ";
        int B; cin >> B;
        auto items = loadItems(warehouseFiles[currentWarehouse]);
        auto chosen = fractionalKnapsack(items, B);
        if (chosen.empty()) {
            cout << "No items fit within budget.\n";
        } else {
            cout << "Suggested items within budget:\n";
            for (auto& it : chosen) {
                cout << "  " << it.name
                     << " | Qty:" << it.quantity
                     << " | Total:" << it.price << "\n";
            }
        }
    } else {
        enhancedBudgetSuggestionMenu(warehouseFiles, currentWarehouse);
    }
    break;
}
            case VIEW_STOCK: {
                cout << "[VIEW_STOCK] Enter password (" << 3 << " attempts): ";
                string inPwd;
                bool authenticated = false;
                for (int attempt = 1; attempt <= 3; ++attempt) {
                    cin >> inPwd;
                    if (kmpSearch(inPwd, DEFAULT_PASSWORD)) {
                        authenticated = true;
                        break;
                    }
                    cout << "Incorrect. Attempts left: " << (3- attempt) << "\n";
                    if (attempt < 3)
                        cout << "Re-enter password: ";
                }
                if (authenticated) {
                    int w; cout << "Select warehouse to view (1-3): "; cin >> w;
                    displayWarehouseDetails(w - 1, warehouseFiles);
                } else {
                    cout << "Access denied. Returning to main menu.\n";
                }
                break;
            }
            case ADVANCED_FEATURES: {
    cout << "[ADVANCED_FEATURES]   1.Low-Stock Alerts  2.Supplier Restock  "
         << "3.Data Visual   4.Expiry alerts  5.Order Items  6.Remove Items  "
         << "7.Enhanced Budget Planning\n";
    cout << "Select feature: ";
    int sub; cin >> sub;
    if (sub == 1) alertLowStock(warehouseFiles);
    else if (sub == 2) restockSuppliers(warehouseFiles);
    else if (sub == 3) visualizeData();
    else if (sub == 4) {
        int days;
        cout << "Enter expiry threshold (in days): ";
        cin >> days;
        expiryManagement(warehouseFiles, days);
    }
    else if (sub == 5) {
        orderItems(warehouseFiles);
    }
    else if (sub == 6) {
        removeItems(warehouseFiles);
    }
    else if (sub == 7) {
        enhancedBudgetSuggestionMenu(warehouseFiles, currentWarehouse);
    }
    else cout << "Returning to main menu...\n";
    break;
                 case EXIT_APP: {
                cout << "Exiting application. Goodbye!\n";
                running = false;
                break;
            }
            case 8:
                orderFromNearestWarehouse(warehouseFiles, dist, currentShop);
                break;
            default: {
                cout << "Invalid choice. Please select 1-7.\n";
                break;
            }
        }
    }
    }
    return 0;
}

