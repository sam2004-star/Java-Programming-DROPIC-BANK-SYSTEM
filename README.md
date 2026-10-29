# Java-Programming-DROPIC-BANK-SYSTEM

import java.security.MessageDigest;
import java.security.SecureRandom;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.format.DateTimeParseException;
import java.util.ArrayList;
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.Scanner;
import java.util.Locale;

public class Main {

    private static final Scanner sc = new Scanner(System.in);
    private static final SecureRandom random = new SecureRandom();

    private static final Map<String, Member> members = new LinkedHashMap<>();
    private static final Map<String, Double> exchangeRates = new LinkedHashMap<>();

    private static final String[] MOBILE_SERVICES = {
        "M-Pesa", "Airtel Money", "T-Kash"
    };

    static class Member {
        String name;
        String dob;
        String idNumber;
        String phone;
        String accountNumber;
        String pinHash;

        double balance = 0.0;
        double loanOwed = 0.0;

        ArrayList<String> history = new ArrayList<>();

        Member(String name, String dob, String idNumber,
               String phone, String accountNumber, String pinHash) {

            this.name = name;
            this.dob = dob;
            this.idNumber = idNumber;
            this.phone = phone;
            this.accountNumber = accountNumber;
            this.pinHash = pinHash;

            history.add(timestamp()
                + " | Account opened | Opening balance KES 0.00");
        }
    }

    public static void main(String[] args) {

        setupExchangeRates();

        System.out.println("==================================================");
        System.out.println("              WELCOME TO DROPIC BANK");
        System.out.println("       Secure Fund Management, Made Simple");
        System.out.println("==================================================");

        boolean running = true;

        while (running) {

            System.out.println("\nMAIN MENU");
            System.out.println("1. New Member Registration");
            System.out.println("2. Existing Member Login");
            System.out.println("3. Exit");

            String choice = prompt("Choose an option: ");

            switch (choice) {
                case "1":
                    registerMember();
                    break;

                case "2":
                    login();
                    break;

                case "3":
                    running = false;
                    System.out.println(
                        "Thank you for choosing Dropic Bank. Goodbye!"
                    );
                    break;

                default:
                    System.out.println("Invalid choice. Enter 1, 2, or 3.");
            }
        }

        sc.close();
    }

    private static void setupExchangeRates() {

        // Illustrative rates only. These are not live market rates.
        exchangeRates.put("KES", 1.0);
        exchangeRates.put("USD", 0.0077);
        exchangeRates.put("EUR", 0.0071);
        exchangeRates.put("GBP", 0.0060);
        exchangeRates.put("UGX", 28.0);
        exchangeRates.put("TZS", 19.5);
        exchangeRates.put("RWF", 10.0);
        exchangeRates.put("NGN", 11.5);
    }

    private static void registerMember() {

        System.out.println("\n--- NEW MEMBER REGISTRATION ---");

        String name;

        while (true) {
            name = prompt("Enter full name: ")
                .trim().replaceAll("\\s+", " ");

            if (name.matches("[\\p{L}][\\p{L} .'-]{1,79}")) {
                break;
            }

            System.out.println(
                "Enter a valid full name with at least 2 characters."
            );
        }

        String dob;

        while (true) {
            dob = prompt("Enter date of birth (YYYY-MM-DD): ").trim();

            try {
                LocalDate date = LocalDate.parse(dob);

                if (date.isBefore(LocalDate.now())
                    && !date.isBefore(LocalDate.now().minusYears(120))) {
                    break;
                }

                System.out.println(
                    "Date of birth must be in the past and within 120 years."
                );

            } catch (DateTimeParseException e) {
                System.out.println(
                    "Invalid date. Example: 2001-05-23"
                );
            }
        }

        String idNumber;

        while (true) {
            idNumber = prompt("Enter 9-digit ID number: ").trim();

            if (idNumber.matches("\\d{9}")) {
                break;
            }

            System.out.println("ID number must contain exactly 9 digits.");
        }

        for (Member existing : members.values()) {
            if (existing.idNumber.equals(idNumber)) {
                System.out.println(
                    "This ID number is already registered."
                );
                return;
            }
        }

        String phone;

        while (true) {
            phone = prompt(
                "Enter Kenyan phone number (10 digits, starting 07 or 01): "
            ).trim();

            if (validKenyanPhone(phone)) {
                break;
            }

            System.out.println(
                "Use a 10-digit number beginning with 07 or 01."
            );
        }

        for (Member existing : members.values()) {
            if (existing.phone.equals(phone)) {
                System.out.println(
                    "This phone number is already registered."
                );
                return;
            }
        }

        String pin = createPin();
        String account = generateUniqueAccountNumber();

        Member member = new Member(
            name, dob, idNumber, phone, account, hashPin(pin)
        );

        members.put(account, member);

        System.out.println("\nREGISTRATION SUCCESSFUL");
        System.out.println("----------------------------------------");
        System.out.println("Name: " + member.name);
        System.out.println("Account number: " + member.accountNumber);
        System.out.println("Registered phone: " + member.phone);
        System.out.println("Opening balance: " + money(member.balance));
        System.out.println("----------------------------------------");
        System.out.println("Keep your account number and PIN private.");
        System.out.println(
            "Demo accounts exist only while the program is running."
        );
    }

