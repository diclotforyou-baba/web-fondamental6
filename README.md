
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SpendWise - Budget Tracker</title>

    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: Arial, sans-serif;
            background: #f4f6f9;
            color: #333;
            padding: 20px;
        }

        header {
            background: #176b4d;
            color: white;
            text-align: center;
            padding: 25px;
            border-radius: 10px;
            margin-bottom: 25px;
        }

        header h1 {
            margin-bottom: 8px;
        }

        main {
            max-width: 850px;
            margin: auto;
        }

        section {
            background: white;
            padding: 20px;
            margin-bottom: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.08);
        }

        h2 {
            margin-bottom: 15px;
            color: #176b4d;
        }

        .summary {
            display: flex;
            gap: 15px;
            flex-wrap: wrap;
        }

        .card {
            flex: 1;
            min-width: 150px;
            background: #e7f4ed;
            padding: 20px;
            border-radius: 8px;
            text-align: center;
        }

        .card p {
            font-size: 20px;
            font-weight: bold;
            margin-top: 10px;
        }

        form {
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        input, select, button {
            padding: 12px;
            border: 1px solid #ccc;
            border-radius: 6px;
            font-size: 15px;
        }

        button {
            background: #176b4d;
            color: white;
            border: none;
            cursor: pointer;
        }

        button:hover {
            background: #105238;
        }

        #budgetMessage {
            margin-top: 15px;
            font-weight: bold;
        }

        #expenseList {
            list-style: none;
        }

        #expenseList li {
            display: flex;
            justify-content: space-between;
            align-items: center;
            gap: 10px;
            padding: 12px;
            border-bottom: 1px solid #ddd;
        }

        .delete-btn {
            background: #c0392b;
            padding: 8px 12px;
        }

        @media (max-width: 600px) {
            .summary {
                flex-direction: column;
            }
        }
    </style>
</head>

