<template>
    <main class="main-wrapper">
        <h1 class="page-title">My Todo App</h1>
        <p class="subtitle">A todo app powered by vue3 and vite!</p>
        <div class="new-task-wrapper">
            <input
                type="text"
                placeholder="Type a new todo item"
                class="new-task-input"
                v-model="newTaskInput"
                @keyup.enter="addTask"
            />
            <button
                class="new-task-button"
                @click="addTask"
            >+Add</button>
        </div>
        <nav>
            <ul class="tab-nav-wrapper">
                <li class="tab-nav-item">
                    <button
                        class="tab-nav-item-button"
                        @click="setView('All')"
                    >All ({{ allTaskLength }})</button>
                </li>
                <li class="tab-nav-item">
                    <button
                        class="tab-nav-item-button"
                        @click="setView('Current')"
                    >Current ({{ currentTaskLength }})</button>
                </li>
                <li class="
                        tab-nav-item">
                    <button
                        class="tab-nav-item-button"
                        @click="setView('Completed')"
                    >Completed ({{ completedTaskLength }})</button>
                </li>
            </ul>
        </nav>
        <ul class="task-list">
            <li
                v-for="taskItem in tasksInView"
                :key="taskItem.id"
                class="task-list-item"
            >
                <div class="task-list-checkbox-wrapper">
                    <IconCheckCircle v-show="taskItem.complete" />
                    <IconCircle
                        v-show="!taskItem.complete"
                        @click=""
                    />
                    <input
                        type="checkbox"
                        v-model="taskItem.complete"
                        :checked="taskItem.complete"
                        class="task-list-checkbox"
                    />
                </div>
                <input
                    v-if="taskItem.edit"
                    class="task-list-edit-input"
                    type="text"
                    v-model="taskItem.label"
                />
                <p
                    v-else
                    class="task-list-text"
                    :class="taskItem.complete ? 'is-complete' : ''"
                >
                    {{ taskItem.label }}
                </p>
                <div class="task-list-cta">
                    <IconEdit
                        class="task-list-cta-icon"
                        @click="toggleEdit(taskItem.id)"
                    /><span class="sr-only">Edit</span>
                    <IconDelete
                        class="task-list-cta-icon"
                        @click="deleteTask(taskItem.id)"
                    /><span class="sr-only">Delete</span>
                </div>
            </li>
        </ul>
    </main>
</template>

<script>
import { computed, reactive, toRefs } from 'vue'
import { v4 as uuid } from 'uuid'
import IconCheckCircle from './components/IconCheckCircle.vue'
import IconCircle from './components/IconCircle.vue'
import IconDelete from './components/IconDelete.vue'
import IconEdit from './components/IconEdit.vue'

export default {
    name: 'App',
    components: {
        IconCheckCircle,
        IconCircle,
        IconDelete,
        IconEdit
    },
    setup () {
        const state = reactive({
            currentView: 'All',
            newTaskInput: '',
            taskList: []
        })
        const taskLists = reactive({
            all: computed(() => state.taskList),
            current: computed(() =>
                state.taskList.filter(item => item.complete === false)
            ),
            completed: computed(() =>
                state.taskList.filter(item => item.complete === true)
            )
        })

        const taskViews = reactive({
            allTaskLength: computed(() => taskLists.all.length),
            currentTaskLength: computed(() => taskLists.current.length),
            completedTaskLength: computed(() => taskLists.completed.length)
        })

        const taskListOverview = reactive([
            { name: 'All', length: computed(() => taskLists.all.length) },
            { name: 'Current', length: computed(() => taskLists.current.length) },
            { name: 'Completed', length: computed(() => taskLists.completed.length) }
        ])
        const tasksInView = computed(() => {
            if (state.currentView === 'Current') {
                return state.taskList.filter(item => item.complete === false)
            } else if (state.currentView === 'Completed') {
                return state.taskList.filter(item => item.complete === true)
            } else {
                return state.taskList
            }
        })
        const addTask = () => {
            state.taskList.push({
                id: uuid(),
                complete: false,
                edit: false,
                label: state.newTaskInput
            })
            state.newTaskInput = ''
        }
        const toggleEdit = taskId => {
            const taskIndex = state.taskList.findIndex(task => task.id === taskId)
            state.taskList[taskIndex].edit = !state.taskList[taskIndex].edit
        }
        const deleteTask = taskId => {
            const taskIndex = state.taskList.findIndex(task => task.id === taskId)
            state.taskList.splice(taskIndex, 1)
        }
        const setView = viewLabel => {
            state.currentView = viewLabel
        }
        return {
            ...toRefs(state),
            ...toRefs(taskViews),
            addTask,
            deleteTask,
            setView,
            tasksInView,
            toggleEdit,
            taskListOverview
        }
    }
}
</script>
<style>
html {
    background-color: #fbfbfb;
}

