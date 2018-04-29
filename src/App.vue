<template>
    <div id="app">
        <button @click="changeDate">Change</button>
        <calendar
            :start-date="startDate"
            :end-date="endDate"
        />
    </div>
</template>

<script>
    import '@/css/styles.scss';
    import Calendar from './components/Calendar.vue';
    import Todos from './components/todos/TodosConsumer';

    import moment from 'moment';

    export default {
        name: 'app',
        components: {
            Todos,
            Calendar
        },

        data: () => ({
            startDate: moment(),
            endDate: moment().add(90, 'days'),
            additionalDays: 90
        }),

        methods: {
            changeDate(){
                this.startDate = moment().add(30, 'days');
                this.endDate = moment(this.startDate).add(this.additionalDays, 'days');
            }
        }
    }
</script>
