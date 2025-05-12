using System;
using System.Collections.Generic;
using System.Linq;
using System.Security.Cryptography;

//#############################################
// User Management Subsystem
//#############################################
class User
{
    private string Name;
    private string Password;
    private string Email;
    private bool Is_Logged_in;
    private BankAccount BAcount;
    private AssetColletion ass_coll = new AssetColletion();

    public User() { }
    public void Addass() { ass_coll.addAsset(); }
    public void SetISlog(bool isLog) { this.Is_Logged_in = isLog; }
    public void CreateAccount(BankAccount BA) { this.BAcount = BA; }
    public void SetName()
    {
        Console.Write("Please enter your name: ");
        Name = Console.ReadLine();
    }
    public void SetEmail()
    {
        Console.Write("Please enter your Email: ");
        Email = Console.ReadLine();
    }
    public void SetPassword()
    {
        Console.Write("Please enter your Password: ");
        Password = Console.ReadLine();
    }
    public BankAccount GetAcount() { return this.BAcount; }
    public string GetName() { return Name; }
    public string GetEmail() { return Email; }
    public string GetPassword() { return Password; }
    public AssetColletion GetassColl() { return ass_coll; }
    public bool is_Logged_in() { return Is_Logged_in; }
};


//#############################################
// Authentication Service Subsystem
//#############################################
class Auth_Service
{
    public static void Sign_Up(User user, List<User> users)
    {
        user.SetName();
        user.SetEmail();
        user.SetPassword();
        if (CheckUsers.FoundCheck(user, users))
        {
            Console.WriteLine("This User already Exist...");
            return;
        }
        user.SetISlog(true);
        users.Add(user);
    }
    public static User Log_in(User user, List<User> users)
    {
        Console.WriteLine("Please Enter Your Email");
        string inputEmail = Console.ReadLine();
        Console.WriteLine("Please Enter Your Password");
        string inputPassword = Console.ReadLine();
        foreach (User u in users)
        {
            if (u.GetEmail() == inputEmail && u.GetPassword() == inputPassword)
            {
                user.SetISlog(true);
                Console.WriteLine($"Welcome back, {u.GetName()}!");
                return u;
            }
        }
        Console.WriteLine("Not found in our database.");
        user.SetISlog(false);
        return user;
    }
};


//#############################################
// User Check Subsystem
//#############################################
class CheckUsers
{
    public static bool FoundCheck(User user, List<User> users)
    {
        if (user.is_Logged_in()) return true;
        foreach (User user1 in users)
        {
            if (user.GetEmail() == user1.GetEmail()) { return true; }
        }
        return false;
    }
};

class UpdateProfile
{
    private User user;

    public UpdateProfile(User u)
    {
        user = u;
    }

    public void Update_Profile()
    {
        Console.WriteLine("Update Menu:");
        Console.WriteLine("1. Change Name");
        Console.WriteLine("2. Change Email");
        Console.WriteLine("3. Change Password");
        Console.Write("Choose an option: ");
        string choice = Console.ReadLine();

        switch (choice)
        {
            case "1":
                Console.Write("Enter new name: ");
                user.SetName();
                break;

            case "2":
                Console.Write("Enter new email: ");
                user.SetEmail();
                break;

            case "3":
                Console.Write("Enter new password: ");
                user.SetPassword();
                break;

            default:
                Console.WriteLine("Invalid option.");
                return;
        }

        Console.WriteLine("Profile updated.");
    }
}

class UserGoals
{
    private string goalName;
    private string description;
    private double targetAmount;
    private double currentAmount;
    private IGoalObserver observer;

    public UserGoals() { currentAmount = 0; }

    public void AttachObserver(IGoalObserver obs)
    {
        observer = obs;
    }

    private void NotifyObserver()
    {
        if (observer != null)
        {
            observer.OnGoalUpdated(this);
        }
    }

    public void SetGoalName(string name) => goalName = name;
    public void SetDescription(string desc) => description = desc;
    public void SetTargetAmount(double amount) => targetAmount = amount;
    public string GetGoalName() => goalName;
    public string GetDescription() => description;
    public double GetTargetAmount() => targetAmount;
    public double GetCurrentAmount() => currentAmount;

    public void AddProgress(double amount)
    {
        currentAmount += amount;
        NotifyObserver();
    }

