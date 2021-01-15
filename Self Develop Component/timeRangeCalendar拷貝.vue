<template>
    <div>
        <div class="calendar-block">
            <div class="year-select-block">
                <div class="year-select-txt" @click="toggleYearSelect">
                    <input type="text" style="cursor: pointer; width: 100%; height: 100%; border: none; background-color: transparent; position: absolute; outline: none; color: transparent;
" @blur="closeMenu()">
                    {{yearSelect}}
                    <span style="font-size: 12px; cursor: pointer">
                        <i class="fas fa-chevron-down"></i>
                    </span>
                </div>
                <div v-show='showYearSelect' class='year-select-modal'>
                    <div
                        v-for='item in yearSelections' :key='item'
                        class="year-option"
                        @click="yearSelectUpdate(item)"
                    >
                        {{item}}
                    </div>
                </div>
                <div class="date-display-txt">
                    <div>{{weekTxt[new Date(this.yearSelect, this.monthSelect -1).getDay()]}},</div>
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
                            :key="index"
                            class='date-block'
                            :class="[
                                day.status === 'key' ? 'key-date-color' : '', // 標註粗體
                                day.status === 'past' ? 'past-date-color' : '', // 上個月的日期
                                day.status === 'select' ? 'select-date-color' : '',
                                day.status === 'select-start' ? 'select-date-color select-start' : '',
                                day.status === 'select-end' ? 'select-date-color select-end' : '',
                                day.picking_range ? 'picking-range-bgc' : '',
                            ]"
                            @click="selectCurrentDate(index)"
                            @mouseover="pickingRange(index)"
                        >
                            <div class="date select">{{day.date}}</div>
                        </div>
                    </div>
                </transition>
                <div class="button-block">
                    <button class="button">Cancel</button>
                    <button class="button" @click="dateRangeConfirm">OK</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: 'calendarDisplay',
        props: {
            defaultDate: {
                type: Array
            }
        },
        data(){
            return {
                weekTxt: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],
                monthSelect: new Date().getMonth() + 1,
                yearSelect: new Date().getFullYear(),
                days: '',
                keyDate: [], // 需標註粗體的日期 ex: [1, 5, 31]
                currentDate: new Date().getDate(),
                showYearSelect: false,
                yearSelections: [],
                showDate: true,
                autoIncreaseHeight: false,
                startSelecting: false, // 是否開始選擇
                selectDateStart: '', // 一開始選擇的日期的 index
                selectDateEnd: '',
                selectDateStartTimeData: {
                    year: '',
                    month: '',
                    date: ''
                },
                selectDateEndTimeData: {
                    year: '',
                    month: '',
                    date: ''
                },
                selectDone: false,
                start: '',
                end: ''
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
            this.buildYearSelections()
        },
        watch: {
            defaultDate: {
                handler(data){
                    if(data.length === 2){
                        this.start = data[0];
                        this.end = data[1];
                        this.selectDateStartTimeData.year = parseInt(this.start.split('-')[0]);
                        this.selectDateStartTimeData.month = parseInt(this.start.split('-')[1]);
                        this.selectDateStartTimeData.date = parseInt(this.start.split('-')[2]);
                        this.selectDateEndTimeData.year = parseInt(this.end.split('-')[0]);
                        this.selectDateEndTimeData.month = parseInt(this.end.split('-')[1]);
                        this.selectDateEndTimeData.date = parseInt(this.end.split('-')[2]);
                        this.refreshDate();
                    }
                },
                immediate: true
            },
            monthSelect(month){
                new Promise(resolve => {
                    this.showDate = false;
                    resolve()
                })
                    .then(() => {
                        this.showDate = true;
                        this.refreshDate();

                        if(this.selectDateStartTimeData.year < this.yearSelect){ // 過去
                            this.selectDateStart = -1
                        } else {
                            if(month > this.selectDateStartTimeData.month){
                                this.selectDateStart = -1
                            } else {
                                this.selectDateStart = this.days.length
                            }
                        }
                    })
            },
            yearSelect(year){
                this.showYearSelect = false;
                new Promise(resolve => {
                    this.showDate = false;
                    resolve()
                })
                    .then(() => {
                        this.showDate = true;
                        this.refreshDate();

                        if(year < this.selectDateStartTimeData.year){
                            this.selectDateStart = this.days.length
                        } else {
                            if(this.monthSelect > this.selectDateStartTimeData.month){
                                this.selectDateStart = -1
                            } else {
                                this.selectDateStart = this.days.length
                            }
                        }
                    })
            },
        },
        methods: {
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
            refreshDate(){
                let total_days = this.getMonthDaysNumber();
                let week_day = this.weekDayStart();
                let year = this.yearSelect;
                let month = this.monthSelect;
                let month_days = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
                let last_month_days = month_days[month -2];
                let days_array = [];
                month - 2 < 0 ? last_month_days = month_days[11] : '';

                if(this.selectDateStartTimeData.date !== '' && this.selectDateEndTimeData.date !== ''){
                    let start_year, end_year, start_month, end_month, start_date, end_date;
                    let start_data = this.selectDateStartTimeData.year + '-' + this.selectDateStartTimeData.month + '-' + this.selectDateStartTimeData.date;
                    let end_data = this.selectDateEndTimeData.year + '-' + this.selectDateEndTimeData.month + '-' + this.selectDateEndTimeData.date;
                    if(new Date(start_data).getTime() < new Date(end_data).getTime()){ // 由前往後
                        start_year = start_data.split('-')[0];
                        start_month = start_data.split('-')[1];
                        start_date = start_data.split('-')[2];
                        end_year = end_data.split('-')[0];
                        end_month = end_data.split('-')[1];
                        end_date = end_data.split('-')[2];
                    } else { // 由後往前
                        start_year = end_data.split('-')[0];
                        start_month = end_data.split('-')[1];
                        start_date = end_data.split('-')[2];
                        end_year = start_data.split('-')[0];
                        end_month = start_data.split('-')[1];
                        end_date = start_data.split('-')[2];
                    }

                    let all_start_time_data = start_year + '-' + start_month + '-' + start_date;
                    let all_end_time_data = end_year + '-' + end_month + '-' + end_date;
                    this.start = all_start_time_data;
                    this.end = all_end_time_data;
                }

                for(let i = 0; i < week_day; i++){
                    let date = last_month_days - week_day + 1;
                    let picking_range = false;
                    let month_copy = month;
                    let year_copy = year;
                    let status = 'past';
                    if(month_copy -1 === 0){
                        month_copy = 12;
                        year_copy--
                    } else {
                        month_copy--
                    }
                    let complete_time = year_copy + '-' + month_copy + '-' + date;
                    if(new Date(this.start).getTime() <= new Date(complete_time).getTime() && new Date(complete_time).getTime() <= new Date(this.end).getTime()){
                        picking_range = true;
                        if(new Date(this.start).getTime() === new Date(complete_time).getTime()){
                            status = 'select-start';
                            picking_range = false
                        }
                        if(new Date(this.end).getTime() === new Date(complete_time).getTime()){
                            status = 'select-end';
                            picking_range = false
                        }
                    }

                    let last_month_date = {
                        date: date, // 每月最後一天也要算 所以 + 1
                        status: status,
                        picking_range: picking_range,
                        last_month: true // 多加一個狀態判斷是不是上個月的日期
                    };
                    days_array.push(last_month_date);
                    last_month_days++
                }

                for(let j = 1; j <= total_days; j++){
                    let date = j;
                    let picking_range = false;
                    let status = this.keyDate.includes(j) ? 'key' : '';
                    let complete_time = year + '-' + month + '-' + date;

                    if(new Date(this.start).getTime() <= new Date(complete_time).getTime() && new Date(complete_time).getTime() <= new Date(this.end).getTime()){
                        picking_range = true;
                        // 開始結束日底色都只有一半 所以class 不一樣，由status區分
                        if(new Date(this.start).getTime() === new Date(complete_time).getTime()){
                            status = 'select-start';
                            picking_range = false
                        }
                        if(new Date(this.end).getTime() === new Date(complete_time).getTime()){
                            status = 'select-end';
                            picking_range = false
                        }
                    } else {
                        picking_range = false
                    }

                    let data = {
                        date: j,
                        status: status,
                        picking_range: picking_range
                    };
                    days_array.push(data)
                }

                this.days = days_array;
                total_days > 30 && week_day > 4 || total_days > 29 && week_day > 5 ? this.autoIncreaseHeight = true : this.autoIncreaseHeight = false;
            },
            monthSelectUpdate(value){
                this.monthSelect = this.monthSelect + value;
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
            selectCurrentDate(index){
                if(this.selectDateStart !== '' && this.startSelecting && this.selectDateEnd === ''){
                    this.selectDateEnd = index;
                    this.startSelecting = false;
                    this.selectDone = true;
                    this.days[index].picking_range = false;
                    this.days[index].status === 'past' && this.monthSelect === 1 ? this.selectDateEndTimeData.year = this.yearSelect - 1 : this.selectDateEndTimeData.year = this.yearSelect;
                    this.days[index].status === 'past' ? this.selectDateEndTimeData.month = this.monthSelect - 1 : this.selectDateEndTimeData.month = this.monthSelect;
                    this.selectDateEndTimeData.month === 0 ? this.selectDateEndTimeData.month = 12 : '';
                    this.days[index].status = 'select';
                    this.selectDateEndTimeData.date = this.days[index].date;
                    this.refreshDate();
                    return
                }

                if(this.selectDateEnd !== ''){
                    this.selectDateStart = '';
                    this.selectDateEnd = '';
                    this.selectDone = false;
                }

                this.days = this.days.map(item => {
                    if(item.status !== 'past'){
                        item.status = '';
                        item.last_month ? item.status = 'past' : ''
                    }
                    item.picking_range = false;
                    return item
                });

                this.resetDateSelect();
                this.startSelecting = true;
                this.selectDateStart = index;
                this.days[index].status === 'past' && this.monthSelect === 1 ? this.selectDateStartTimeData.year = this.yearSelect - 1 : this.selectDateStartTimeData.year = this.yearSelect
                this.days[index].status === 'past' ? this.selectDateStartTimeData.month = this.monthSelect - 1 : this.selectDateStartTimeData.month = this.monthSelect;
                this.selectDateStartTimeData.month === 0 ? this.selectDateStartTimeData.month = 12 : '';
                this.days[index].status = 'select';
                this.selectDateStartTimeData.date = this.days[index].date;
            },
            pickingRange(index){
                if(this.startSelecting){
                    this.days.map((item, item_index) => {
                        if(index > this.selectDateStart){
                            item_index > this.selectDateStart && item_index <= index ? this.days[item_index].picking_range = true : this.days[item_index].picking_range = false
                        } else {
                            item_index < this.selectDateStart && index <= item_index ? this.days[item_index].picking_range = true : this.days[item_index].picking_range = false;
                        }
                    })
                }
            },
            resetDateSelect(){
                this.selectDateStart = ''; // 一開始選擇的日期的 index
                this.selectDateEnd = '';
                this.selectDateStartTimeData = {
                    year: '',
                    month: '',
                    date: ''
                };
                this.selectDateEndTimeData = {
                    year: '',
                    month: '',
                    date: ''
                };
                this.start = '';
                this.end = '';
            },
            dateRangeConfirm(){
                let data = {
                    start: this.start,
                    end: this.end
                };
                this.$emit('input', data)
            },
            buildYearSelections(){
                let current_year = new Date().getFullYear();
                this.yearSelections = [current_year, current_year-1, current_year-2]
            },
            closeMenu(){
                this.showYearSelect = false
            }
        }
    }
