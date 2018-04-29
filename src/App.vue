<template>
    <div id="app">
        <calendar/>
    </div>
</template>

<script>
    import '@/css/styles.scss';
    import Calendar from './components/Calendar.vue';
    import Todos from './components/todos/TodosConsumer';

    export default {
        name: 'app',
        components: {
            Todos,
            Calendar
        }
    }
</script>