.tab-nav-wrapper {
    display: flex;
    column-gap: 30px;
    list-style: none;
    margin: 45px 0 20px;
    padding: 0;
}

.tab-nav-item {
    padding-bottom: 2px;
}
.tab-nav-item:hover .tab-button {
    color: #0728bf;
}
.tab-nav-item.is-active {
    border-bottom: 3px solid #0631f8;
}
.tab-nav-item.is-active .tab-button {
    color: #2d2d2d;
}
.tab-nav-item-button {
    border: 0;
    background-color: transparent;
    color: #6b6b6b;
    letter-spacing: 1px;
    font-weight: bold;
    font-family: 'Source Sans Pro';
    padding: 0;
    font-size: 1rem;
}
.tab-nav-item-button:hover {
    cursor: pointer;
}

.task-list-checkbox-wrapper {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
}
.task-list-checkbox {
    position: absolute;
    left: -3px;
    bottom: 2px;
    opacity: 0;
}
.task-list {
    padding: 0;
}
.task-list-cta-icon {
    opacity: 0;
    transition: 0.2s opacity ease-in;
}
.task-list-cta-icon .icon-object {
    fill: #2d2d2d;
}
.task-list-cta-icon:hover .icon-object {
    fill: #0728bf;
}
.task-list-item {
    border: 1px solid #f6f6f6;
    box-shadow: 2px 2px 8px 0 #cfcfcf;
    border-radius: 8px;
    display: flex;
    align-items: center;
    padding: 0 16px;
    margin-bottom: 16px;
}
.task-list-item:hover {
    border: 1px solid #0631f8;
}
.task-list-item:hover .task-list-cta-icon,
.task-list-item:focus .task-list-cta-icon {
    opacity: 1;
}
.task-list-cta {
    display: flex;
    column-gap: 16px;
}
.task-list-edit-input,
.task-list-text {
    margin-left: 12px;
    font-weight: bold;
    flex: 1;
    border: 0;
    font-size: 16px;
}
.task-list-text.is-complete {
    color: #6b6b6b;
    text-decoration: line-through;
}

.main-wrapper {
    max-width: 600px;
    margin: 0 auto;
}

.page-title {
    font-family: 'DM Serif Display', serif;
    font-size: 44px;
    letter-spacing: 1.84px;
    color: #2d2d2d;
    margin-top: 104px;
    margin-bottom: 0;
}
.subtitle {
    font-size: 1em;
    font-weight: bold;
    color: #6b6b6b;
    margin-top: 0;
}

.new-task-wrapper {
    display: flex;
}
.new-task-input {
    padding: 16px;
    font-weight: 600;
    color: #2d2d2d;
    flex: 1;
    border-top-left-radius: 8px;
    border-bottom-left-radius: 8px;
    border: 1px solid #f6f6f6;
    box-shadow: 2px 2px 8px 0 #c0c0c0;
    letter-spacing: 1px;
    font-size: 1rem;
}
.new-task-input:hover {
    border: 1px solid #0631f8;
}
.new-task-input::placeholder {
    color: #959595;
}
.new-task-button {
    background-color: #0631f8;
    color: #fff;
    padding: 18px 24px;
    font-weight: 900;
    border: 0;
    border-top-right-radius: 8px;
    border-bottom-right-radius: 8px;
    transition: 0.2s background ease-in;
    font-size: 1rem;
}
.new-task-button:hover {
    background-color: #082ac9;
}
</style>