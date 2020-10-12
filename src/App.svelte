<script>
	// init
	let userObject = null;
	const userbase = window.userbase;
	let authPromise = userbase.init({ appId: __myapp.env.BASE_APP_ID })
		.then(({user}) => userObject = user);

	// auth
	let username, password;

	const signIn = () => authPromise = userbase.signIn({username, password}).then(user => userObject = user);
	const signUp = () => authPromise = userbase.signUp({username, password}).then(user => userObject = user);
	const signOut = () => authPromise = userbase.signOut().then(() => userObject = null);

	// $: console.log(userObject);

	// db
	let todoPromise;
	let todos = []
	let newTodo = '';
	const databaseName = 'todos';
	const changeHandler = (items) => todos = items;
	$: if(userObject) todoPromise = userbase.openDatabase({ databaseName, changeHandler });
	const addTodo = () => {
		if(newTodo) {
			todoPromise = userbase.insertItem({ databaseName, item: newTodo });
			newTodo = '';
		}
	}
	const deleteTodo = (itemId) => {
		todoPromise = userbase.deleteItem({ databaseName, itemId });
	}
	const updateTodo = (index) => {
		const {item, itemId} = todos[index];
		todoPromise = userbase.updateItem({ databaseName, itemId, item })
	}
</script>

{#await authPromise}Loading...{:then _}
	{#if !userObject}
		<form>
			<label for="username">Username</label>
			<input id="username" type="text" bind:value={username}><br>
			<label for="password">Password</label>
			<input id="password" type="password" bind:value={password}><br>
			<button on:click={signIn} type="button">Sign in</button>
			<button on:click={signUp} type="button">Sign up</button>
		</form>
	{:else}
		<h1>Hi, {userObject.username}!</h1>
		<button on:click={signOut} style="position: absolute; top: 10px; right: 10px;">🚪 Sign out</button>
		{#await todoPromise}Loading todos...{:then _}
			<label for="new-todo">New Todo</label>
			<input id="new-todo" type="text" bind:value={newTodo}>
			<button on:click={addTodo}>Add todo</button><br>
			{#each todos as {item, itemId}, index}
				<input type="text" bind:value={todos[index].item} on:blur={() => updateTodo(index)}>
				<button on:click={() => deleteTodo(itemId)}>X</button><br>
			{/each}
		{:catch error} Error! {error} {/await}
	{/if}
{:catch error} Error! {error} {/await}