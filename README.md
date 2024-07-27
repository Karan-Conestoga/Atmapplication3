using System;
using System.Collections.Generic;

namespace KaranPreetAssignment3
{
    public class AtmApplication
    {
        private Bank bank;
        private Account currentAccount;

        public AtmApplication()
        {
            bank = new Bank();
        }

        public void CreateAccount(int accountNumber, double initialBalance, double interestRate, string accountHolderName)
        {
            bank.AddAccount(new Account(accountNumber, initialBalance, interestRate, accountHolderName));
        }

        public Account SelectAccount(int accountNumber)
        {
            currentAccount = bank.GetAccount(accountNumber);
            return currentAccount;
        }

        public double CheckBalance()
        {
            if (currentAccount != null)
            {
                return currentAccount.GetBalance();
            }
            throw new InvalidOperationException("No account selected.");
        }

        public void Deposit(double amount)
        {
            if (currentAccount != null)
            {
                currentAccount.Deposit(amount);
            }
            else
            {
                throw new InvalidOperationException("No account selected.");
            }
        }

        public bool Withdraw(double amount)
        {
            if (currentAccount != null)
            {
                return currentAccount.Withdraw(amount);
            }
            throw new InvalidOperationException("No account selected.");
        }

        public List<string> GetTransactions()
        {
            if (currentAccount != null)
            {
                return currentAccount.GetTransactions();
            }
            throw new InvalidOperationException("No account selected.");
        }

        public void ExitAccount()
        {
            currentAccount = null;
        }
    }
}
-

using System.Collections.Generic;

namespace KaranPreetAssignment3
{
    public class Bank
    {
        private List<Account> accounts;

        public Bank()
        {
            // Initialize with 10 default accounts
            accounts = new List<Account>();
            for (int i = 100; i < 110; i++)
            {
                accounts.Add(new Account(i, 100, 3, $"Holder{i}"));
            }
        }

        public void AddAccount(Account account)
        {
            accounts.Add(account);
        }

        public Account GetAccount(int accountNumber)
        {
            foreach (var account in accounts)
            {
                if (account.AccountNumber == accountNumber)
                {
                    return account;
                }
            }
            return null; // or throw an exception if you prefer
        }
    }
}

using System;
using System.Collections.Generic;

namespace KaranPreetAssignment3
{
    public class Account
    {
        public int AccountNumber { get; set; }
        public double Balance { get; private set; }
        public double InterestRate { get; set; }
        public string AccountHolderName { get; set; }
        private List<string> transactions;

        public Account(int accountNumber, double initialBalance, double interestRate, string accountHolderName)
        {
            AccountNumber = accountNumber;
            Balance = initialBalance;
            InterestRate = interestRate;
            AccountHolderName = accountHolderName;
            transactions = new List<string>();
            AddTransaction("Account created with initial balance: " + initialBalance);
        }

        public void Deposit(double amount)
        {
            if (amount > 0)
            {
                Balance += amount;
                AddTransaction("Deposited: " + amount);
            }
        }

        public bool Withdraw(double amount)
        {
            if (amount > 0 && amount <= Balance)
            {
                Balance -= amount;
                AddTransaction("Withdrew: " + amount);
                return true;
            }
            return false;
        }

        public double GetBalance()
        {
            return Balance;
        }

        public List<string> GetTransactions()
        {
            return transactions;
        }

        private void AddTransaction(string transaction)
        {
            transactions.Add(transaction);
        }
    }
}

using System;
using System.Collections.Generic;
using System.Windows;

namespace KaranPreetAssignment3
{
    public partial class MainWindow : Window
    {
        private AtmApplication atmApplication;

        public MainWindow()
        {
            InitializeComponent();
            atmApplication = new AtmApplication();
        }

        private void CreateAccount_Click(object sender, RoutedEventArgs e)
        {
            CreateAccountPanel.Visibility = Visibility.Visible;
            SelectAccountPanel.Visibility = Visibility.Collapsed;
            AccountMenuPanel.Visibility = Visibility.Collapsed;
        }

        private void SubmitAccount_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                int accountNumber = int.Parse(AccountNumberTextBox.Text);
                double initialBalance = double.Parse(InitialBalanceTextBox.Text);
                double interestRate = double.Parse(InterestRateTextBox.Text);
                string accountHolderName = AccountHolderNameTextBox.Text;

                atmApplication.CreateAccount(accountNumber, initialBalance, interestRate, accountHolderName);
                MessageBox.Show("Account created successfully!");

                CreateAccountPanel.Visibility = Visibility.Collapsed;
            }
            catch (FormatException)
            {
                MessageBox.Show("Please enter valid data.");
            }
        }

        private void SelectAccount_Click(object sender, RoutedEventArgs e)
        {
            SelectAccountPanel.Visibility = Visibility.Visible;
            CreateAccountPanel.Visibility = Visibility.Collapsed;
            AccountMenuPanel.Visibility = Visibility.Collapsed;
        }

        private void SubmitSelection_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                int accountNumber = int.Parse(SelectAccountNumberTextBox.Text);
                var account = atmApplication.SelectAccount(accountNumber);

                if (account != null)
                {
                    AccountMenuPanel.Visibility = Visibility.Visible;
                    AccountMenuTextBlock.Text = $"Account Number: {account.AccountNumber}\nHolder Name: {account.AccountHolderName}\nBalance: {account.GetBalance()}";

                    SelectAccountPanel.Visibility = Visibility.Collapsed;
                }
                else
                {
                    MessageBox.Show("Account not found.");
                }
            }
            catch (FormatException)
            {
                MessageBox.Show("Please enter a valid account number.");
            }
        }

        private void CheckBalance_Click(object sender, RoutedEventArgs e)
        {
            double balance = atmApplication.CheckBalance();
            MessageBox.Show($"Balance: {balance}");
        }

        private void Deposit_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                string amountText = Microsoft.VisualBasic.Interaction.InputBox("Enter amount to deposit:", "Deposit", "0");
                double amount = double.Parse(amountText);
                atmApplication.Deposit(amount);
                MessageBox.Show("Deposit successful!");
            }
            catch (FormatException)
            {
                MessageBox.Show("Please enter a valid amount.");
            }
        }

        private void Withdraw_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                string amountText = Microsoft.VisualBasic.Interaction.InputBox("Enter amount to withdraw:", "Withdraw", "0");
                double amount = double.Parse(amountText);

                if (atmApplication.Withdraw(amount))
                {
                    MessageBox.Show("Withdrawal successful!");
                }
                else
                {
                    MessageBox.Show("Insufficient funds.");
                }
            }
            catch (FormatException)
            {
                MessageBox.Show("Please enter a valid amount.");
            }
        }

        private void DisplayTransactions_Click(object sender, RoutedEventArgs e)
        {
            List<string> transactions = atmApplication.GetTransactions();

            if (transactions.Count > 0)
            {
                string transactionsText = string.Join("\n", transactions);
                MessageBox.Show($"Transactions:\n{transactionsText}");
            }
            else
            {
                MessageBox.Show("No transactions found.");
            }
        }

        private void ExitAccount_Click(object sender, RoutedEventArgs e)
        {
            atmApplication.ExitAccount();
            SelectAccountPanel.Visibility = Visibility.Visible;
            AccountMenuPanel.Visibility = Visibility.Collapsed;
        }

        private void Exit_Click(object sender, RoutedEventArgs e)
        {
            Application.Current.Shutdown();
        }
    }
}
