<template>
    <div id="app">
        <input type="date" v-model="input_date"/>

        <div> {{ result_weeks.length }} недель</div>

        <div v-if="result_weeks.length > 0"
             id="calendar">

            <div class="years_old">
                <div class="cell"
                     v-for="n in years_old">{{ n }}
                </div>
            </div>

            <div class="grid">
                <div v-for="cell in before_years * weeks_in_year"
                     class="cell"></div>

                <div v-for="week in result_weeks"
                     @click="mouseOver(week)"
                     :class="{ select : select_week == week }"
                     class="cell active"></div>

                <div v-for="cell in after_years * weeks_in_year"
                     class="cell"></div>
            </div>

            <div class="years">
                <div class="cell"
                     v-for="year in years">{{ year }}
                </div>
            </div>

        </div>

        <div class="fix_bottom" v-if="select_week">
            <div class="container">
                <div class="row">{{ select_week_format }}</div>
            </div>
        </div>
    </div>
</template>

<script>
    import { eachWeekOfInterval, format } from 'date-fns'

    export default {
        name: 'App',
        data() {
            return {
                weeks_in_year: 52,
                before_years: 10,
                input_date: localStorage.getItem('calendar_input') || '',
                after_years: 10,

                before_weeks: [],
                result_weeks: [],
                after_weeks: [],

                select_week: ''
            }
        },
        computed: {
            select_week_format(){
                var current_week_string = format(this.select_week, 'dd MMMMMM yyyy')

                var index_prev_week = this.result_weeks.indexOf(this.select_week);
                if ( index_prev_week > 0 ){
                    var prev_week = this.result_weeks[index_prev_week - 1];
                    var prev_week_string = format(prev_week, 'dd MMMMMM')
                    return `${prev_week_string} - ${current_week_string}`;
                }
                return `${current_week_string}`;
            },
            years_old() {
                var self = this;
                var array_of_years_old = [];
                var length = Math.floor(this.result_weeks.length / this.weeks_in_year) + self.after_years;

                for (let i = 0; i <= length; i++) {
                    array_of_years_old.push(i)
                }

                for (let i = 0; i < self.before_years; i++) {
                    var value_for_unshift = array_of_years_old[0] - 1;
                    array_of_years_old.unshift(value_for_unshift)
                }

                return array_of_years_old;
            },
            years() {
                var array_of_years = [];

                this.result_weeks.forEach(function (week) {
                    var year = week.getFullYear();
                    if (!array_of_years.includes(year)) {
                        array_of_years.push(year)
                    }
                });

                for (let i = 0; i < this.before_years; i++) {
                    var value_for_unshift = array_of_years[0] - 1;
                    array_of_years.unshift(value_for_unshift)
                }
                for (let i = 0; i < this.after_years; i++) {
                    var value_for_push = array_of_years[array_of_years.length - 1] + 1;
                    array_of_years.push(value_for_push)
                }

                return array_of_years;
            }
        },
        mounted() {
            if ( this.input_date ){
                this.calcWeeks()
            }
        },
        methods: {
            mouseOver( week ) {
                this.select_week = week;
            },
            calcWeeks(){
                // this.before_weeks = eachWeekOfInterval({
                //   start: new Date(),
                //   end: new Date( newVal )
                // });

                this.result_weeks = eachWeekOfInterval({
                    start: new Date( this.input_date ),
                    end: new Date()
                });


                // this.after_weeks = eachWeekOfInterval({
                //   start: new Date(newVal),
                //   end: new Date()
                // });
            }
        },
        watch: {
            input_date(newVal) {
                if ( newVal ){
                    localStorage.setItem('calendar_input', newVal)
                    this.calcWeeks()
                }
            }
        }
    }
</script>

<style lang="scss">
    @mixin grid {
        display: grid;
        grid-gap: 3px;
    }

    input {
        padding: 10px;
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
    }

    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        text-align: center;
        padding-bottom: 80px;
    }

    #calendar {
        width: 900px;
        margin: 0 auto;

        display: grid;
        grid-template-columns: 20px 1fr 20px;

        .years_old, .years {
            @include grid;
        }

        .grid {
            @include grid;
            grid-template-columns: repeat(52, 1fr);
            .cell {
                background: #a9b9ad;
                &.active {
                    cursor: pointer;
                    background: #42b983;
                    &.select, &:hover {
                        background: #154610;
                    }
                }

            }
        }
    }

    .fix_bottom {
        position: fixed;
        bottom: 0;
        left: 0;
        background: #e4e4e4;
        width: 100%;
        padding: 20px 0;
    }
    .cell {
        font-size: 12px;
        /*background: #368109;*/
        height: 14px;
    }

</style>
