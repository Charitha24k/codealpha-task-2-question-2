import java.io.*;
import java.util.*;

// Stock class
class Stock {
    String symbol;
    double price;

    Stock(String symbol, double price) {
        this.symbol = symbol;
        this.price = price;
    }
}

// Portfolio class with file I/O
class Portfolio {
    Map<String, Integer> holdings = new HashMap<>();
    double balance;
    private static final String FILE_NAME = "portfolio.txt";

    Portfolio(double initialBalance) {
        this.balance = initialBalance;
        loadPortfolioFromFile(); // Load data if exists
    }

    void buyStock(Stock stock, int quantity) {
        double totalCost = stock.price * quantity;
        if (balance >= totalCost) {
            holdings.put(stock.symbol, holdings.getOrDefault(stock.symbol, 0) + quantity);
            balance -= totalCost;
            System.out.println("Bought " + quantity + " shares of " + stock.symbol);
        } else {
            System.out.println("Insufficient balance to buy.");
        }
    }

    void sellStock(Stock stock, int quantity) {
        int owned = holdings.getOrDefault(stock.symbol, 0);
        if (owned >= quantity) {
            holdings.put(stock.symbol, owned - quantity);
            balance += stock.price * quantity;
            System.out.println("Sold " + quantity + " shares of " + stock.symbol);
        } else {
            System.out.println("You don't have enough shares to sell.");
        }
    }

    void viewPortfolio(Map<String, Stock> stockMarket) {
        System.out.println("\n--- Portfolio ---");
        System.out.println("Balance: ₹" + balance);
        for (String symbol : holdings.keySet()) {
            int qty = holdings.get(symbol);
            double price = stockMarket.get(symbol).price;
            System.out.println(symbol + ": " + qty + " shares (₹" + price + " each)");
        }
        System.out.println("-----------------\n");
    }

    void savePortfolioToFile() {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(FILE_NAME))) {
            writer.write("BALANCE=" + balance + "\n");
            for (Map.Entry<String, Integer> entry : holdings.entrySet()) {
                writer.write(entry.getKey() + "=" + entry.getValue() + "\n");
            }
            System.out.println("Portfolio saved to file.");
        } catch (IOException e) {
            System.out.println("Error saving portfolio: " + e.getMessage());
        }
    }

    void loadPortfolioFromFile() {
        File file = new File(FILE_NAME);
        if (!file.exists()) return;

        try (BufferedReader reader = new BufferedReader(new FileReader(FILE_NAME))) {
            String line;
            while ((line = reader.readLine()) != null) {
                if (line.startsWith("BALANCE=")) {
                    balance = Double.parseDouble(line.split("=")[1]);
                } else {
                    String[] parts = line.split("=");
                    if (parts.length == 2) {
                        holdings.put(parts[0], Integer.parseInt(parts[1]));
                    }
                }
            }
            System.out.println("Portfolio loaded from file.");
        } catch (IOException e) {
            System.out.println("Error loading portfolio: " + e.getMessage());
        }
    }
}

// Main class
public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        Map<String, Stock> stockMarket = new HashMap<>();
        stockMarket.put("TCS", new Stock("TCS", 3500));
        stockMarket.put("INFY", new Stock("INFY", 1600));
        stockMarket.put("RELIANCE", new Stock("RELIANCE", 2500));

        Portfolio portfolio = new Portfolio(10000); // Load or start fresh

        while (true) {
            System.out.println("=== Stock Trading Menu ===");
            System.out.println("1. View Market");
            System.out.println("2. Buy Stock");
            System.out.println("3. Sell Stock");
            System.out.println("4. View Portfolio");
            System.out.println("5. Exit");
            System.out.print("Choose option: ");
            int choice = sc.nextInt();

            switch (choice) {
                case 1:
                    System.out.println("\n--- Market Data ---");
                    for (Stock stock : stockMarket.values()) {
                        System.out.println(stock.symbol + ": ₹" + stock.price);
                    }
                    System.out.println("-------------------\n");
                    break;

                case 2:
                    System.out.print("Enter stock symbol to buy: ");
                    String buySymbol = sc.next().toUpperCase();
                    System.out.print("Enter quantity: ");
                    int buyQty = sc.nextInt();
                    if (stockMarket.containsKey(buySymbol)) {
                        portfolio.buyStock(stockMarket.get(buySymbol), buyQty);
                    } else {
                        System.out.println("Stock not found.");
                    }
                    break;

                case 3:
                    System.out.print("Enter stock symbol to sell: ");
                    String sellSymbol = sc.next().toUpperCase();
                    System.out.print("Enter quantity: ");
                    int sellQty = sc.nextInt();
                    if (stockMarket.containsKey(sellSymbol)) {
                        portfolio.sellStock(stockMarket.get(sellSymbol), sellQty);
                    } else {
                        System.out.println("Stock not found.");
                    }
                    break;

                case 4:
                    portfolio.viewPortfolio(stockMarket);
                    break;

                case 5:
                    portfolio.savePortfolioToFile();
                    System.out.println("Exiting trading system. Goodbye!");
                    sc.close();
                    return;

                default:
                    System.out.println("Invalid choice.");
            }
        }
    }
}