    private static String createPin() {

        while (true) {

            System.out.println("\nCreate a 4-digit PIN.");
            System.out.println("1. Show PIN after entry");
            System.out.println("2. Hide PIN as ****");

            String visibility = prompt("Choose 1 or 2: ").trim();

            if (!visibility.equals("1") && !visibility.equals("2")) {
                System.out.println("Choose 1 or 2.");
                continue;
            }

            String pin = prompt("Enter new 4-digit PIN: ").trim();

            if (!pin.matches("\\d{4}")) {
                System.out.println("PIN must contain exactly 4 digits.");
                continue;
            }

            if (visibility.equals("1")) {
                System.out.println("PIN entered: " + pin);
            } else {
                System.out.println("PIN entered: ****");
            }

            String confirm = prompt("Confirm 4-digit PIN: ").trim();

            if (!pin.equals(confirm)) {
                System.out.println("PINs do not match. Try again.");
                continue;
            }

            return pin;
        }
    }

    private static void login() {

        System.out.println("\n--- EXISTING MEMBER LOGIN ---");

        if (members.isEmpty()) {
            System.out.println("No accounts exist. Register first.");
            return;
        }

        String account = prompt(
            "Enter your 10-digit account number: "
        ).trim();

        Member member = members.get(account);

        if (member == null) {
            System.out.println("Account not found.");
            return;
        }

        for (int attempts = 3; attempts > 0; attempts--) {

            String pin = prompt(
                "Enter your 4-digit PIN ("
                + attempts + " attempt(s) left): "
            ).trim();

            if (hashPin(pin).equals(member.pinHash)) {

                System.out.println(
                    "\nLogin successful. Welcome, " + member.name + "!"
                );

                dashboard(member);
                return;
            }

            System.out.println("Incorrect PIN.");
        }

        System.out.println("Too many failed attempts. Please try again later.");
    }

    private static void dashboard(Member member) {

        boolean loggedIn = true;

        while (loggedIn) {

            System.out.println("\n==================================================");
            System.out.println("              DROPIC BANK DASHBOARD");
            System.out.println("Customer: " + member.name);
            System.out.println("Account: " + member.accountNumber);
            System.out.println("Balance: " + money(member.balance));
            System.out.println("==================================================");

            System.out.println("1. View Account Balance");
            System.out.println("2. Deposit from Mobile Money");
            System.out.println("3. Withdraw to Mobile Money");
            System.out.println("4. Transfer to Another Dropic Account");
            System.out.println("5. International Transfer (Demo)");
            System.out.println("6. Exchange Rate Calculator");
            System.out.println("7. Loans and Repayment");
            System.out.println("8. Transaction History");
            System.out.println("9. TransUnion / Credit Information");
            System.out.println("10. My Account Details");
            System.out.println("11. Logout");

            String choice = prompt("Choose an option: ");

            switch (choice) {

                case "1":
                    System.out.println(
                        "Available balance: " + money(member.balance)
                    );
                    break;

                case "2":
                    depositFromMobileMoney();
                    break;

                case "3":
                    withdrawToMobileMoney();
                    break;

                case "4":
                    transferBetweenAccounts(member);
                    break;

                case "5":
                    internationalTransfer(member);
                    break;

                case "6":
                    exchangeCalculator();
                    break;

                case "7":
                    loanMenu(member);
                    break;

                case "8":
                    showHistory(member);
                    break;

                case "9":
                    creditInformation();
                    break;

                case "10":
                    showAccountDetails(member);
                    break;

                case "11":
                    loggedIn = false;
                    System.out.println("You have been logged out safely.");
                    break;

                default:
                    System.out.println("Choose a number from 1 to 11.");
            }
        }
    }

