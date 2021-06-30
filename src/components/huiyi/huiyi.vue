npm install --save @fullcalendar/vue @fullcalendar/daygrid @fullcalendar/timegrid @fullcalendar/moment @fullcalendar/interaction



<script>
import { list } from 'api/personnel/meeting'
import { parseTime } from 'utils/index'
import FullCalendar from '@fullcalendar/vue'
import dayGridPlugin from '@fullcalendar/daygrid'
import timeGridPlulgin from '@fullcalendar/timegrid'
import '@fullcalendar/core/locales/zh-cn' // 支持中文
import momentPlugin from '@fullcalendar/moment'
import interactionPlugin from '@fullcalendar/interaction'
import { isNull, isNotNull } from '@/utils/common'
import * as timeUtils from '@/utils/index'
 
export default {
  components: {
    FullCalendar // make the <FullCalendar> tag available
  },
    data () {
        return {
            tableData: [],
            pageIndex: 0,
            recordSize: 0,
            pageSize: 0,
            primaryId: 0,
            selectDate:"",
            meetingroom:"",
            meetingRoomList:[],
            calendarPlugins: [dayGridPlugin, timeGridPlulgin, momentPlugin, interactionPlugin],
            // 随时判断时间是否合法，通过返回true/false来判断是否能够选择
            handlerSeelctAllow: info => {
                const currentDate = new Date()
                const start = info.start
                const end = info.end
                return (start <= end && start >= currentDate)
            },
            pickerOptions: {
                disabledDate(time) {
                    return time.getTime() < Date.now() - 8.64e7
                }
            },
            newItem:{},
            editItem:{},
            events:[]
        }
    },
    created () {
        // this.findList()
    },
    methods: {
 
        /**
         * 表头格式设置
         */
        columnHeaderHtml(mom){
            let array = ['周日','周一','周二','周三','周四','周五','周六']
            for(let i=0; i<=6; i++){
                if(mom.getDay() === i ){
                    return mom.getFullYear() 
                    + '/' 
                    + ((mom.getMonth()+1) <10 ? ('0'+(mom.getMonth()+1)) : (mom.getMonth()+1)) 
                    + '/' 
                    + (mom.getDate() <10 ? ('0'+mom.getDate()) : mom.getDate()) 
                    + '<br>' + array[i]                 
                }
            }
        },
        
        //初始化会议室列表
        initRoom(meetingRoomList){
            this.meetingRoomList = meetingRoomList
            this.meetingroom = this.meetingRoomList[0].id
            this.findList()
        },
 
        // 点击已预约会议
        eventClick(info){
            let primaryId = info.event.id 
            if(info.event.backgroundColor == 'orange'){
                this.$emit("goEdit", primaryId,this.meetingroom)
            }else {
                this.$emit("goEdit", primaryId,this.meetingroom,true)
            }
        },
 
        handleDateClick(info){
            console.log(info)
        },
        handleSelect(info){
            console.log(info)
        },
 
	    // 检索按钮点击
	    searchButton() {
            this.pageIndex = 0
            this.findList()
        },
	    // 检索列表
        findList() {
            let me = this
            me.events = []
            // this.$refs.myfullCalendar.agenda = 'h:mm{ - h:mm}'
            let calendarApi = this.$refs.myfullCalendar.getApi()
            if(isNotNull(me.selectDate)){
                // 跳转日期
                calendarApi.viewSpecs.timeGridWeek.options.firstDay = this.selectDate.getDay()
                calendarApi.gotoDate(timeUtils.parseTime(this.selectDate, "{y}-{m}-{d}"))
                let ss = this.$refs.myfullCalendar
            }else{
                calendarApi.gotoDate(timeUtils.parseTime(new Date(), "{y}-{m}-{d}"))
            }
            let pageParam = {
                pageIndex: me.pageIndex,
                pageSize: 0,
                meetingRoom: me.meetingroom,
            }
            me.$store.dispatch('setLoadingArea', '#meetingForm')
            list(pageParam).then(data =>{
                me.tableData = data.result
                me.tableData.forEach(t=>{
                    let meetingDate = timeUtils.parseTime(t.meetingDate, "{y}-{m}-{d}")
                    let meetingBeginTime = timeUtils.parseTime(t.meetingBeginTime, "{h}:{i}")
                    let meetingEndTime = timeUtils.parseTime(t.meetingEndTime, "{h}:{i}")
                    let start = meetingDate + " " + meetingBeginTime + ":00"
                    let end = meetingDate + " " + meetingEndTime + ":00"
                    me.events.push({
                        title: (t.reverseUserName || "") + '《' + (t.topic.length > 6 ? (t.topic.substring(6,0) + '...') : t.topic) + '》',
                        start: start,
                        end: end,
                        color: t.isEdit ? 'orange' : '#4286f5',
                        id:t.meetingId
                    })
                })
                me.recordSize = data.pager.recordSize
                me.pageSize = data.pager.pageSize
            })
        },
	    // 时间格式化
        dateFormat(row,column){
            let date = row[column.property]
            if (date == undefined || date == null || date == '') {
                return ""
            }
            return parseTime(date, "{y}-{m}-{d} {h}:{i}:{s}")
        },
	    // 分页点击
        pageChange(val) {
            this.pageIndex = val - 1
            this.findList()
        },
	    
	    // 新增按钮点击
        addButton() {
            if(isNull(this.meetingroom)){
                this.$message({
                    type: 'warning',
                    message: '请先选择会议室'
                })
                return
            }
            this.$emit("goEdit",null,this.meetingroom)
        },
        
    }
}
</script>
页面编写：
<template>
    <div class="app-container">
        <el-row>
            <el-col :span="24">
                <el-form :inline="true">
                    <el-form-item>
                        <el-select placeholder="会议室" v-model="meetingroom" class="input-search-width" filterable @change="findList()">
                            <el-option v-for="item in meetingRoomList" :key="item.id" :label="item.name"
                            :value="item.id"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item>
                        <el-date-picker v-model="selectDate" placeholder="日期" type="date" :picker-options="pickerOptions" clearable class="input-search-width">
                        </el-date-picker>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" icon="el-icon-search" @click="findList()">搜索</el-button>
                    </el-form-item>
                </el-form>
            </el-col>
            <el-col :span="24" class="warp-add">
                <el-button type="primary" icon="el-icon-circle-plus-outline" @click="addButton()">预约</el-button>
            </el-col>
            <el-col :span="24" class="warp-table" style="margin-top:10px">
                <div style="margin-top:-15px;">
                    <full-calendar
                        ref="myfullCalendar"
                        :plugins="calendarPlugins"
                        :all-day-slot="false"
                        :header="{
                            left:'prev',
                            center:'title',
                            right: 'next'
                        }"
                        :slot-event-overlap="false"
                        :events="events"
                        :button-text="{
                            today: '今天'
                        }"
                        :unselect-auto="false"
                        :select-overlap="false"
                        :business-hours="{
                            startTime: '08:00',
                            endTime:'24:00',
                            daysOfWeek: [ 0, 1, 2, 3, 4, 5, 6 ]
                        }"
                        :select-allow="handlerSeelctAllow"
                        :height="670"
                        columnHeaderFormat="dddd YYYY/M/d"
                        :columnHeaderHtml="columnHeaderHtml" 
                        select-mirror="true"
                        :firstDay="(new Date().getDay())"
                        :minDate="(new Date().getDay())"
                        aspectRatio=2
                        min-time="08:00:00"
                        max-time="24:00:00"
                        scrollTime = "08:00:00"
                        selectable="true"
                        slot-duration="00:30"
                        slotLabelInterval="00:30"
                        slot-label-format="HH:mm"
                        title-format="YYYY年MM月D日"
                        default-view="timeGridWeek"
                        locale="zh-cn"
                        @eventClick = "eventClick"
                        @dateClick="handleDateClick"
                        @select="handleSelect"/>
                </div>
            </el-col>
        </el-row>
    </div>