<body>

    <header>
        <h1>SpendWise</h1>
        <p>Manage your money wisely!</p>
    </header>

    <main>

        <!-- Dashboard -->
        <section>
            <h2>My Dashboard</h2>

            <div class="summary">

                <div class="card">
                    <h3>My Budget</h3>
                    <p id="budgetDisplay">KSh 0.00</p>
                </div>

                <div class="card">
                    <h3>Total Expenses</h3>
                    <p id="totalExpenses">KSh 0.00</p>
                </div>

                <div class="card">
                    <h3>Balance</h3>
                    <p id="balance">KSh 0.00</p>
                </div>

            </div>

            <p id="budgetMessage">
                Set your budget to get started.
            </p>
        </section>

        <!-- Set Budget -->
        <section>
            <h2>Set Your Budget</h2>

            <form id="budgetForm">

                <input
                    type="number"
                    id="budgetInput"
                    placeholder="Enter your budget"
                    min="0.01"
                    step="0.01"
                    required
                >

                <button type="submit">Set Budget</button>

            </form>
        </section>

        <!-- Add Expense -->
        <section>
            <h2>Add an Expense</h2>

            <form id="expenseForm">

                <input
                    type="text"
                    id="expenseName"
                    placeholder="Expense name"
                    required
                >

                <input
                    type="number"
                    id="expenseAmount"
                    placeholder="Amount (KSh)"
                    min="0.01"
                    step="0.01"
                    required
                >

                <select id="expenseCategory" required>
                    <option value="">Select category</option>
                    <option value="Food">Food</option>
                    <option value="Transport">Transport</option>
                    <option value="Shopping">Shopping</option>
                    <option value="Bills">Bills</option>
                    <option value="Other">Other</option>
                </select>

                <button type="submit">Add Expense</button>

            </form>
        </section>

        <!-- Expense Records -->
        <section>
            <h2>Expense Records</h2>

            <ul id="expenseList"></ul>
        </section>

    </main>

    <script>

        // 1. Store expenses in an array

        let expenses = [];
        let budget = 0;


        // 2. Select HTML elements

        const budgetForm =
            document.getElementById("budgetForm");

        const expenseForm =
            document.getElementById("expenseForm");

        const budgetDisplay =
            document.getElementById("budgetDisplay");

        const totalExpensesDisplay =
            document.getElementById("totalExpenses");

        const balanceDisplay =
            document.getElementById("balance");

        const budgetMessage =
            document.getElementById("budgetMessage");

        const expenseList =
            document.getElementById("expenseList");


        // 3. Format money

        function formatMoney(amount) {
            return "KSh " + amount.toLocaleString("en-KE", {
                minimumFractionDigits: 2,
                maximumFractionDigits: 2
            });
        }


        // 4. Set budget using an event listener

        budgetForm.addEventListener("submit", function(event) {

            event.preventDefault();

            const newBudget = Number(
                document.getElementById("budgetInput").value
            );

            if (!Number.isFinite(newBudget) || newBudget <= 0) {

                alert("Please enter a valid budget.");

                return;
            }

            budget = newBudget;

            updateDashboard();

            budgetForm.reset();

        });


        // 5. Add expenses using an event listener

        expenseForm.addEventListener("submit", function(event) {

            event.preventDefault();

            const name =
                document.getElementById("expenseName").value.trim();

            const amount = Number(
                document.getElementById("expenseAmount").value
            );

            const category =
                document.getElementById("expenseCategory").value;


            if (
                !name ||
                !category ||
                !Number.isFinite(amount) ||
                amount <= 0
            ) {

                alert("Please enter valid expense details.");

                return;
            }


            // Store expense as an object in the array

            const expense = {
                name: name,
                amount: amount,
                category: category
            };

            expenses.push(expense);

            expenseForm.reset();

            updateDashboard();

        });


        // 6. Calculate total expenses using a loop

        function calculateTotal() {

            let total = 0;

            for (let i = 0; i < expenses.length; i++) {

                total += expenses[i].amount;

            }

            return total;

        }


        // 7. Update dashboard dynamically

        function updateDashboard() {

            const total = calculateTotal();

            const remaining = budget - total;

            budgetDisplay.textContent =
                formatMoney(budget);

            totalExpensesDisplay.textContent =
                formatMoney(total);

            balanceDisplay.textContent =
                formatMoney(remaining);


            // 8. Make decisions using conditional statements

            if (budget === 0) {

                budgetMessage.textContent =
                    "Please set your budget.";

            } else if (remaining < 0) {

                budgetMessage.textContent =
                    "Warning! You have exceeded your budget.";

            } else if (remaining === 0) {

                budgetMessage.textContent =
                    "You have used your entire budget.";

            } else if (total >= budget * 0.8) {

                budgetMessage.textContent =
                    "Careful! You have used 80% or more of your budget.";

            } else {

                budgetMessage.textContent =
                    "Great! You are within your budget.";

            }


            displayExpenses();

        }


        // 9. Display expenses using a loop and DOM manipulation

        function displayExpenses() {

            expenseList.textContent = "";

            for (let i = 0; i < expenses.length; i++) {

                const expense = expenses[i];

                const listItem =
                    document.createElement("li");

                const details =
                    document.createElement("span");

                details.textContent =
                    expense.name + " (" +
                    expense.category + ") - " +
                    formatMoney(expense.amount);


                // Create delete button

                const deleteButton =
                    document.createElement("button");

                deleteButton.textContent = "Delete";

                deleteButton.className = "delete-btn";


                // 10. Handle delete button clicks

                deleteButton.addEventListener("click", function() {

                    expenses.splice(i, 1);

                    updateDashboard();

                });


                listItem.appendChild(details);

                listItem.appendChild(deleteButton);

                expenseList.appendChild(listItem);

            }

        }


        // 11. Display initial dashboard

        updateDashboard();

    </script>

</body>
</html>
