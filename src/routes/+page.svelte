<script lang="ts">
  import Button from "$lib/components/ui/button/button.svelte";
  import Input from "$lib/components/ui/input/input.svelte";

  type Todo = {
    id: string;
    text: string;
    done: boolean;
  };

  type Filter = "all" | "active" | "completed";

  let todos = $state<Todo[]>([]);

  let filter = $state<Filter>("all");

  let filteredTodos = $derived(filterTodos());

  $effect(() => {
    const savedTodos = localStorage.getItem("todos");
    savedTodos && (todos = JSON.parse(savedTodos));
  });

  $effect(() => {
    localStorage.setItem("todos", JSON.stringify(todos));
  });

  function addTodo(event: KeyboardEvent) {
    if (event.key !== "Enter") return;
    const todoElement = event.target as HTMLInputElement;
    const id = window.crypto.randomUUID();
    const text = todoElement.value;
    const done = false;
    todos = [...todos, { id, text, done }];
    todoElement.value = "";
  }

  function editTodo(event: Event) {
    const inputElement = event.target as HTMLInputElement;
    const index = +inputElement.dataset.index!;
    todos[index].text = inputElement.value;
  }

  function toggleTodo(event: Event) {
    const checkboxElement = event.target as HTMLInputElement;
    const index = +checkboxElement.dataset.index!;
    todos[index].done = !todos[index].done;
  }

  function setFilter(newFilter: Filter) {
    filter = newFilter;
  }

  function filterTodos() {
    switch (filter) {
      case "all":
        return todos;
      case "active":
        return todos.filter((todo) => !todo.done);
      case "completed":
        return todos.filter((todo) => todo.done);
    }
  }

  function remaining() {
    return todos.filter((todo) => !todo.done).length;
  }

  // $effect(() => {
  //   console.log("todos", todos);
  //   console.log("filter", filter);
  // });
  $inspect(todos);
</script>

<main class="mt-12">
  <h1 class="place-self-center">Todos</h1>
  <div class="todos">
    {#each filteredTodos as todo, i}
      <div class:completed={todo.done} class="todo">
        <input oninput={editTodo} data-index={i} value={todo.text} type="text" />
        <input onchange={toggleTodo} data-index={i} value={todo.done} type="checkbox" />
      </div>
    {/each}
    <Input type="text" placeholder="Add ToDo" onkeyup={addTodo} />
  </div>
  <div class="[&>*]:filters place-self-center hover:[&>Button:nth-child(2)]:text-indigo-500">
    <Button onclick={() => setFilter("all")}>All</Button>
    <Button onclick={() => setFilter("active")}>Active</Button>
    <Button onclick={() => setFilter("completed")}>Completed</Button>
    <p>{remaining()} remaining</p>
  </div>
</main>

<style>
</style>
