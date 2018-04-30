<template>
    <div>
        <slot name="heading"
              :has-previous="hasPrevious"
              :has-next="hasNext"
              :previous-month="previousMonth"
              :next-month="nextMonth"
              :current-month="selectedMonth">
            <button class="btn btn-primary" v-if="hasPrevious" @click="previousMonth">Previous</button>
            {{ selectedMonth && selectedMonth.text }}
            <button class="btn btn-primary" v-if="hasNext" @click="nextMonth">Next</button>
        </slot>

        <div class="container-fluid calendar--wrapper">
            <!-- Days -->
            <div class="row flex-nowrap">
                <div class="col calendar--header" v-for="day in days" :key="day">
                    {{ day | dayFormat }}
                </div>
            </div>

            <!-- Day cells -->
            <div class="row flex-nowrap" v-for="(week, weekIndex) in weeks" :key="week.id">
                <div class="col calendar--day--container"
                     v-for="(day, dayIndex) in week.days"
                     :id="id(weekIndex, dayIndex, day.value)"
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
    import Moment from 'moment';
    import {extendMoment} from 'moment-range';
    import CalendarDayEvent from './CalendarDayEvent';

    const moment = extendMoment(Moment);

    export default {
        name: 'Calendar',

        components: {CalendarDayEvent},

        props: {
            days: {
                type: Array,
                default() {
                    return ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'];
                }
            },

            startDate: {},

            endDate: {},

            events: {
                type: Object,
                default() {
                    return {
                        [moment().add(2, 'days').format('YYYY-MM-DD')]: {
                            price: 32.323,
                            desc: 45,
                            duration: 3,
                            products: [{
                                product_id: 18,
                                price: 23123
                            }]
                        },
                        [moment().add(7, 'days').format('YYYY-MM-DD')]: {
                            price: 32.323,
                            desc: 45,
                            duration: 2,
                            products: [{
                                product_id: 18,
                                price: 23123
                            }]
                        },
                        [moment().add(10, 'days').format('YYYY-MM-DD')]: {
                            price: 32.323,
                            desc: 45,
                            duration: 7,
                            products: [{
                                product_id: 18,
                                price: 23123
                            }]
                        },
                        [moment().add(15, 'days').format('YYYY-MM-DD')]: {
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

        mounted() {
            this.diffBetweenDates(
                moment(this.startDate),
                moment(this.endDate)
            );
        },

        watch: {
            selectedMonth({value}) {
                const id = value.toLowerCase(),
                    el = document.querySelector(`#${id}`);

                if (el) el.scrollIntoView({
                    behavior: 'smooth'
                });
            },

            startDate() {
                this.diffBetweenDates(
                    moment(this.startDate),
                    moment(this.endDate)
                );
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

                this.months = [];
                for (let index = 0; index <= diffInMonths; index++) {
                    const nextMonth = moment(date1).add(index, 'month').format('MMMM');
                    this.months.push({id: this.uui(), value: nextMonth, text: nextMonth})
                }
                this.selectedMonth = this.months[0];

                this.buildWeeks(date1, weeks, diffInDays);
            },

            buildWeeks(date1, weeks) {
                this.weeks = Array(weeks)
                    .fill([])
                    .map((_, index) => ({id: this.uui(), days: this.addDays(date1, index)}));
            },

            addDays(initialDate, weekIndex) {
                const start = weekIndex * 7;
                return Array(7).fill().map((_, index) => {
                    const value = moment(initialDate).add(start + index, 'day');

                    return {id: this.uui(), value};
                });
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
                days.forEach((day, index) => {
                    const classes = index === 0 ? ['highlight', 'first--highlight'] : 'highlight';

                    this.selectedDays[this.shorFormat(day)] = {
                        classes
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

    .calendar--day--container:focus {
        outline: 0;
    }

    .calendar--day--container {
        border: 2px solid transparent;
    }

    .calendar--day--container:hover {
        border: 2px solid cornflowerblue;
    }

    .first--highlight,
    .calendar--day--container.first--highlight:hover {
        border: 0;
        border-left: 5px solid cornflowerblue;
    }
</style>