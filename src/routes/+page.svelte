<script lang="ts">
	import { initializeApp } from 'firebase/app';
	import {
		getFirestore,
		collection,
		onSnapshot,
		QuerySnapshot,
		doc,
		updateDoc,
		deleteDoc,
		addDoc
	} from 'firebase/firestore';
	import { firebaseConfig } from '$lib/firebaseConfig';

	const firebaseApp = initializeApp(firebaseConfig);
	const db = getFirestore();

	interface todo {
		task?: string;
		isComplete?: boolean;
		createdAt?: Date;
		id: string;
	}

	let todos: todo[] = [];
	let task = '';
	let error = '';

	const colRef = collection(db, 'todos');
	const getTodos = onSnapshot(colRef, (QuerySnapshot) => {
		let fbTodos: todo[] = [];
		QuerySnapshot.forEach((doc) => {
			let todo = { ...doc.data(), id: doc.id };
			fbTodos = [todo, ...fbTodos];
		});
		todos = fbTodos;
	});

	console.log({ firebaseApp, db });

	const addTodo = async () => {
		if (task !== '') {
			const docRef = await addDoc(collection(db, 'todos'), {
				task: task,
				isComplete: false,
				createdAt: new Date()
			});
			error = '';
		} else {
			error = 'Task is empty';
		}
		task = '';
	};

	const markTodoAsComplete = async (todo: todo) => {
		//todos[index].isComplete = !todos[index].isComplete;
		await updateDoc(doc(db, 'todos', todo.id), {
			isComplete: !todo.isComplete
		});
	};

	const deleteTodo = async (id: string) => {
		await deleteDoc(doc(db, 'todos', id));
	};

	const keyIsPressed = (event: KeyboardEvent) => {
		if (event.key === 'Enter') {
			addTodo();
		}
	};
</script>

<div class="h-screen bg-base-200 flex items-center flex-col">
	<h1 class="text-3xl font-bold mt-10">Todo App</h1>
	<div class="mt-5">
		<div class="flex justify-between">
			<input
				type="text"
				placeholder="Add a task"
				bind:value={task}
				class="input max-w-xs mr-2"
			/>
			<button on:click={addTodo} class="btn btn-primary">Add</button>
		</div>

		<ol class="mt-5 px-1 bg-base-100 rounded-lg">
			{#each todos as todo}
				<li class="flex justify-between p-2 font-semibold">
					<span class:complete={todo.isComplete}>
						{todo.task}
					</span>
					<div>
						<span>
							<button
								on:click={() => markTodoAsComplete(todo)}
								class="text-lg text-green-500 font-bold hover:text-green-600">✓</button
							>
						</span>
						<span>
							<button
								on:click={() => deleteTodo(todo.id)}
								class="text-lg text-red-600 hover:text-red-700 font-bold">✗</button
							>
						</span>
					</div>
				</li>
			{:else}
				<p>No todos found</p>
			{/each}
			<p class="error">{error}</p>
		</ol>
	</div>
</div>

<svelte:window on:keydown={keyIsPressed} />

<style>
	.complete {
		text-decoration: line-through;
		@apply text-gray-400 ;
	}
	.error {
		@apply text-red-600;
	}
</style>
