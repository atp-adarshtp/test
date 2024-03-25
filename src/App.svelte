<script>
    import { setContext, onMount } from "svelte";
    import Navbar from "./Navbar.svelte";
    import ExpensesList from "./ExpensesList.svelte";
    import Totals from "./Totals.svelte";
    import ExpenseForm from "./ExpenseForm.svelte";

    let expenses = [];
    let setName = "";
    let setAmount = null;
    let setId = null;
    let isFormOpen = false;
    let isEditing = false;
	$: isEditing = setId ? true : false;
	$: total = expenses.reduce((acc, curr) => {
		console.log({ acc, amount: curr.amount });
		return (acc += curr.amount);
	}, 0);


    function showForm() {
        isFormOpen = true;
    }

    function hideForm() {
        isFormOpen = false;
        setName = "";
        setAmount = null;
        setId = null;
    }

    function removeExpense(id) {
        expenses = expenses.filter((item) => item.id !== id);
        setLocalStorage();
    }

    function clearExpenses() {
        expenses = [];
        setLocalStorage();
    }

    function addExpense({ name, amount }) {
        let expense = { id: Math.random() * Date.now(), name, amount };
        expenses = [expense, ...expenses];
        setLocalStorage();
    }

    function setModifiedExpense(id) {
        let expense = expenses.find((item) => item.id === id);
		console.log(expense);
        setId = expense.id;
        setName = expense.name;
        setAmount = expense.amount;
        showForm();
    }

    function editExpense({ name, amount }) {
        expenses = expenses.map((item) => {
            return item.id === setId ? { ...item, name, amount } : { ...item };
        });
        setId = null;
        setAmount = null;
        setName = "";
        setLocalStorage();
    }

	setContext("remove", removeExpense);
	setContext("modify", setModifiedExpense);

    function setLocalStorage() {
        localStorage.setItem("expenses", JSON.stringify(expenses));
    }

    onMount(() => {
        expenses = localStorage.getItem("expenses")
            ? JSON.parse(localStorage.getItem("expenses"))
            : [];
        // total = expenses.reduce((acc, curr) => acc + curr.amount, 0);
    });
</script>

<Navbar {showForm} />
<main class="content">
    {#if isFormOpen}
        <ExpenseForm
            {addExpense}
            name={setName}
            amount={setAmount}
            {isEditing}
            {editExpense}
            {hideForm}
        />
    {/if}
    <Totals title="total expenses" {total} />
    <ExpensesList {expenses} />
    <button on:click={clearExpenses} class="btn btn-primary btn-block">
        Clear Expenses
    </button>
</main>
