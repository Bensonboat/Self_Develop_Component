<template>
    <div>
        <div class="calendar-block">
            <div class="year-select-block">
                <div class="year-select-txt" @click="toggleYearSelect">
                    {{yearSelect}}
                    <span style="font-size: 12px; cursor: pointer">
                        <i class="fas fa-chevron-down"></i>
                    </span>
                </div>
                <div v-show='showYearSelect' class='year-select-modal'>
                    <div class="year-option" v-for='item in yearSelections' :key='item' @click="yearSelectUpdate(item)">{{item}}</div>
                </div>
                <div class="date-display-txt">
                    <div>{{weekTxt[new Date(this.yearSelect, this.monthSelect -1, this.currentDate).getDay()]}},</div>
                    <div>{{this.monthSelect | monthTextSwitcher | shortHand(3)}} {{currentDate}}</div>
                </div>
            </div>
            <div class="calendar-right-side" :class="[autoIncreaseHeight ? 'increase-height' : '']">
                <div class="month-year-display-block">
                    <div class='month-btn left-btn' @click="monthSelectUpdate(-1)">
                        <i class="fas fa-chevron-down"></i>
                    </div>
                    <div class="month-select-block">
                        <span style="margin-right: 10px">{{monthSelect | monthTextSwitcher}}</span>
                        <span>{{yearSelect}}</span>
                    </div>
                    <div class='month-btn right-btn' @click="monthSelectUpdate(1)">
                        <i class="fas fa-chevron-down"></i>
                    </div>
                </div>
                <transition name="fade">
                    <div class="week-txt-row" v-if="showDate">
                        <div v-for='item in weekTxt' :key="item" class="week-txt">{{item | shortHand(2)}}</div>
                        <div
                            v-for='(day, index) in days'
                            :key="index" class='date-block'
                            :class="[day.status === 'key' ? 'key-date-color' : '',
                                    day.status === 'past' ? 'past-date-color' : '',
                                    ]">
                            <div
                                :class="['date', { 'current-date-color' : currentDate === day.date && day.status !== 'past' }]"
                                @click="day.status === 'key' ? $emit('pick-date', day.date) : ''" >
                                {{ day.date }}</div>
                        </div>
                    </div>
                </transition>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: 'calendarDisplay',
        props: {
            calendarKeyDate: {
                type: Array,
                default: []
            }
        },
        data(){
            return {
                weekTxt: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],
                monthSelect: new Date().getMonth() + 1,
                yearSelect: new Date().getFullYear(),
                days: '',
                keyDate: [],
                currentDate: new Date().getDate(),
                showYearSelect: false,
                yearSelections: [2018, 2019, 2020, 2021, 2022, 2023],
                showDate: true,
                autoIncreaseHeight: false,
            }
        },
        filters: {
            monthTextSwitcher(month){
                switch (month){
                    case 1:
                        return 'January';
                    case 2:
                        return 'February';
                    case 3:
                        return 'March';
                    case 4:
                        return 'April';
                    case 5:
                        return 'May';
                    case 6:
                        return 'Jun';
                    case 7:
                        return 'July';
                    case 8:
                        return 'August';
                    case 9:
                        return 'September';
                    case 10:
                        return 'October';
                    case 11:
                        return 'November';
                    case 12:
                        return 'December';
                }
            },
            shortHand(data, num){
                return data.slice(0, num)
            }
        },
        mounted(){
            this.refreshDate();
            this.buildYearSelectList()
        },
        watch: {
            calendarKeyDate(data) {
                this.keyDate = data.map((item)=> {
                    return parseInt(item);
                });
                this.refrashKeyDate();
            },
            monthSelect(month) {
                this.refrashKeyDate();
                this.currentDate = new Date().getDate();
                if (month !== new Date().getMonth() + 1) {
                    this.currentDate = null;
                }
                this.$emit('get-date', `${this.yearSelect}-${month < 10 ? `0${month}` : month}`);
            },
            yearSelect(year){
                this.showYearSelect = false;
                this.refrashKeyDate();
                this.currentDate = new Date().getDate();
                if (year !== new Date().getFullYear()) {
                    this.currentDate = null;
                }
                this.$emit('get-date', `${year}-${this.monthSelect < 10? `0${this.monthSelect}`: this.monthSelect}`);
            }
        },
        methods: {
            refrashKeyDate() {
                new Promise(resolve => {
                    this.showDate = false;
                    resolve()
                }).then(() => {
                    this.showDate = true;
                    this.refreshDate();
                });
            },
            // 每月 1 號 禮拜幾
            weekDayStart(){
                let month = this.monthSelect;
                let year = this.yearSelect;
                let day = new Date(year, month -1, 1);
                return (day.getDay());
            },
            // 確認閏年
            isLeapYear(year){
                if(year % 4 === 0 || year % 400 === 0){
                    return true
                } else {
                    return false
                }
            },
            // 取得每月天數
            getMonthDaysNumber(){
                let month = this.monthSelect;
                let year = this.yearSelect;
                let is_leap_year = this.isLeapYear(year);
                let month_days = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];

                if(month === '2' && is_leap_year){
                    return 29
                } else if (month === '2' && !is_leap_year){
                    return month_days[1]
                } else {
                    return month_days[month - 1]
                }
            },
            refreshDate() {
                let total_days = this.getMonthDaysNumber();
                let week_day = this.weekDayStart();
                let month = this.monthSelect;
                let month_days = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
                let last_month = month_days[month -2];
                month - 2 < 0 ? last_month = month_days[11] : '';

                let days_array = [];
                for(let i = 0; i < week_day; i++){
                    let last_month_date = {
                        date: last_month - week_day + 1,
                        status: 'past'
                    };
                    days_array.push(last_month_date);
                    last_month++
                }

                for(let j = 1; j <= total_days; j++){
                    let data = {
                        date: j,
                        status: this.keyDate.includes(j) ? 'key' : ''
                    };
                    days_array.push(data)
                }

                this.days = days_array;

                if(total_days > 30 && week_day > 4 || total_days > 29 && week_day > 5){
                    this.autoIncreaseHeight = true
                } else {
                    this.autoIncreaseHeight = false
                }
            },
            monthSelectUpdate(v){
                this.monthSelect = this.monthSelect + v;
                if(this.monthSelect === 13){
                    this.monthSelect = 1;
                    this.yearSelect++
                } else if(this.monthSelect === 0){
                    this.monthSelect = 12;
                    this.yearSelect--
                }
            },
            yearSelectUpdate(year){
                this.yearSelect = year
            },
            toggleYearSelect(){
                this.showYearSelect = !this.showYearSelect
            },
            buildYearSelectList(){
                let current_year = new Date().getFullYear();
                // 可檢視總共三年
                this.yearSelections = [current_year, current_year-1, current_year-2];
            }
        }
    }
