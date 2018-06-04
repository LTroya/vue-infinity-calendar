<template>
    <div id="app">
        <calendar :start-date="startDate"
                  :end-date="endDate">
            <template slot="heading" slot-scope="props">
                <button class="btn btn-primary" v-if="props.hasPrevious" @click="props.previousMonth">Previous</button>
                {{ props.currentMonth && props.currentMonth.text }}
                <button class="btn btn-primary" v-if="props.hasNext" @click="props.nextMonth">Next</button>
            </template>
        </calendar>

        <div v-for="month in Object.keys(eventsByMonth)" :key="month">
            {{ month }}
            <monthly-mobile-event
                    v-for="date in Object.keys(eventsByMonth[month])"
                    :data="eventFrom(month, date)"
                    :key="date"
                    :date="date"
                    :month="month"
            ></monthly-mobile-event>
        </div>
    </div>
</template>

<script>
    /* eslint-disable no-console */

    import '@/css/styles.scss';
    import Calendar from './components/Calendar.vue';
    import MonthlyMobileEvent from './components/MonthlyMobileEvent.vue';
    import Todos from './components/todos/TodosConsumer';

    import moment from 'moment';

    window.moment = moment;

    export default {
        name: 'app',
        components: {
            Todos,
            Calendar,
            MonthlyMobileEvent
        },

        data: () => ({
            startDate: moment(),
            endDate: moment().add(10, 'days'),
            additionalDays: 90,
            events: {
                '2018-05-03': {
                    price: 28.215,
                    desc: 45
                },
                '2018-05-10': {
                    price: 28.215,
                    desc: 45
                },
                '2018-05-12': {
                    price: 28.215,
                    desc: 45
                },
                '2018-06-07': {
                    price: 28.215,
                    desc: 45
                },
                '2018-06-03': {
                    price: 28.215,
                    desc: 45
                },
                '2018-06-20': {
                    price: 28.215,
                    desc: 45
                },
                '2018-07-03': {
                    price: 28.215,
                    desc: 45
                },
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
            },

            eventFrom(month, date) {
                return this.eventsByMonth[month][date];
            }
        }
    }
</script>
