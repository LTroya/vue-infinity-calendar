<template>
    <div class="calendar--day--event">
        <h3 class="event--price">{{ event.price }}</h3>
        <p class="event--discount" v-text="desc"></p>
    </div>
</template>

<script>
    export default {
        props: {
            event: {
                type: Object,
                default() {
                    return {};
                }
            }
        },

        computed: {
            desc() {
                return this.event.desc ? `${this.event.desc}% desc` : '';
            }
        }
    }
</script>

<style>
    .calendar--day--event {
        height: 100px;
    }
</style>