    private static void depositFromMobileMoney() {

        System.out.println("\n--- DEPOSIT FROM MOBILE MONEY ---");

        Member target = selectAccount(
            "Choose the Dropic Bank account to CREDIT:"
        );

        if (target == null) {
            return;
        }

        int serviceIndex = chooseMobileService(
            "Choose the mobile-money service sending the deposit:"
        );

        if (serviceIndex < 0) {
            return;
        }

        String senderPhone = prompt(
            "Enter sending phone number (10 digits, 07 or 01): "
        ).trim();

        if (!validKenyanPhone(senderPhone)) {
            System.out.println("Invalid phone number. Deposit cancelled.");
            return;
        }

        double amount = readAmount("Enter deposit amount in KES: ");

        System.out.println("\nCONFIRM DEPOSIT (SIMULATION)");
        System.out.println(
            "From: " + MOBILE_SERVICES[serviceIndex]
            + " (" + senderPhone + ")"
        );
        System.out.println(
            "To account: " + target.accountNumber + " - " + target.name
        );
        System.out.println("Amount: " + money(amount));

        if (!confirmAction()) {
            System.out.println("Deposit cancelled.");
            return;
        }

        target.balance += amount;

        target.history.add(
            timestamp() + " | Mobile-money deposit from "
            + MOBILE_SERVICES[serviceIndex] + " " + senderPhone
            + " | +" + money(amount)
            + " | Balance " + money(target.balance)
        );

        System.out.println("Demo deposit successful.");
        System.out.println("New balance: " + money(target.balance));
        System.out.println("No real mobile-money payment was processed.");
    }

    private static void withdrawToMobileMoney() {

        System.out.println("\n--- WITHDRAW TO MOBILE MONEY ---");

        Member source = selectAccount(
            "Choose the Dropic Bank account to DEBIT:"
        );

        if (source == null) {
            return;
        }

        if (source.balance <= 0) {
            System.out.println("This account has insufficient funds.");
            return;
        }

        int serviceIndex = chooseMobileService(
            "Choose the mobile-money service receiving the withdrawal:"
        );

        if (serviceIndex < 0) {
            return;
        }

        String recipientPhone = prompt(
            "Enter receiving phone number (10 digits, 07 or 01): "
        ).trim();

        if (!validKenyanPhone(recipientPhone)) {
            System.out.println("Invalid phone number. Withdrawal cancelled.");
            return;
        }

        double amount = readAmount("Enter withdrawal amount in KES: ");

        if (amount > source.balance) {
            System.out.println(
                "Insufficient funds. Available balance: "
                + money(source.balance)
            );
            return;
        }

        System.out.println("\nCONFIRM WITHDRAWAL (SIMULATION)");
        System.out.println(
            "From account: " + source.accountNumber + " - " + source.name
        );
        System.out.println(
            "To: " + MOBILE_SERVICES[serviceIndex]
            + " (" + recipientPhone + ")"
        );
        System.out.println("Amount: " + money(amount));

        if (!confirmAction()) {
            System.out.println("Withdrawal cancelled.");
            return;
        }

        source.balance -= amount;

        source.history.add(
            timestamp() + " | Withdrawal to "
            + MOBILE_SERVICES[serviceIndex] + " " + recipientPhone
            + " | -" + money(amount)
            + " | Balance " + money(source.balance)
        );

        System.out.println("Demo withdrawal successful.");
        System.out.println("New balance: " + money(source.balance));
        System.out.println("No real mobile-money payment was sent.");
    }

    private static void transferBetweenAccounts(Member sender) {

        System.out.println("\n--- TRANSFER TO ANOTHER DROPIC ACCOUNT ---");

        if (sender.balance <= 0) {
            System.out.println("Insufficient funds.");
            return;
        }

        String recipientAccount = prompt(
            "Enter recipient's 10-digit account number: "
        ).trim();

        Member recipient = members.get(recipientAccount);

        if (recipient == null) {
            System.out.println("Recipient account not found.");
            return;
        }

        if (recipient.accountNumber.equals(sender.accountNumber)) {
            System.out.println("You cannot transfer to the same account.");
            return;
        }

        double amount = readAmount("Enter transfer amount in KES: ");

        if (amount > sender.balance) {
            System.out.println(
                "Insufficient funds. Available balance: "
                + money(sender.balance)
            );
            return;
        }

        System.out.println("\nCONFIRM TRANSFER");
        System.out.println(
            "From: " + sender.accountNumber + " (" + sender.name + ")"
        );
        System.out.println(
            "To: " + recipient.accountNumber + " (" + recipient.name + ")"
        );
        System.out.println("Amount: " + money(amount));

        if (!confirmAction()) {
            System.out.println("Transfer cancelled.");
            return;
        }

        sender.balance -= amount;
        recipient.balance += amount;

        sender.history.add(
            timestamp() + " | Transfer to " + recipient.accountNumber
            + " | -" + money(amount)
            + " | Balance " + money(sender.balance)
        );

        recipient.history.add(
            timestamp() + " | Transfer from " + sender.accountNumber
            + " | +" + money(amount)
            + " | Balance " + money(recipient.balance)
        );

        System.out.println("Demo transfer completed.");
        System.out.println(
            "Your new balance: " + money(sender.balance)
        );
    }