</script>

<style lang='sass' scoped>
    .week-txt-row
        display: flex
        flex-wrap: wrap
        margin-top: 10px
        width: 100%
    .week-txt
        width: 14.28%
        text-align: center
        font-size: 16px
    .calendar-block
        width: 470px
        min-height: 300px
        margin: 16px 25px 0 0
        padding: 0 23px 0 0
        border-radius: 5px
        box-shadow: 0 2px 10px 0 rgba(0, 0, 0, 0.04)
        background-color: #ffffff
        color: #2b579b
        display: flex
        justify-content: space-between
    .date-block
        color: #7c8bac
        width: 14.28%
        height: 27px
        text-align: center
        margin-top: 9px
        font-family: NotoSansTC
        font-size: 16px
        position: relative
        display: flex
        justify-content: center
        align-items: center
        &:hover
            .date
                background-color: rgba(43, 87, 155, 0.1)
                border-radius: 50%
    .date
        width: 27px
        height: 27px
        display: flex
        justify-content: center
        align-items: center
    .month-select-block
        font-family: HelveticaNeue
        font-size: 16px
        font-weight: bold
        font-stretch: normal
        font-style: normal
        line-height: normal
        letter-spacing: normal
        text-align: center
        color: #2b579b
    .year-select-block
        width: 100px
        margin: 0 23px 0 0
        padding: 16px 19px 169px 20px
        border-radius: 5px 0 0 5px
        background-color: #2b579b
    .year-select-txt
        height: 26px
        font-family: NotoSansTC
        font-size: 18px
        font-weight: bold
        color: #ffffff
        height: 60px
        cursor: pointer
        /*display: flex*/
        /*align-items: center*/
    .date-display-txt
        width: 61px
        height: 52px
        font-family: NotoSansTC
        font-size: 18px
        font-weight: bold
        letter-spacing: normal
        color: #ffffff
        display: flex
        flex-direction: column
        justify-content: space-between
    .key-date-color
        font-size: 16px
        font-weight: bold
        cursor: pointer
        color: #2b579b
    .past-date-color
        color: #d4d4d4
    .calendar-right-side
        position: relative
    .month-year-display-block
        height: 60px
        display: flex
        justify-content: space-around
        flex-wrap: wrap
        align-items: center
    .month-btn
        position: absolute
        /*width: 7px*/
        /*height: 12px*/
        font-size: 15px
        cursor: pointer
        color: #d4d4d4
    .left-btn
        left: 15px
        transform: rotate(90deg)
    .right-btn
        right: 20px
        transform: rotate(-90deg)
    .current-date-color
        background-color: #2b579b
        color: white
        border-radius: 50%
    .year-option
        margin: 5px 0
        user-select: none
        &:hover
            opacity: .2
            cursor: pointer
    .fade-enter-active
        transition: all 1s ease
    .fade-leave-active
        transition: all .1s ease
    .fade-enter
        transform: translateX(50px)
        opacity: 0
    .fade-leave-to
        transform: translateX(-50px)
        opacity: 0
    .increase-height
        padding-bottom: 25px
    .year-select-modal
        font-size: 12px
        background-color: rgba(0,0,0,.1)
        color: white
        text-align: center
        padding: 5px 15px
        background-color: #7c8bac
        border-radius: 5px
        top: 70px
        position: absolute

    @media screen and (max-width: 992px)
        .calendar-block
            width: 452px
            box-sizing: border-box
        .fade-enter
            transform: translateX(10px)
            opacity: 0
        .fade-leave-to
            transform: translateX(-10px)
            opacity: 0


    @media screen and (max-width: 768px)
        .calendar-block
            flex-direction: column
            justify-content: flex-start
            width: 310px
            min-height: 330px
            padding: 0
        .year-select-block
            width: 310px
            height: 50px
            margin: 0 0 20px
            padding: 15px 30px
            box-sizing: border-box
            display: flex
            justify-content: space-between
            border-radius: 5px 5px 0 0
        .month-year-display-block
            height: 20px
            margin-bottom: 20px
        .date-display-txt
            display: flex
            font-size: 16px
            flex-direction: row
        .year-select-txt, .date-display-txt
            flex: 1
        .date-display-txt
            justify-content: flex-end


</style>
