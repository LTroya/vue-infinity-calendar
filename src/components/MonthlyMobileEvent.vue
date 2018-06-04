<template>
    <div>
        <p>Day: {{ day }}</p>
        <p>{{ event.price }}</p>
        <p>{{ event.desc }}</p>
    </div>
</template>

<script>
    import moment from 'moment';

    export default {
        props: {
            data: {
                type: Object,
                required: true
            },

            month: {
                type: String
            },

            date: {
                type: String
            }
        },

        data() {
            return {
                event: this.data
            }
        },

        computed: {
            day() {
                return moment(this.date).format('dd DD');
            }
        }
    }
</script>