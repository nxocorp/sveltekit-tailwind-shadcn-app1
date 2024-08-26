<script lang="ts">

	import { onMount } from 'svelte';
	import { writable } from 'svelte/store';

	import { Button } from '$lib/components/ui/button';	

	const todos = writable([]);
	
	// Load data from localStorage
	onMount(() => {
		const storedTodos = localStorage.getItem('todos');
		if (storedTodos) {
			todos.set(JSON.parse(storedTodos));
		}
	});

	// Save data to localStorage
	const saveToLocalStorage = (items) => {
		localStorage.setItem('todos', JSON.stringify(items));
	};

	// Add a new todo
	const addTodo = (data) => {
		todos.update((currentTodos) => {
			const newTodos = [ ...currentTodos, { 
				id: 				Date.now(), 
				title: 			data.title.value || '', 
				completed: 	data.check.checked, 
				index: 			currentTodos.length
			}];
			saveToLocalStorage(newTodos);
			return newTodos;
		});
	};

	// Update a todo
	const updateTodo = (id, updates) => {
		todos.update((currentTodos) => {
			const newTodos = currentTodos.map((todo) =>
				todo.id === id ? { ...todo, ...updates } : todo
			);
			saveToLocalStorage(newTodos);
			return newTodos;
		});
	};

	// Delete a todo
	const deleteTodo = (id) => {
		todos.update((currentTodos) => {
			const newTodos = currentTodos
				.filter((todo) => todo.id !== id)
				.map((todo, index) => ({ ...todo, index }));
			saveToLocalStorage(newTodos);
			return newTodos;
		});
	};

	// Function to move a todo item up or down
	const moveTodo = (id, direction) => {
		todos.update((currentTodos) => {
			const index = currentTodos.findIndex(todo => todo.id === id);
			if ((direction === 'up' && index === 0) || (direction === 'down' && index === currentTodos.length - 1)) {
					return currentTodos; // Can't move further
			}

			const newIndex = direction === 'up' ? index - 1 : index + 1;
			const newTodos = [...currentTodos];
			[newTodos[index], newTodos[newIndex]] = [newTodos[newIndex], newTodos[index]];

			newTodos[index].index = index;
			newTodos[newIndex].index = newIndex;
			
			saveToLocalStorage(newTodos);
			return newTodos;
		});
	};

	
</script>


<main class="max-w-xl mx-auto mt-10">
	<h1 class="text-2xl font-bold mb-5">Todo List</h1>

	<form class="flex mb-4"
		on:submit|preventDefault={() => {
			const title = document.querySelector('#newTitle');
			const check = document.querySelector('#newCheck');
			const data = {title, check};
			if (title) {
				addTodo(data);
				title.value = '';
			}
		}}
	>
		<input type="checkbox" 
			id="newCheck"
			class="mr-2" />
		<input type="text" 
			id="newTitle"
			class="flex-1 p-2 border rounded-l mr-2"
			placeholder="Add a new todo..." />
		<Button type="submit" 
			class="p-2 bg-blue-500 text-white rounded-r">
			Add
			</Button>
	</form>

	<ul>
		{#each $todos.sort((a, b) => a.index - b.index) as todo }
			<li class="flex justify-between items-center mb-2 {todo.completed ? 'opacity-50' : ''}" >
				
				<input type="checkbox" 
					class="mr-2" 
					checked={todo.completed}
					on:change={(e) => { updateTodo(todo.id, { completed: e.target.checked }); }} />

				<Button
					class="p-1 mr-1 bg-gray-300 text-black rounded"
					disabled={todo.index === 0} 
					on:click={() => moveTodo(todo.id, 'up')} > ↑ </Button>
				<Button
					class="p-1 mr-2 bg-gray-300 text-black rounded"
					disabled={todo.index === $todos.length - 1} 
					on:click={() => moveTodo(todo.id, 'down')} > ↓ </Button>
				
				<input type="text"
					class="flex-1 p-2 border rounded mr-2"
					placeholder="Enter todo item"
					value={todo.title}
					on:input={(e) => { updateTodo(todo.id, { title: e.target.value }); }}
					/>
				
				<Button
					class="p-2 bg-red-500 text-white rounded"
					on:click={() => deleteTodo(todo.id)}
					> Delete </Button>
				
			</li>
		{/each}
		
	</ul>
</main>

<style>
	
</style>
