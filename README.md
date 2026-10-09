const STORAGE_KEY = "todo-list-items";
const FILTERS = {
  all: () => true,
  active: (task) => !task.completed,
  completed: (task) => task.completed,
};

const todoForm = document.getElementById("todo-form");
const todoInput = document.getElementById("todo-input");
const todoList = document.getElementById("todo-list");
const taskCount = document.getElementById("task-count");
const clearCompletedBtn = document.getElementById("clear-completed");
const filterButtons = document.querySelectorAll(".filter-btn");

let tasks = JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];
let currentFilter = "all";

function saveTasks() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks));
}

function render() {
  const visibleTasks = tasks.filter(FILTERS[currentFilter]);

  todoList.innerHTML = "";

  if (visibleTasks.length === 0) {
    const empty = document.createElement("li");
    empty.className = "empty-state";
    empty.textContent = "No tasks to display";
    todoList.appendChild(empty);
  } else {
    visibleTasks.forEach((task) => {
      const li = document.createElement("li");
      li.className = `todo-item ${task.completed ? "completed" : ""}`;

      const main = document.createElement("div");
      main.className = "todo-main";

      const checkbox = document.createElement("input");
      checkbox.type = "checkbox";
      checkbox.className = "todo-checkbox";
      checkbox.checked = task.completed;
      checkbox.setAttribute("aria-label", `Mark "${task.text}" as complete`);
      checkbox.addEventListener("change", () => {
        task.completed = checkbox.checked;
        saveTasks();
        render();
      });

      const text = document.createElement("span");
      text.className = "todo-text";
      text.textContent = task.text;

      const deleteBtn = document.createElement("button");
      deleteBtn.type = "button";
      deleteBtn.className = "delete-btn";
      deleteBtn.textContent = "×";
      deleteBtn.setAttribute("aria-label", `Delete "${task.text}"`);
      deleteBtn.addEventListener("click", () => {
        tasks = tasks.filter((item) => item.id !== task.id);
        saveTasks();
        render();
      });

      main.appendChild(checkbox);
      main.appendChild(text);

      li.appendChild(main);
      li.appendChild(deleteBtn);

      todoList.appendChild(li);
    });
  }

  const remaining = tasks.filter((task) => !task.completed).length;
  taskCount.textContent =
    remaining === 1 ? "1 task left" : `${remaining} tasks left`;
}

function addTask(text) {
  const trimmed = text.trim();

  if (!trimmed) {
    todoInput.focus();
    return;
  }

  tasks.unshift({
    id: Date.now() + Math.random(),
    text: trimmed,
    completed: false,
  });

  saveTasks();
  render();
  todoInput.value = "";
  todoInput.focus();
}

todoForm.addEventListener("submit", (event) => {
  event.preventDefault();
  addTask(todoInput.value);
});

filterButtons.forEach((button) => {
  button.addEventListener("click", () => {
    currentFilter = button.dataset.filter;
    filterButtons.forEach((btn) => btn.classList.toggle("active", btn === button));
    render();
  });
});

clearCompletedBtn.addEventListener("click", () => {
  tasks = tasks.filter((task) => !task.completed);
  saveTasks();
  render();
});

render();
