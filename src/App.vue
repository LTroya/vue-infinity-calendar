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
    /* eslint-disable no-console */

    import '@/css/styles.scss';
    import Calendar from './components/Calendar.vue';
    import Todos from './components/todos/TodosConsumer';

    import moment from 'moment';
    window.moment = moment;

    export default {
        name: 'app',
        components: {
            Todos,
            Calendar
        },

        data: () => ({
            startDate: moment(),
            endDate: moment().add(90, 'days'),
            additionalDays: 90,
            events: {
                '2018-05-03': {id: 1},
                '2018-05-10': {id: 1},
                '2018-05-12': {id: 1},
                '2018-06-07': {id: 1},
                '2018-06-03': {id: 1},
                '2018-06-20': {id: 1},
                '2018-07-03': {id: 1},
            },
            eventsByMonth: {}
        }),

        mounted() {
            console.group(`Events`);
            this.eventsByMonth = this.splitEventsInMonths(this.events);
            this.logEventsByMonth(this.eventsByMonth);
            console.groupEnd();
        },

        methods: {
            splitEventsInMonths(events) {
                let eventsByMonth = {};
                Object.keys(this.events).forEach(date => {
                    const month = moment(date).format('MMMM');
                    eventsByMonth[month] = eventsByMonth[month] || {};
                    eventsByMonth[month][date] = events[date];
                });
                return eventsByMonth;
            },

            logEventsByMonth(eventsByMonth) {
                Object.keys(eventsByMonth).forEach(month => {
                    console.group(`Month ${month}`);
                    Object.keys(eventsByMonth[month]).forEach(date => {
                        console.log(`${date}`, eventsByMonth[month][date]);
                    });
                    console.groupEnd();
                });
            }
        }
    }
</script>
