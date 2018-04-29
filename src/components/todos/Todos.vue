<template>
    <ul>
        <li v-for="todo in todos" v-bind:key="todo.id">
            <slot v-bind:todo="todo" v-bind:alter="alter">
                {{ todo.text }}

                <button @click="alter(todo)">Change</button>
            </slot>
        </li>
    </ul>
</template>

<script>
    import uuidv1 from 'uuid/v1';
    export default {
        data: () => ({
            todos: [
                {id: uuidv1(), text: 'Hello World' },
                {id: uuidv1(), text: 'Hello World 2' }
            ]
        }),

        methods: {
            alter(todo, nextText) {
                todo.text = `Changed to id ===> ${nextText ? nextText : todo.id}`;
            }
        }
    }
</script>