    public bool IsAchieved() => currentAmount >= targetAmount;
    public double GetProgressPercentage() => targetAmount == 0 ? 0 : Math.Min(100.0, (currentAmount / targetAmount) * 100.0);
}

class GoalInputHandler
{
    public static void CreateGoalFromInput(UserGoals Goal)
    {
        Console.Write("Enter Goal Name: ");
        Goal.SetGoalName(Console.ReadLine());

        Console.Write("Enter Description: ");
        Goal.SetDescription(Console.ReadLine());

        Console.Write("Enter Target Amount: ");
        double amount;
        while (!double.TryParse(Console.ReadLine(), out  amount))
        {
            Console.Write("Invalid input. Enter numeric Target Amount: ");
        }
        Goal.SetTargetAmount(amount);
    }

    public static void AddProgressToGoal(UserGoals goal)
    {
        Console.Write("Enter amount to add: ");
        double amount;
        while (!double.TryParse(Console.ReadLine(), out amount))
        {
            Console.Write("Invalid input. Enter numeric amount: ");
        }
        goal.AddProgress(amount);
        Console.WriteLine("Progress added!");
    }
}

class GoalViewer
{
    public static void DisplayGoal(UserGoals goal)
    {
        if (goal == null)
        {
            Console.WriteLine("No goal set on this asset.");
            return;
        }

        Console.WriteLine("\nGoal Details:");
        Console.WriteLine($"Name        : {goal.GetGoalName()}");
        Console.WriteLine($"Description : {goal.GetDescription()}");
        Console.WriteLine($"Target      : {goal.GetTargetAmount():C}");
        Console.WriteLine($"Saved       : {goal.GetCurrentAmount():C} ({goal.GetProgressPercentage():F2}%)");
    }
}

interface IGoalObserver
{
    void OnGoalUpdated(UserGoals goal);
}

class Notify : IGoalObserver
{
    private User user;

    public Notify(User u)
    {
        user = u;
    }

    public void OnGoalUpdated(UserGoals goal)
    {
        Console.WriteLine($"\nHey {user.GetName()}, your goal '{goal.GetGoalName()}' was updated.");
        Console.WriteLine($"Current progress: {goal.GetProgressPercentage():F2}%");

        if (goal.IsAchieved())
        {
            Console.WriteLine("Congratulations! You achieved your goal!");
        }
    }
}

class AssetColletion
{
    private double zakatAmout;
    List<Asset> assets = new List<Asset>();
    public List<Asset> GetAssets() { return assets; }
    public void addAsset()
    {
        Console.WriteLine("Add Asset: ");
        Console.Write("Enter Asset Name: ");
        string theName = Console.ReadLine();
        Console.Write("Enter Purchase Name/Date: ");
        string pName = Console.ReadLine();
        Console.Write("Enter Value of asset: ");
        double val, rsk;
        while (!double.TryParse(Console.ReadLine(), out  val))
        {
            Console.Write("Invalid input. Enter numeric Value of asset: ");
        }
        Console.Write("Enter Risk Value: ");
        while (!double.TryParse(Console.ReadLine(), out  rsk))
        {
            Console.Write("Invalid input. Enter numeric Risk Value: ");
        }
        Asset a1 = new Asset(theName, pName, val, rsk);
        assets.Add(a1);
        Console.WriteLine("Added Asset successfully");
    }

    public void editAsset()
    {
        if (assets.Count == 0)
        {
            Console.WriteLine("No assets to edit.");
            return;
        }

        Console.WriteLine("Enter the index of the Asset you want to edit: ");
        this.printAsset();
        if (!int.TryParse(Console.ReadLine(), out int n) || n < 0 || n >= assets.Count)
        {
            Console.WriteLine("Invalid index.");
            return;
        }

        Asset selectedAsset = assets[n];
        Console.WriteLine("What do you want to edit in the asset (name, purchase date, value, or risk):");
        string choice = Console.ReadLine().ToLower();
        switch (choice)
        {
            case "name":
                Console.Write("Enter new asset name: ");
                selectedAsset.AssetName = Console.ReadLine();
                break;

            case "purchase date":
            case "purchase":
                Console.Write("Enter new purchase name/date: ");
                selectedAsset.PurchaseName = Console.ReadLine();
                break;

            case "value":
            case "value of asset":
                Console.Write("Enter new value of asset: ");
                double newVal, newRisk;
                while (!double.TryParse(Console.ReadLine(), out  newVal))
                {
                    Console.Write("Invalid input. Enter numeric value: ");
                }
                selectedAsset.ValueOfAsset = newVal;
                break;

            case "risk":
            case "risk value":
                Console.Write("Enter new risk value: ");
                while (!double.TryParse(Console.ReadLine(), out  newRisk))
                {
                    Console.Write("Invalid input. Enter numeric risk value: ");
                }
                selectedAsset.RiskValue = newRisk;
                break;

            default:
                Console.WriteLine("Invalid choice.");
                return;
        }

        Console.WriteLine("Asset updated successfully");
    }

