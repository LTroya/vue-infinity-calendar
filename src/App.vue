<template>
    <div id="app">
        <calendar
                :start-date="startDate"
                :end-date="endDate">
            <template slot="heading" slot-scope="props">
                <button class="btn btn-primary" v-if="props.hasPrevious" @click="props.previousMonth">Previous</button>
                {{ props.currentMonth && props.currentMonth.text }}
                <button class="btn btn-primary" v-if="props.hasNext" @click="props.nextMonth">Next</button>
            </template>
        </calendar>
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

        methods: {}
    }
</script>