    private static void internationalTransfer(Member member) {

        System.out.println("\n--- INTERNATIONAL TRANSFER (DEMO ONLY) ---");
        System.out.println("1. Wise");
        System.out.println("2. Western Union");
        System.out.println("3. Remitly");
        System.out.println("4. Deel");

        String choice = prompt("Choose provider: ").trim();

        String provider;

        switch (choice) {
            case "1":
                provider = "Wise";
                break;
            case "2":
                provider = "Western Union";
                break;
            case "3":
                provider = "Remitly";
                break;
            case "4":
                provider = "Deel";
                break;
            default:
                System.out.println("Invalid provider.");
                return;
        }

        System.out.println("Provider selected: " + provider);
        System.out.println(
            "This prototype does not connect to " + provider
            + " or send international payments."
        );
        System.out.println(
            "Use the exchange calculator for an illustrative conversion."
        );
    }

    private static void exchangeCalculator() {

        System.out.println("\n--- EXCHANGE RATE CALCULATOR ---");
        System.out.println("Illustrative demo rates only; NOT live rates.");
        System.out.println(
            "Supported currencies: "
            + String.join(", ", exchangeRates.keySet())
        );

        String from = prompt("Convert FROM currency: ")
            .trim().toUpperCase();

        String to = prompt("Convert TO currency: ")
            .trim().toUpperCase();

        if (!exchangeRates.containsKey(from)
            || !exchangeRates.containsKey(to)) {
            System.out.println(
                "Unsupported currency. Choose from the listed currencies."
            );
            return;
        }

        double amount = readAmount("Enter amount: ");

        double result =
            amount / exchangeRates.get(from) * exchangeRates.get(to);

        System.out.printf(
            Locale.US,
            "%.2f %s = %.2f %s%n",
            amount, from, result, to
        );
    }

    private static void loanMenu(Member member) {

        boolean back = false;

        while (!back) {

            System.out.println("\n--- LOANS AND REPAYMENT (DEMO) ---");
            System.out.println(
                "Outstanding demo loan: " + money(member.loanOwed)
            );
            System.out.println("1. Request a demo loan");
            System.out.println("2. Repay demo loan");
            System.out.println("3. Back");

            String choice = prompt("Choose an option: ").trim();

            switch (choice) {

                case "1": {
                    double loan = readAmount(
                        "Enter requested loan amount in KES: "
                    );

                    if (loan > 50000) {
                        System.out.println("Demo loan limit is KES 50,000.");
                        break;
                    }

                    System.out.println(
                        "This is a simulation, not a real loan offer."
                    );

                    if (confirmAction()) {
                        member.loanOwed += loan;
                        member.balance += loan;

                        member.history.add(
                            timestamp() + " | Demo loan credited | +"
                            + money(loan)
                            + " | Balance " + money(member.balance)
                        );

                        System.out.println(
                            "Demo loan added. Outstanding loan: "
                            + money(member.loanOwed)
                        );
                    } else {
                        System.out.println("Loan request cancelled.");
                    }
                    break;
                }

                case "2": {
                    if (member.loanOwed <= 0) {
                        System.out.println("No demo loan is outstanding.");
                        break;
                    }

                    double repayment = readAmount(
                        "Enter repayment amount in KES: "
                    );

                    if (repayment > member.balance) {
                        System.out.println("Insufficient account balance.");
                        break;
                    }

                    if (repayment > member.loanOwed) {
                        System.out.println(
                            "Repayment cannot exceed the outstanding loan of "
                            + money(member.loanOwed)
                        );
                        break;
                    }

                    if (confirmAction()) {
                        member.balance -= repayment;
                        member.loanOwed -= repayment;

                        member.history.add(
                            timestamp() + " | Demo loan repayment | -"
                            + money(repayment)
                            + " | Balance " + money(member.balance)
                        );

                        System.out.println(
                            "Repayment recorded. Loan remaining: "
                            + money(member.loanOwed)
                        );
                    } else {
                        System.out.println("Repayment cancelled.");
                    }
                    break;
                }

                case "3":
                    back = true;
                    break;

                default:
                    System.out.println("Choose 1, 2, or 3.");
            }
        }
    }

