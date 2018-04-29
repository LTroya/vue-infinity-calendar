<template>
    <div>
        <label>
            <select name="select" v-model="selectedMonth">
                <option v-for="month in months" :value="month" :key="month.id">{{ month.text }}</option>
            </select>
        </label>

        <button class="btn btn-primary" v-if="hasPrevious" @click="previousMonth">Previous</button>
        {{ selectedMonth && selectedMonth.text }}
        <button class="btn btn-primary" v-if="hasNext" @click="nextMonth">Next</button>

        <div class="container-fluid calendar--wrapper">
            <!-- Days -->
            <div class="row flex-nowrap">
                <div class="col calendar--header" v-for="day in days" :key="day">
                    {{ day | dayFormat }}
                </div>
            </div>

            <!-- Day cells -->
            <div class="row flex-nowrap" v-for="(week, weekIndex) in weeks" :key="week.id">
                <div class="col"
                     v-for="(day, dayIndex) in week.days"
                     v-bind:id="id(weekIndex, dayIndex, day.value)"
                     :key="day.id"
                     :class="classesFor(day)"
                     @click="onDayClick(day)"
                     tabindex="0">

                    <div class="calendar--day d-flex justify-content-between">
                        <span>{{ day.value | number }}</span>
                        <span v-if="isFirstDayOfMonth(weekIndex, dayIndex, day.value)">{{ day.value | month }}</span>
                    </div>

                    <calendar-day-event :event="eventFor(day)"/>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    /* eslint-disable no-console */
    import Moment from 'moment';
    import {extendMoment} from 'moment-range';

    import CalendarDayEvent from './CalendarDayEvent';

    const moment = extendMoment(Moment);
    window.moment = moment;

    export default {
        name: 'FlexCalendar',

        components: {CalendarDayEvent},

        props: {
            days: {
                type: Array,
                default() {
                    return ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'];
                }
            },

            startDate: {
                default() {
                    return moment.now()
                }
            },

            endDate: {
                default() {
                    return moment().add(90, 'days');
                }
            },

            events: {
                type: Object,
                default() {
                    return {
                        '2018-04-28': {
                            price: 32.323,
                            desc: 45,
                            duration: 3,
                            products: [{
                                product_id: 18,
                                price: 23123
                            }]
                        },
                        '2018-04-30': {
                            price: 32.323,
                            desc: 45,
                            duration: 2,
                            products: [{
                                product_id: 18,
                                price: 23123
                            }]
                        },
                        '2018-05-06': {
                            price: 32.323,
                            desc: 45,
                            duration: 7,
                            products: [{
                                product_id: 18,
                                price: 23123
                            }]
                        },
                        '2018-05-17': {
                            price: 32.323,
                            desc: 45,
                            duration: 5,
                            products: [{
                                product_id: 18,
                                price: 23123
                            }]
                        }
                    }
                }
            }
        },

        data: () => ({
            weeks: [],
            months: [],
            selectedMonth: null,
            selectedDays: {},
            log: true
        }),

        mounted() {
            const date1 = moment(this.startDate);
            const date2 = moment(this.endDate);

            if (this.log) this.showDebugInfo(date1, date2);
        },

        filters: {
            number(date) {
                return date.format('DD');
            },

            month(date) {
                return date.format('MMMM');
            },

            dayFormat(value) {
                return value.substr(0, 1).toUpperCase() + value.substr(1, 1).toLowerCase();
            },
        },

        watch: {
            selectedMonth({value}) {
                const id = value.toLowerCase(),
                    el = document.querySelector(`#${id}`);

                if (el) el.scrollIntoView({
                    behavior: 'smooth'
                });
            }
        },

        computed: {
            hasPrevious() {
                if (!this.selectedMonth) return false;
                const index = this.getMonthIndex();
                return index !== 0;
            },

            hasNext() {
                if (!this.selectedMonth) return false;
                const index = this.getMonthIndex();
                return index < this.months.length - 1;
            }
        },

        methods: {
            getMonthIndex() {
                return this.months.findIndex(m => m.id === this.selectedMonth.id);
            },

            previousMonth() {
                const index = this.getMonthIndex();
                this.selectedMonth = this.months[index - 1];
            },

            nextMonth() {
                const index = this.getMonthIndex();
                this.selectedMonth = this.months[index + 1];
            },

            diffInDays(d1, d2) {
                return this.diff(d1, d2, 'days');
            },

            diffInWeeks(d1, d2) {
                return this.diff(d1, d2, 'weeks');
            },

            diffInMonths(d1, d2) {
                return this.diff(d1, d2, 'months');
            },

            /**
             * Calculate diff between two dates
             *
             * @param d1 - from, instance of moment
             * @param d2 - to, instance of moment
             * @param measurement - weeks, days, months, year, hours, minutes or seconds
             * @returns {*|number}
             */
            diff(d1, d2, measurement) {
                return d2.diff(d1, measurement);
            },

            diffBetweenDates(date1, date2) {
                const diffInDays = this.diffInDays(date1, date2);
                const diffInWeeks = this.diffInWeeks(date1, date2);
                const diffInMonths = this.diffInMonths(date1, date2);
                const weeks = diffInWeeks + 1;

                for (let index = 0; index <= diffInMonths; index++) {
                    const nextMonth = moment(date1).add(index, 'month').format('MMMM');
                    this.months.push({id: this.uui(), value: nextMonth, text: nextMonth})
                }
                this.selectedMonth = this.months[0];

                if (this.log) this.showDebugMonths();

                if (this.log) console.groupCollapsed('Weeks');
                this.buildWeeks(date1, weeks, diffInDays);
                if (this.log) console.groupEnd('Weeks');
            },

            buildWeeks(date1, weeks) {
                this.weeks = Array(weeks)
                    .fill([])
                    .map((_, index) => ({id: this.uui(), days: this.addDays(date1, index)}));
            },

            addDays(initialDate, weekIndex) {
                const start = weekIndex * 7;
                const result = Array(7).fill().map((_, index) => {
                    const value = moment(initialDate).add(start + index, 'day');

                    return {id: this.uui(), value};
                });

                if (this.log) this.showDebugWeeks(result, weekIndex);

                return result;
            },

            isFirstDayOfMonth(weekIndex, dayIndex, day) {
                return weekIndex === 0 && dayIndex === 0
                    || parseInt(day.format('D')) === 1;
            },

            id(weekIndex, dayIndex, day) {
                return this.isFirstDayOfMonth(weekIndex, dayIndex, day)
                    ? day.format('MMMM').toLowerCase()
                    : '';
            },

            uui() {
                return Math.random() * 100000000;
            },

            shorFormat(date) {
                return date.format('YYYY-MM-DD');
            },

            hasDayEvent(day) {
                const d = this.shorFormat(day.value);
                return this.events[d];
            },

            eventFor(day) {
                const d = this.shorFormat(day.value);
                return this.events[d];
            },

            showDebugInfo(date1, date2) {
                console.group('Calendar');

                console.groupCollapsed('Day information');
                console.groupCollapsed('Date #1');
                console.log('Index of the week', date1.day());
                console.log('Index of the month', date1.month());
                console.log('Iso index of the week', weekDay1);
                console.log('Day of the week', this.days[weekDay1]);
                console.groupEnd();
                console.groupCollapsed('Date #2');
                console.log('Index of the week', date2.day());
                console.log('Iso index of the week', weekDay2);
                console.log('Day of the week', this.days[weekDay2]);
                console.groupEnd();
                console.groupEnd();

                this.diffBetweenDates(date1, date2);
                console.groupEnd();

                const weekDay1 = date1.isoWeekday() - 1;
                const weekDay2 = date2.isoWeekday() - 1;

                const diffInDays = this.diffInDays(date1, date2);
                const diffInWeeks = this.diffInWeeks(date1, date2);
                const diffInMonths = this.diffInMonths(date1, date2);

                console.groupCollapsed('Diff between dates');
                console.log('Diff in days', diffInDays);
                console.log('Diff in weeks', diffInWeeks);
                console.log('Diff in months', diffInMonths);
                console.groupEnd();

                console.groupCollapsed('Some maths');
                console.log(`Weeks => ${diffInDays} / 7 =`, diffInDays / 7);
                console.log(`Remaining days => ${diffInDays} mod 7 =`, diffInDays % 7);
                console.log('Added week', diffInDays % 7 === 0);
                console.groupEnd();

                console.groupCollapsed('Events');
                Object.keys(this.events).forEach(event => {
                    const {price, desc, products} = this.events[event];
                    console.groupCollapsed(`Event ${event}`);
                    console.log(`Price`, price);
                    console.log(`Desc`, desc);
                    console.log(`Products`, [...products]);
                    console.groupEnd();
                });
                console.groupEnd();

                const fromDate = moment();
                const toDate = moment().add(15, 'days');
                const range = moment().range(fromDate, toDate);
                const diff = range.diff('days');
                const days = Array.from(range.by('days'));

                console.group('Moment range calculations');
                console.log(`fromDate`, fromDate);
                console.log(`toDate`, toDate);
                console.log(`Range`, range);
                console.log(`Diff`, diff);
                console.log(`Days`, days);
                console.groupCollapsed('Moment range calculations');
                days.forEach((day, index) => {
                    console.log(`#${index} Day`, day.format('DD-MM-YYYY'));
                });
                console.groupEnd();
                console.groupEnd();
            },

            showDebugMonths() {
                console.groupCollapsed('Months');
                console.log(this.months);
                console.groupEnd();
            },

            showDebugWeeks(result, weekIndex) {
                console.groupCollapsed(`Week ${weekIndex + 1}`);
                console.log(result);
                console.groupEnd();
            },

            classesFor(day) {
                const d = this.shorFormat(day.value);
                const classes = (this.selectedDays[d] || {}).classes;

                return classes ? classes : [];
            },

            onDayClick(day) {
                if (!this.hasDayEvent(day)) return this.selectedDays = {};

                const {duration} = this.eventFor(day),
                    {value: d} = day,
                    toDate = moment(d).add(duration, 'days'),
                    range = moment().range(d, toDate),
                    days = Array.from(range.by('days'));

                this.selectedDays = {};
                days.forEach(day => {
                    this.selectedDays[this.shorFormat(day)] = {
                        classes: 'highlight'
                    }
                });
            }
        }
    }
</script>

<style>
    .col,
    .calendar--day--cell {
        background: #e6e6e6;
        color: #6a6a6d;
        border: 1px solid white;
    }

    .event--price {
        font-weight: bolder;
        text-align: center;
        font-size: 20px;
        padding: 15px 0;
    }

    .event--discount {
        color: #4c89b5;
        font-size: 15px;
        text-align: center;
        font-weight: bold;
    }

    .calendar--day {
        font-weight: 300;
        font-size: 15px;
    }

    .calendar--header {
        background: #196aa5;
        color: #fff;
        padding: 5px 0 5px 10px;
        font-weight: bold;
        min-width: 112px;
    }

    .calendar--wrapper {
        overflow: auto;
        height: 530px;
    }

    .highlight {
        background: #fff;
        color: #111;
    }
</style>