    public void printAsset()
    {
        int i = 0;
        Console.WriteLine("Assets are: \n");
        foreach (Asset a in assets)
        {
            Console.Write($"{i++} -> ");
            Console.WriteLine(a);
        }
    }

    public void removeAsset()
    {
        if (assets.Count == 0)
        {
            Console.WriteLine("No assets to remove.");
            return;
        }
        Console.WriteLine("Enter the index of the asset you want to remove:");
        printAsset();
        if (!int.TryParse(Console.ReadLine(), out int index) || index < 0 || index >= assets.Count)
        {
            Console.WriteLine("Invalid index. No asset removed.");
            return;
        }
        assets.RemoveAt(index);
        Console.WriteLine("Asset removed successfully");
    }


    public double GetTotalAssetValue()
    {
        double total = 0;
        foreach (Asset a in assets)
        {
            total += a.ValueOfAsset;
        }
        return total;
    }
}

class Asset
{
    private UserGoals AssetGoal;
    public string AssetName { get; set; }
    public string PurchaseName { get; set; }
    public double ValueOfAsset { get; set; }
    public double RiskValue { get; set; }

    public UserGoals GetGoal() { return AssetGoal; }
    public Asset(string assetName, string purchaseName, double valueofasset, double riskValue)
    {
        AssetName = assetName;
        PurchaseName = purchaseName;
        ValueOfAsset = valueofasset;
        RiskValue = riskValue;
    }

    public override string ToString()
    {
        return $"Asset: {AssetName}, Purchase: {PurchaseName}, Value: {ValueOfAsset}, Risk: {RiskValue}";
    }

    public void AddGoal(UserGoals goal)
    {
        this.AssetGoal = goal;
    }
}

class calculateZakat
{
    private AssetColletion assetCollection;

    public calculateZakat(AssetColletion assetCol)
    {
        assetCollection = assetCol;
    }

    public double CalculateZakat()
    {
        double total = assetCollection.GetTotalAssetValue();
        return total * 0.025;
    }
}


interface ITransactionStrategy
{
    public void Execute(string transactionID, double amount, BankAccount account);
}

class DepositStrategy : ITransactionStrategy
{
    public void Execute(string transactionID, double amount, BankAccount account)
    {
        account.InternalAddTransaction(transactionID, Transaction.TransactionType.Deposit, amount);
        Console.WriteLine($"Deposited {amount:C} successfully.");
    }
}

class WithdrawalStrategy : ITransactionStrategy
{
    public void Execute(string transactionID, double amount, BankAccount account)
    {
        account.InternalAddTransaction(transactionID, Transaction.TransactionType.Withdrawal, amount);
        Console.WriteLine($"Withdrew {amount:C} successfully.");
    }
}

class TransferStrategy : ITransactionStrategy
{
    public void Execute(string transactionID, double amount, BankAccount account)
    {
        account.InternalAddTransaction(transactionID, Transaction.TransactionType.Transfer, amount);
        Console.WriteLine($"Transferred {amount:C} successfully.");
    }
}

class PaymentStrategy : ITransactionStrategy
{
    public void Execute(string transactionID, double amount, BankAccount account)
    {
        account.InternalAddTransaction(transactionID, Transaction.TransactionType.Payment, amount);
        Console.WriteLine($"Paid {amount:C} successfully.");
    }
}

class BankAccount
{
    private string bankName;
    private string accountNumber;
    private double Amount;
    private List<Transaction> t = new List<Transaction>();

    public BankAccount(string bankName, string accountNumber)
    {
        this.bankName = bankName;
        this.accountNumber = accountNumber;
    }

