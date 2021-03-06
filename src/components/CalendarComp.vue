<template>
<div>
    <div v-if="!detailMode">
        <div class="month">
            <ul>
                <li class="prev" @click="changeMonth(1)">◀</li>
                <li class="next" @click="changeMonth(0)">▶</li>
                <li><span style="font-size:12px">{{contextYear}}년</span> {{contextMonth}}월</li>
            </ul>
        </div>
        <ul class="weekdays">
            <li v-for="day in days" :key="day">{{day}}</li>
        </ul>
        <ul class="dates">
            <li v-for="blank in firstDayOfMonth" :key="blank+100"><br>&nbsp;</li><li v-for="date in daysInMonth" :key="date" :class="{'current-day':(contextYear==initialYear && contextMonth==initialMonth && date==initialDate),'selected-day':(contextYear==selectedYear && contextMonth==selectedMonth && date==selectedDate)}" @click="dateSelectedChange(date)">{{date}}<br><span :class="{'event-day':date in promises}">&nbsp;{{date in promises ? promises[date][0]['title'] : ''}}&nbsp;</span></li>
        </ul>
    </div>
    <div v-if="detailMode" id="detail-section">
        <input class="btn" type="button" @click="detailMode=false" value="◀">
        <p>{{contextYear}}년 {{contextMonth}}월 {{detailDate}}일</p>
        <h2>{{detail[0]['title']}}</h2>
        <p><font-awesome-icon icon="map-marker-alt" />{{detail[0]['place']}}</p>
        <p><font-awesome-icon icon="clock" />{{detail[0]['time']}}에 {{detail[0]['meeting_place']}}에서 모임</p>
        <p><font-awesome-icon icon="user-friends" /><span v-for="count in detail.length-1" :key="count">{{detail[count]['name']}} </span></p>
        <input class="btn" type="button" @click="requestDeleteEvent" value="약속 삭제">
    </div>
</div>
</template>

<script>
import moment from 'moment'

export default {
    props: ['userInfo', 'scheduleMode', 'selectedDate', 'selectedMonth', 'selectedYear'],
    data() {
        return {
            days: ['일', '월', '화', '수', '목', '금', '토'],
            dateInitial: moment(),
            initialYear: 0,
            initialMonth: 0,
            initialDate: 0,
            dateContext: moment(),
            contextYear: 0,
            contextMonth: 0,
            contextDate: 0,
            daysInMonth: 0,
            firstDayOfMonth: 0,
            promises: {},
            detailMode: false,
            detail: {},
            detailDate: ''
        }
    },
    methods: {
        initCalendar() {
            this.initialYear = this.dateInitial.format('Y')
            this.initialMonth = this.dateInitial.format('M')
            this.initialDate = this.dateInitial.format('D')
            this.contextYear = this.dateContext.format('Y')
            this.contextMonth = this.dateContext.format('M')
            this.contextDate = this.dateContext.format('D')
            this.firstDayOfMonth = moment(this.dateContext).subtract((this.contextDate - 1), 'days').weekday()
            this.daysInMonth = this.dateContext.daysInMonth()
            this.requestPromises(this.contextMonth)
        },
        changeMonth(reverse) {
            if (!reverse) {
                this.dateContext = moment(this.dateContext).add(1, 'month')
            } else {
                this.dateContext = moment(this.dateContext).subtract(1, 'month')
            }
            this.initCalendar()
        },
        dateSelectedChange(date) {
            if (this.scheduleMode) {
                this.$emit('change-selected', 'd', date)
                this.$emit('change-selected', 'm', this.contextMonth)
                this.$emit('change-selected', 'y', this.contextYear)
                this.$emit('close')
            } else {
                if (date in this.promises) {
                    this.detail = this.promises[date]
                    this.detailDate = date
                    this.detailMode = true
                }
            }
        },
        requestPromises(cal) {
            this.$http.get(`${this.$server}/calendar/${this.userInfo.id}?month=${this.contextYear}-${this.convertDate(this.contextMonth)}`,{ withCredentials: true }).then(res => {
                console.log(res.data)
                this.promises = res.data
            })
        },
        requestDeleteEvent() {
            if (confirm('정말로 이 약속을 삭제하시겠습니까?')) {
                this.$http.delete(`${this.$server}/calendar/${this.userInfo.id}`, { withCredentials: true }).then(res => {
                    console.log(res.data)
                    this.detailMode = false
                    this.initCalendar()
                })
            }
        },
        convertDate(num) {
            var cnum = String(num)
            if (cnum.length == 1) {
                return `0${cnum}`
            } else {
                return cnum
            }
        }
    },
    watch: {
        scheduleMode(mode) {
            this.detailMode = false
            if (mode) {
                this.$emit('change-selected', 'd', this.initialDate)
                this.$emit('change-selected', 'm', this.contextMonth)
                this.$emit('change-selected', 'y', this.contextYear)
                this.initCalendar()
            } else {
                this.$emit('change-selected', 'y', 0)
                this.initCalendar()
            }
        }
    },
    mounted() {
        this.initCalendar()
    }
}
</script>

<style scoped>
li {
    list-style-type: none;
}

.btn {
    padding: 10px;
    -webkit-appearance: none;
    border: 1px solid #dde;
    border-radius: 4px;
    background: #fff;
    font-size: 1em;
    color: #333;
    cursor: pointer;
}

svg {
    margin-right: 10px;
}

.month {
    padding: 50px 25px;
    width: 100%;
    box-sizing: border-box;
    text-align: center;
}

.month ul {
    margin: 0;
    padding: 0;
}

.month ul li {
    font-size: 20px;
    text-transform: uppercase;
    letter-spacing: 3px;
}

.month .prev, .month .next {
    cursor: pointer;
    user-select: none;
}
.month .prev {
    float: left;
}
.month .next {
    float: right;
}

.weekdays {
    margin: 0;
    padding: 10px 0;
}

.weekdays li {
    display: inline-block;
    width: 14.1%;
    color: #666;
    text-align: center;
    font-family: SpoqaHanSansBold, sans-serif !important;
}

.dates {
    padding: 10px 0;
    margin: 0;
}

.dates li {
    display: inline-block;
    width: 14.1%;
    height: 50px;
    text-align: center;
    font-size: 12px;
    color: #777;
    cursor: pointer
}

.current-day {
    color: red !important;
    font-weight: bolder;
}

.selected-day {
    background: lightgray;
    transform: scale(0.9);
    transition: .5s ease;
}

.event-day {
    background: purple;
    color: #fff;
    padding: 0 3px;
}

#detail-section {
    padding: 50px 30px;
}
#detail-section h2 {
    color: #334;
}
#detail-section p {
    color: #667;
}
</style>
