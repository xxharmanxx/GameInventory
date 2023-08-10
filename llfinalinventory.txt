#include <iostream>
#include <string>

struct Item {
    std::string name;
    int quantity;

    Item(const std::string& itemName, int itemQuantity) : name(itemName), quantity(itemQuantity) {}
};

struct Node {
    Item item;
    Node* next;

    Node(const Item& newItem) : item(newItem), next(nullptr) {}
};

class GameInventory {
private:
    Node* head;
    int size;

public:
    GameInventory() : head(nullptr), size(0) {}

    ~GameInventory() {
        clearInventory();
    }

    void addToInventory(const Item& item) {
        Node* newNode = new Node(item);

        if (head == nullptr) {
            head = newNode;
        } else {
            Node* current = head;
            while (current->next != nullptr) {
                current = current->next;
            }
            current->next = newNode;
        }

        size++;
    }

    void removeFromInventory() {
        if (head == nullptr) {
            std::cout << "Inventory is empty." << std::endl;
            return;
        }

        Node* current = head;
        Node* previous = nullptr;

        while (current->next != nullptr) {
            previous = current;
            current = current->next;
        }

        std::cout << "Removing item: " << current->item.name << std::endl;

        delete current;
        current = nullptr;

        if (previous != nullptr) {
            previous->next = nullptr;
        } else {
            head = nullptr;
        }

        size--;
    }

    void displayInventory() {
        if (head == nullptr) {
            std::cout << "Inventory is empty." << std::endl;
            return;
        }

        Node* current = head;
        int count = 0;
        int row = 1;

        while (current != nullptr) {
            std::cout << current->item.name << " x" << current->item.quantity << "\t";

            count++;
            if (count == 4) {
                std::cout << std::endl;
                count = 0;
                row++;
            }

            current = current->next;
        }

        std::cout << std::endl;
    }

    void clearInventory() {
        Node* current = head;

        while (current != nullptr) {
            Node* next = current->next;
            delete current;
            current = next;
        }

        head = nullptr;
        size = 0;
    }

    int getSize() const {
        return size;
    }

    Node* getHead() const {
        return head;
    }
};

class Game {
private:
    GameInventory inventory;
    GameInventory combineGrid;

public:
    void start() {
        std::cout << "Welcome to the game!" << std::endl;
        std::cout << "You can 'walk', 'search', or 'exit'." << std::endl;

        while (true) {
            std::string action;
            std::cout << "> ";
            std::cin >> action;

            if (action == "walk") {
                // Move to the next location
                std::cout << "Walking to the next location..." << std::endl;
            } else if (action == "search") {
                searchLocation();
            } else if (action == "exit") {
                std::cout << "Exiting the game. Goodbye!" << std::endl;
                break;
            } else {
                std::cout << "Invalid action. Try again." << std::endl;
            }
        }
    }

    void searchLocation() {
        std::cout << "Searching the location..." << std::endl;

        // Randomly determine if an item is found or not
        bool itemFound = (rand() % 2) == 0;

        if (itemFound) {
            std::cout << "You found an item!" << std::endl;
            std::cout << "What do you want to do with the item?" << std::endl;
            std::cout << "1. Pick item" << std::endl;
            std::cout << "2. Discard item" << std::endl;
            std::cout << "3. Combine item" << std::endl;
            std::cout << "4. Ignore item" << std::endl;
            std::cout << "5. Display inventory" << std::endl;

            int choice;
            std::cout << "> ";
            std::cin >> choice;

            switch (choice) {
                case 1:
                    pickItem();
                    break;
                case 2:
                    discardItem();
                    break;
                case 3:
                    combineItem();
                    break;
                case 4:
                    std::cout << "You ignore the item and continue searching." << std::endl;
                    break;
                case 5:
                    displayInventory();
                    break;
                default:
                    std::cout << "Invalid choice. Ignoring the item." << std::endl;
                    break;
            }
        } else {
            std::cout << "No item found. Keep searching!" << std::endl;
        }
    }

    void pickItem() {
        if (inventory.getSize() >= 36) {
            std::cout << "Inventory is full. Discard an item to pick a new one." << std::endl;
        } else {
            std::string itemName;
            int itemQuantity;

            std::cout << "Enter the name of the item: ";
            std::cin >> itemName;
            std::cout << "Enter the quantity of the item: ";
            std::cin >> itemQuantity;

            inventory.addToInventory(Item(itemName, itemQuantity));

            std::cout << "Item added to the inventory." << std::endl;
        }
    }

    void discardItem() {
        if (inventory.getSize() == 0) {
            std::cout << "Inventory is empty. Nothing to discard." << std::endl;
        } else {
            inventory.removeFromInventory();
        }
    }

    void combineItem() {
        if (inventory.getSize() < 2) {
            std::cout << "Insufficient items in the inventory to combine." << std::endl;
            return;
        }

        std::string item1, item2;
        std::cout << "Enter the names of the two items to combine: ";
        std::cin >> item1 >> item2;

        // Check if the items are present in the inventory
        Node* current = inventory.getHead();
        Item* item1Ptr = nullptr;
        Item* item2Ptr = nullptr;

        while (current != nullptr) {
            if (current->item.name == item1) {
                item1Ptr = &(current->item);
            } else if (current->item.name == item2) {
                item2Ptr = &(current->item);
            }

            current = current->next;
        }

        if (item1Ptr == nullptr || item2Ptr == nullptr) {
            std::cout << "One or both items not found in the inventory." << std::endl;
            return;
        }

        // Combine the items and add to the combine grid
        std::string combinedItem = getCombinedItem(item1, item2);
        if (combinedItem.empty()) {
            std::cout << "Items cannot be combined." << std::endl;
            return;
        }

        combineGrid.addToInventory(Item(combinedItem, 1));

        // Remove the original items from the inventory
        inventory.removeFromInventory();
        inventory.removeFromInventory();

        std::cout << "Items combined successfully. New item added to the combine grid." << std::endl;
    }

    std::string getCombinedItem(const std::string& item1, const std::string& item2) {
        if (item1 == "elder sword" && item2 == "enchanted blade") {
            return "reheated sword";
        } else if (item1 == "simple sword" && item2 == "capstone") {
            return "reheated sword";
        } else {
            return "";
        }
    }

    void displayInventory() {
        std::cout << "Inventory:" << std::endl;
        inventory.displayInventory();

        std::cout << "Combine Grid:" << std::endl;
        combineGrid.displayInventory();
    }
};

int main() {
    srand(time(NULL)); // Seed the random number generator

    Game game;
    game.start();

    return 0;
}