    public void AddTransaction(ITransactionStrategy strategy, string transactionID, double amount)
    {
        if (strategy == null)
        {
            Console.WriteLine("Transaction strategy is null.");
            return;
        }
        strategy.Execute(transactionID, amount, this);
    }

    internal void InternalAddTransaction(string transactionID, Transaction.TransactionType type, double amount)
    {
        t.Add(new Transaction(type, transactionID, amount));
    }

    public List<Transaction> getList()
    {
        return t;
    }
}

class Transaction
{
    public string TransactionID { get; set; }
    public double Amount { get; set; }
    public TransactionType Type { get; set; }

    public Transaction(TransactionType Type, string TransactionID, double Amount)
    {
        this.Type = Type;
        this.TransactionID = TransactionID;
        this.Amount = Amount;
    }
    public enum TransactionType
    {
        Deposit,
        Withdrawal,
        Transfer,
        Payment
    }
}

class TransacionHistory
{
    public static void ViewTransactionHistory(List<Transaction> t)
    {
        if (t == null || t.Count == 0)
        {
            Console.WriteLine("No transactions found.");
            return;
        }
        foreach (var transaction in t)
        {
            Console.WriteLine($"ID: {transaction.TransactionID}, Type: {transaction.Type}, Amount: {transaction.Amount:C}");
        }
    }
}

class main_menu
{
    private User user;
    private AssetColletion ass_coll;

    private Dictionary<string, ITransactionStrategy> transactionStrategies = new Dictionary<string, ITransactionStrategy>(StringComparer.OrdinalIgnoreCase)
    {
        { "deposit", new DepositStrategy() },
        { "withdrawal", new WithdrawalStrategy() },
        { "transfer", new TransferStrategy() },
        { "payment", new PaymentStrategy() }
    };

    public main_menu(User user, AssetColletion ass_col)
    {
        this.user = user;
        this.ass_coll = ass_col;
    }
    public void start()
    {
        while (true)
        {
            Console.WriteLine("\n==== Main Menu ====");
            Console.WriteLine("1. Manage Assets");
            Console.WriteLine("2. Manage Goals");
            Console.WriteLine("3. View Zakat Calculation");
            Console.WriteLine("4. Bank Account");
            Console.WriteLine("5. Make Transaction");
            Console.WriteLine("6. View Transaction History");
            Console.WriteLine("7. Update Profile");
            Console.WriteLine("8. Logout");
            Console.Write("Choose an option: ");
            string choice = Console.ReadLine();
            switch (choice)
            {
                case "1":
                    managAsset();
                    break;
                case "2":
                    managGoals();
                    break;
                case "3":
                    calculateZakat zakat = new calculateZakat(user.GetassColl());
                    Console.WriteLine($"Zakat due: {zakat.CalculateZakat():C}");
                    break;
                case "4":
                    Console.Write("Please enter your Bank name: ");
                    string Bname = Console.ReadLine();
                    Console.Write("Please enter your Bank Account number: ");
                    string Bnumber = Console.ReadLine();
                    BankAccount BA = new BankAccount(Bname, Bnumber);
                    user.CreateAccount(BA);
                    break;
                case "5":
                    if (user.GetAcount() == null)
                    {
                        Console.WriteLine("You must create a bank account first (Option 4).");
                        break;
                    }
                    Console.Write("Enter the type of transaction (Deposit, Withdrawal, Transfer, Payment): ");
                    string tType = Console.ReadLine();
                    Console.Write("Enter the Transaction ID: ");
                    string Id = Console.ReadLine();
                    Console.Write("Enter the amount: ");
                    if (!double.TryParse(Console.ReadLine(), out double a))
                    {
                        Console.WriteLine("Invalid amount.");
                        break;
                    }

                    if (transactionStrategies.TryGetValue(tType, out ITransactionStrategy strategy))
                    {
                        user.GetAcount().AddTransaction(strategy, Id, a);
                    }
                    else
                    {
                        Console.WriteLine("Invalid transaction type.");
                    }
                    break;
                case "6":
                    if (user.GetAcount() == null)
                    {
                        Console.WriteLine("You must create a bank account first (Option 4).");
                        break;
                    }
                    TransacionHistory.ViewTransactionHistory(user.GetAcount().getList());
                    break;
                case "7":
                    UpdateProfile new_profile = new UpdateProfile(user);
                    new_profile.Update_Profile();
                    break;
                case "8":
                    Console.WriteLine("Logged out successfully.");
                    return;

                default:
                    Console.WriteLine("Invalid option. Try again.");
                    break;
            }
        }
    }
    public void managAsset()
    {
        Console.WriteLine("\n--- Assets ---");
        Console.WriteLine("1. Add New Asset");
        Console.WriteLine("2. View All Assets");
        Console.WriteLine("3. Remove Asset");
        Console.WriteLine("4. Edit Asset");
        Console.Write("Select option: ");
        string gChoice = Console.ReadLine();
        switch (gChoice)
        {
            case "1":
                user.Addass();
                break;
            case "2":
                AssetColletion assC = user.GetassColl();
                if (assC.GetAssets().Count() == 0) Console.WriteLine("No assets added yet.");
                else user.GetassColl().printAsset();
                break;
            case "3":
                assC = user.GetassColl();
                if (assC.GetAssets().Count() == 0) Console.WriteLine("No assets added yet.");
                else user.GetassColl().removeAsset();
                break;
            case "4":
                assC = user.GetassColl();
                if (assC.GetAssets().Count() == 0) Console.WriteLine("No assets added yet.");
                else user.GetassColl().editAsset();
                break;
            default:
                Console.WriteLine("Invalid option.");
                break;
        }
    }