</script>

<style lang='sass' scoped>
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
        .week-txt-row
            display: flex
            flex-wrap: wrap
            margin-top: 10px
            width: 100%
            align-items: center
        .week-txt
            width: 14.28%
            text-align: center
            font-size: 16px
        .date-block
            color: #7c8bac
            width: 14.28%
            height: 24px
            text-align: center
            margin-top: 9px
            font-family: NotoSansTC
            font-size: 16px
            position: relative
            display: flex
            justify-content: center
            align-items: center
            .date
                border-radius: 50%
                cursor: pointer

        .date
            width: 32px
            height: 32px
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
            position: relative
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
            font-size: 15px
            cursor: pointer
            color: #d4d4d4
            .fas
                &:hover
                    color: #2b579b
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
        // RANGE
        .select-date-color
            .select
                background-color: #2b579b
                color: white
                border-radius: 50%
                box-shadow: 0 1px 10px 0 rgba(0, 0, 0, 0.3)
        .select-end
            background: linear-gradient(90deg, rgba(69,125,195, .2) 50%, #FFFFFF 50%)
        .select-start
            background: linear-gradient(90deg, #FFFFFF 50%, rgba(69,125,195, .2) 50%)
        .picking-range-bgc
            background-color: rgba(69,125,195, .2)
            color: #2b579b
        .button-block
            padding: 19px
            display: flex
            justify-content: flex-end
            .button
                display: flex
                align-items: center
                justify-content: center
                width: 80px
                padding: 8px 0
                border-radius: 2px
                border: solid 1px rgba(21, 49, 107, 0.1)
                background-color: rgba(0, 0, 0, 0)
                color: #7c8bac
                font-size: 14px
                margin-left: 10px
                &:hover
                    border-color: #7c8bac

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