    private static void showHistory(Member member) {

        System.out.println("\n--- TRANSACTION HISTORY ---");

        if (member.history.isEmpty()) {
            System.out.println("No transactions recorded.");
            return;
        }

        for (String entry : member.history) {
            System.out.println(entry);
        }
    }

    private static void creditInformation() {

        System.out.println("\n--- CREDIT INFORMATION ---");

        System.out.println(
            "Dropic Bank does not retrieve or store real credit reports."
        );

        System.out.println(
            "For an official credit report or dispute, contact "
            + "TransUnion Kenya through its official channels."
        );

        System.out.println(
            "Never share your PIN, password, or one-time verification code."
        );
    }

    private static void showAccountDetails(Member member) {

        System.out.println("\n--- ACCOUNT DETAILS ---");
        System.out.println("Name: " + member.name);
        System.out.println("Date of birth: " + member.dob);
        System.out.println("ID number: " + member.idNumber);
        System.out.println("Registered phone: " + member.phone);
        System.out.println("Account number: " + member.accountNumber);
        System.out.println("Available balance: " + money(member.balance));
        System.out.println(
            "Outstanding demo loan: " + money(member.loanOwed)
        );
    }

    private static Member selectAccount(String heading) {

        if (members.isEmpty()) {
            System.out.println("No accounts are registered.");
            return null;
        }

        System.out.println("\n" + heading);
        System.out.println("Registered Dropic Bank accounts:");

        for (Member member : members.values()) {
            System.out.println(
                member.accountNumber + " | " + member.name
                + " | Balance: " + money(member.balance)
            );
        }

        String account = prompt("Enter the account number: ").trim();

        Member selected = members.get(account);

        if (selected == null) {
            System.out.println("Account not found. No transaction was made.");
            return null;
        }

        return selected;
    }

    private static int chooseMobileService(String heading) {

        System.out.println("\n" + heading);

        for (int i = 0; i < MOBILE_SERVICES.length; i++) {
            System.out.println((i + 1) + ". " + MOBILE_SERVICES[i]);
        }

        String choice = prompt("Select service (1-3): ").trim();

        if (!choice.matches("[123]")) {
            System.out.println("Invalid service. Transaction cancelled.");
            return -1;
        }

        return Integer.parseInt(choice) - 1;
    }

    private static boolean confirmAction() {

        while (true) {

            String choice = prompt(
                "Confirm? Enter Y to continue or N to cancel: "
            ).trim();

            if (choice.equalsIgnoreCase("Y")) {
                return true;
            }

            if (choice.equalsIgnoreCase("N")) {
                return false;
            }

            System.out.println("Please enter Y or N.");
        }
    }

    private static double readAmount(String message) {

        while (true) {

            String input = prompt(message).trim().replace(",", "");

            try {
                double amount = Double.parseDouble(input);

                if (Double.isFinite(amount) && amount > 0) {
                    return amount;
                }

            } catch (NumberFormatException ignored) {
                // Invalid input is handled below.
            }

            System.out.println(
                "Enter a valid positive amount, e.g. 1500 or 1500.50."
            );
        }
    }

    private static boolean validKenyanPhone(String phone) {
        return phone.matches("0[17]\\d{8}");
    }

    private static String generateUniqueAccountNumber() {

        String account;

        do {
            long number = 1_000_000_000L
                + random.nextLong(9_000_000_000L);

            account = String.valueOf(number);

        } while (members.containsKey(account));

        return account;
    }

    private static String hashPin(String pin) {

        try {
            MessageDigest digest = MessageDigest.getInstance("SHA-256");

            byte[] bytes = digest.digest(
                pin.getBytes(java.nio.charset.StandardCharsets.UTF_8)
            );

            StringBuilder result = new StringBuilder();

            for (byte b : bytes) {
                result.append(String.format("%02x", b));
            }

            return result.toString();

        } catch (Exception e) {
            throw new IllegalStateException(
                "PIN hashing is unavailable.", e
            );
        }
    }

    private static String prompt(String message) {
        System.out.print(message);
        return sc.nextLine();
    }

    private static String money(double amount) {
        return String.format(Locale.US, "KES %,.2f", amount);
    }

    private static String timestamp() {
        return LocalDateTime.now()
            .withNano(0)
            .toString()
            .replace('T', ' ');
    }
}
