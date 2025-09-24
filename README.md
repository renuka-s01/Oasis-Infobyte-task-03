# Oasis-Infobyte-task-03
 Oasis Infobyte Internship task-3 ->ATM INTERFACE

#ATM.java
 import java.util.Scanner;

public class ATM {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        Bank bank = new Bank();
        bank.loadDummyUsers(); 

        System.out.println("Welcome to the ATM System!");

        System.out.print("Enter User ID: ");
        String userId = scanner.nextLine();

        System.out.print("Enter PIN: ");
        String pin = scanner.nextLine();

        User currentUser = bank.authenticateUser(userId, pin);

        if (currentUser != null) {
            System.out.println("\nLogin successful! Welcome, " + currentUser.getUserId());
            ATMOperations operations = new ATMOperations(currentUser, bank);
            operations.showMenu();
        } else {
            System.out.println("Invalid User ID or PIN.");
        }

        scanner.close();
    }
}



#User.java
import java.util.ArrayList;
import java.util.List;

public class User {
    private String userId;
    private String pin;
    private double balance;
    private List<Transaction> transactionHistory;

    public User(String userId, String pin, double balance) {
        this.userId = userId;
        this.pin = pin;
        this.balance = balance;
        this.transactionHistory = new ArrayList<>();
    }

    public String getUserId() {
        return userId;
    }

    public boolean validatePin(String inputPin) {
        return this.pin.equals(inputPin);
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        balance += amount;
        addTransaction("Deposit", amount);
    }

    public boolean withdraw(double amount) {
        if (amount <= balance) {
            balance -= amount;
            addTransaction("Withdraw", amount);
            return true;
        }
        return false;
    }

    public boolean transfer(User recipient, double amount) {
        if (amount <= balance) {
            balance -= amount;
            recipient.deposit(amount);
            addTransaction("Transfer to " + recipient.getUserId(), amount);
            return true;
        }
        return false;
    }

    public void addTransaction(String type, double amount) {
        transactionHistory.add(new Transaction(type, amount));
    }

    public void showTransactionHistory() {
        if (transactionHistory.isEmpty()) {
            System.out.println("No transactions yet.");
        } else {
            System.out.println("Transaction History:");
            for (Transaction t : transactionHistory) {
                System.out.println(t);
            }
        }
    }
}


#Transaction.java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class Transaction {
    private String type;
    private double amount;
    private String timestamp;

    public Transaction(String type, double amount) {
        this.type = type;
        this.amount = amount;
        this.timestamp = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
    }

    @Override
    public String toString() {
        return timestamp + " - " + type + ": $" + amount;
    }
}




#Bank.java
import java.util.HashMap;

public class Bank {
    private HashMap<String, User> users;

    public Bank() {
        users = new HashMap<>();
    }

    public void loadDummyUsers() {
        users.put("user1", new User("user1", "1234", 1000.0));
        users.put("user2", new User("user2", "5678", 500.0));
    }

    public User authenticateUser(String userId, String pin) {
        User user = users.get(userId);
        if (user != null && user.validatePin(pin)) {
            return user;
        }
        return null;
    }

    public User getUser(String userId) {
        return users.get(userId);
    }
}



#ATMOperations.java
import java.util.Scanner;

public class ATMOperations {
    private User user;
    private Bank bank;
    private Scanner scanner;

    public ATMOperations(User user, Bank bank) {
        this.user = user;
        this.bank = bank;
        this.scanner = new Scanner(System.in);
    }

    public void showMenu() {
        int choice;
        do {
            System.out.println("\n===== ATM MENU =====");
            System.out.println("1. Transaction History");
            System.out.println("2. Withdraw");
            System.out.println("3. Deposit");
            System.out.println("4. Transfer");
            System.out.println("5. Quit");
            System.out.print("Choose an option: ");

            while (!scanner.hasNextInt()) {
                System.out.println("Invalid input. Please enter a number.");
                scanner.next();
            }

            choice = scanner.nextInt();

            switch (choice) {
                case 1:
                    user.showTransactionHistory();
                    break;
                case 2:
                    handleWithdraw();
                    break;
                case 3:
                    handleDeposit();
                    break;
                case 4:
                    handleTransfer();
                    break;
                case 5:
                    System.out.println("Thank you for using the ATM. Goodbye!");
                    break;
                default:
                    System.out.println("Invalid choice. Please try again.");
            }
        } while (choice != 5);
    }

    private void handleWithdraw() {
        System.out.print("Enter amount to withdraw: ");
        double amount = scanner.nextDouble();
        if (user.withdraw(amount)) {
            System.out.println("Withdraw successful. Remaining balance: $" + user.getBalance());
        } else {
            System.out.println("Insufficient balance.");
        }
    }

    private void handleDeposit() {
        System.out.print("Enter amount to deposit: ");
        double amount = scanner.nextDouble();
        user.deposit(amount);
        System.out.println("Deposit successful. Current balance: $" + user.getBalance());
    }

    private void handleTransfer() {
        System.out.print("Enter recipient User ID: ");
        scanner.nextLine(); 
        String recipientId = scanner.nextLine();
        User recipient = bank.getUser(recipientId);
        if (recipient == null) {
            System.out.println("User not found.");
            return;
        }

        System.out.print("Enter amount to transfer: ");
        double amount = scanner.nextDouble();
        if (user.transfer(recipient, amount)) {
            System.out.println("Transfer successful. Remaining balance: $" + user.getBalance());
        } else {
            System.out.println("Insufficient balance or error in transfer.");
        }
    }
}