    public void managGoals()
    {
        Console.WriteLine("\n--- Asset Goals ---");
        if (user.GetassColl().GetAssets().Count == 0)
        {
            Console.WriteLine("No assets added yet.");
            return;
        }
        user.GetassColl().printAsset();
        Console.Write("Enter the index of the asset you want to add a goal to: ");
        if (!int.TryParse(Console.ReadLine(), out int choice) || choice < 0 || choice >= user.GetassColl().GetAssets().Count)
        {
            Console.WriteLine("Invalid index.");
            return;
        }
        Asset currentAsset = user.GetassColl().GetAssets()[choice];
        Console.WriteLine("1. Add Goal to Asset");
        Console.WriteLine("2. View Goals");
        Console.WriteLine("3. Add Progress to Goal");
        Console.Write("Select option: ");
        string gChoice = Console.ReadLine();
        switch (gChoice)
        {
            case "1":
                UserGoals userGoals = new UserGoals();
                GoalInputHandler.CreateGoalFromInput(userGoals);
                currentAsset.AddGoal(userGoals);
                userGoals.AttachObserver(new Notify(user));
                break;
            case "2":
                GoalViewer.DisplayGoal(currentAsset.GetGoal());
                break;
            case "3":
                UserGoals goal = currentAsset.GetGoal();
                if (goal == null)
                {
                    Console.WriteLine("No goal set on this asset.");
                    break;
                }
                Console.Write("Enter amount to add: ");
                if (!double.TryParse(Console.ReadLine(), out double valueToAdd))
                {
                    Console.WriteLine("Invalid amount.");
                    break;
                }
                goal.AddProgress(valueToAdd);
                break;
            default:
                Console.WriteLine("Invalid option.");
                break;
        }
    }
}

class Program
{
    static void Main(string[] args)
    {
        List<User> users = new List<User>();
        User currentUser = null;
        AssetColletion assetCollection = new AssetColletion();

        while (true)
        {
            Console.WriteLine("\n==== Welcome to Financial Manager ====");
            Console.WriteLine("1. Sign Up");
            Console.WriteLine("2. Log In");
            Console.WriteLine("3. Exit");
            Console.Write("Choose an option: ");
            string input = Console.ReadLine();
            switch (input)
            {
                case "1":
                    currentUser = new User();
                    Auth_Service.Sign_Up(currentUser, users);
                    if (currentUser.is_Logged_in())
                    {
                        main_menu menu = new main_menu(currentUser, assetCollection);
                        menu.start();
                    }
                    break;
                case "2":
                    currentUser = new User();
                    currentUser = Auth_Service.Log_in(currentUser, users);
                    if (!currentUser.is_Logged_in())
                    {
                        break;
                    }
                    main_menu menu2 = new main_menu(currentUser, assetCollection);
                    menu2.start();
                    break;
                case "3":
                    return;
                default:
                    Console.WriteLine("Invalid option.");
                    break;
            }
        }
    }
}