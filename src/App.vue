<template>
    <div id="app">
        <header>
            <div class="input">
                <input type="date" v-model="input_date"/>
            </div>
            <div class="active_count">
                <!--                <div>{{ active_years_length }} years</div>,-->
                <span>{{ active_months_length }} months</span>,
                <span>{{ active_weeks_length }} weeks</span>,
                <span>{{ active_days_length }} days</span>
            </div>

            <div class="select_weeks" v-if="select_week.hasOwnProperty('date')">
                <div class="string">{{ select_week_format() }}</div>
            </div>
        </header>


        <div id="calendar">

            <div class="years_old">
                <div class="cell"
                     v-for="n in years_old()"
                     :style="yearsOldCellBg(n)">{{ n }}
                </div>
            </div>

            <div class="grid">

                <div v-for="week in result_weeks"
                     @click="selectWeek( week )"
                     :class="{ select : select_week == week, active : week.active }"
                     class="cell">
                    <div class="tooltip">{{ week.date }}</div>
                </div>

            </div>

            <div class="years">
                <div class="cell"
                     v-for="year in years_array">{{ year }}
                </div>
            </div>

        </div>


    </div>
</template>

<script>
    import {
        differenceInYears,
        differenceInMonths,
        differenceInWeeks,
        differenceInDays,
        eachWeekOfInterval,
        format,
        addYears,
        subYears,
        isWithinInterval,
        isBefore,
        parseISO,
        startOfYear,
        differenceInCalendarWeeks
    } from 'date-fns'

    export default {
        name: 'App',
        data() {
            return {
                // options
                weeks_in_year: 52,
                before_years: 5,
                end_of_years: 122,
                // end options

                input_date: localStorage.getItem('calendar_input') || '',

                result_weeks: [
                    // {
                    //     date: Date object,
                    //     active: true/false
                    // }
                ],
                active_days_length: 0,
                active_weeks_length: 0,
                active_months_length: 0,
                active_years_length: 0,
                select_week: {}
            }
        },
        mounted() {
                this.calcWeeksGrid()
        },
        computed: {
            years_array() {
                var array_of_years = [];

                this.result_weeks.forEach(function (week) {
                    if (!array_of_years.includes(week.date.getFullYear())) {
                        array_of_years.push(week.date.getFullYear())
                    }
                })
                return array_of_years;
            },
        },
        methods: {
            select_week_format() {
                var current_week_string = format(this.select_week.date, 'dd MMMMMM yyyy')

                var index_prev_week = this.result_weeks.indexOf(this.select_week);
                if (index_prev_week > 0) {
                    var prev_week = this.result_weeks[index_prev_week - 1];
                    var prev_week_string = format(prev_week.date, 'dd MMMMMM')
                    return `${prev_week_string} - ${current_week_string}`;
                }
                return `${current_week_string}`;
            },
            years_old() {
                var start = -this.before_years;
                var result = [];
                while (start <= this.end_of_years) {
                    result.push(start++);
                }
                return result
            },

            yearsOldCellBg(number) {
                switch (number) {
                    case 100:
                        return {background: '#7ef16b'}
                    case 60: // pensia men
                        return {background: '#00d9ff'}
                    case 55: // pensia woman
                        return {background: '#ff5b78'}
                }

                if (!(number % 10)) return {background: '#a9b9ad'}


            },
            selectWeek(week) {
                this.select_week = week;
            },
            calcWeeksGrid() {
                // Main function for generate calendar weeks grid

                // Arhitecture: ( _Date named is Date object )
                // calendar_start_Date -> 5 year ago from birthday
                // active_start_Date -> from input element, maybe birthday
                // active_now_Date // Now
                // calendar_end_Date // birthday + 122 years. Max duration human life

                console.time("calcWeeksGrid")

                var active_now_Date = new Date();
                var active_start_Date = this.input_date ? parseISO(this.input_date) : active_now_Date; // дата рождения 14-04-1988


                // Проверка что введенное значение не больше сегодняшней даты
                if ( isBefore(active_now_Date, active_start_Date ) ){
                    console.log("isBefore")
                    return false;
                }


                var calendar_start_Date = subYears(active_start_Date, this.before_years) // на 5 ago from birthday 14-04-1983
                var calendar_end_Date = addYears(active_start_Date, this.end_of_years) // до 122 лет 14-04-2110

                    // Всего недель в сетке
                var calendar_weeks_array = eachWeekOfInterval({
                    start:calendar_start_Date,
                    end:calendar_end_Date
                }, {weekStartsOn: 1});

                console.log(calendar_weeks_array)


                var calendar_weeks_obj = [];

                calendar_weeks_array.map(function (week) {

                    var active = isWithinInterval(week, {
                        start: active_start_Date,
                        end: active_now_Date
                    })

                    var obj = {
                        date: week,
                        active: active
                    }
                    calendar_weeks_obj.push(obj)
                })

                // Main return
                this.result_weeks = calendar_weeks_obj;




                // This optionally display count of days or month and weeks, years etc
                this.active_days_length = differenceInDays(
                    active_now_Date,
                    active_start_Date
                );

                this.active_weeks_length = differenceInWeeks(
                    active_now_Date,
                    active_start_Date
                );

                this.active_months_length = differenceInMonths(
                    active_now_Date,
                    active_start_Date
                );

                this.active_years_length = differenceInYears(
                    active_now_Date,
                    active_start_Date
                );

                console.timeEnd("calcWeeksGrid")
            }
        },
        watch: {
            input_date(newVal) {
                if (newVal) {
                    localStorage.setItem('calendar_input', newVal)
                    this.calcWeeksGrid()
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

    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        text-align: center;

        header {
            background: #d7f1cd;
            position: fixed;
            width: 100%;
            top: 0;
            left: 0;
            display: grid;
            grid-gap: 3px;
            grid-template-columns: 1fr 2fr 1fr;
            align-items: center;
            justify-items: start;
            z-index: 9;
            .input {
                input {
                    padding: 10px;
                    font-family: 'Avenir', Helvetica, Arial, sans-serif;
                }
            }

            .active_count {
                /*align-items: center;*/
                /*align-self: start;*/
            }

            .select_weeks {

            }

        }

        #calendar {
            padding: 100px 0;
            width: 1100px;
            margin: 0 auto;
            display: grid;
            grid-template-columns: 20px 1fr 20px;

            .cell {
                font-size: 12px;
                height: 15px;
                border-radius: 2px;
                position: relative;

            }

            .years_old, .years {
                @include grid;
            }

            .grid {
                @include grid;
                grid-template-columns: repeat(52, 1fr);

                .cell {
                    border: 1px #868686 solid;
                    .tooltip {
                        width: 120px;
                        position: absolute;
                        background: white;
                        display: none;
                        z-index: 9;
                        bottom: calc( 100% + 4px );
                        left: 50%;
                        transform: translateX(-50%);
                        padding: 10px;
                        border: 1px black solid;
                        border-radius: 3px;
                        &:before {
                            content: '';
                            position: absolute;
                            border: 6px solid transparent;
                            border-top: 6px solid black;
                            bottom: -12px;
                            left: 63px;
                        }
                    }
                    &:hover {
                        .tooltip {
                            display: block;
                        }
                    }

                    &.active {
                        cursor: pointer;
                        background: #42b983;
                        border: 1px #42b983 solid;

                        &.select, &:hover {
                            background: #c7c6f7;
                            border: 1px #c7c6f7 solid;
                        }
                    }
                }
            }
        }
    }


</style>
