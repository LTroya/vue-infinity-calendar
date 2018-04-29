<template>
    <todos>
        <template slot-scope="props">
            <p>{{ props.todo.id}} => {{ props.todo.text }}</p>

            <button @click="props.alter(props.todo, 'nextText')">
                Change
            </button>
        </template>
    </todos>
</template>

<script>
    import Todos from './Todos';

    export default {
        components: {Todos}
    }
</script>