</template>
3.css样式代码：

<style>
    .fc-button-primary {
        color: #FFF !important;
        /* font-size: 18px !important; */
        /* background-color: #4286f5 !important; */
        background-color: #EAF2F8 !important;
        border-color: #EAF2F8 !important;
        border-radius: 100px !important;
        width: 40px !important;
        height: 40px !important;
    }
    .fc-button-primary:focus {
        -webkit-box-shadow: 0 0 0 0.2rem rgba(234, 242, 248) !important;
        box-shadow: 0 0 0 0.2rem rgba(234, 242, 248) !important;
    }
    .fc-button .fc-icon {
        vertical-align: middle;
        font-size: 1.5em;
        color: #1E65D1;
    }
    .fc-left{
        width:37% !important;
        text-align:right !important;
    }
    .fc-center{
        width:26% !important;
        text-align:left !important;
        margin-left:-502px !important;
    }
    .fc-right{
        width:37% !important;
        text-align:left !important;
        margin-left:-595px !important;
    }
    .fc-event {
        position: relative;
        display: block;
        font-size: 0.6em !important;
        line-height: 1.4 !important;
        border-radius: 3px;
        border: 1px solid #3788d8;
        /* right: -5px; */
        right: -5px !important;
    }
    .fc-icon {
        display: inline-block;
        width: 1em !important;
        height: 1em !important;
        text-align: center !important;
        margin-left: -2px !important;
        margin-top: -2px !important;
    }
    .fc-time-grid .fc-slats td {
        height: 1.9em !important;
        border-bottom: 0 !important;
    }
    .fc th, .fc td {
        border-style: solid !important;
        border-width: 1px !important;
        padding: 0 !important;
        vertical-align: top !important;
        height: 30px !important;
        /* margin-top:5px !important; */
    }
    .fc .fc-axis {
        vertical-align: middle !important;
        padding: 0 4px !important;
        white-space: nowrap !important;
        color: #666666 !important;
    }
    .fc th, .fc td .fc-day-header {
        border-style: solid !important;
        border-width: 1px !important;
        padding: 0 !important;
        vertical-align: middle !important;
        color: #333333;
        font-weight: normal !important;
    }
    .fc th, .fc td .fc-widget-content{
        height: 35px !important;
    }
    .fc-toolbar .fc-header-toolbar .fc-center .h2{
        font-size: 14px !important;
    }
    .fc-toolbar.fc-header-toolbar {
        margin-bottom: 1.1em !important;
    }
    .fc-widget-header .fc-today{
        color: #1E65D1 !important;
    }
    .fc-unthemed td.fc-today {
        background: #ffffff !important; }
    .fc .fc-row {
        border-style: solid;
        border-width: 0;
        background-color: #EAF2F8;
    }
    .fc-toolbar h2 {
        font-size: 1.3em !important;
        margin: 0 !important;
        color: #333333 !important;
        font-weight: normal !important;
    }
</style>
 
<style lang="scss" scope>
@import '@fullcalendar/core/main.css';
@import '@fullcalendar/daygrid/main.css';
@import '@fullcalendar/timegrid/main.css';
 
</style>
 

