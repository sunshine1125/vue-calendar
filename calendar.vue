<template>
    <div class="calendar">
        <input type="text" class="control-height" v-model="defaultDay" @click="toggle">
        <div v-show="show"  id="calendar">
            <div class="change">
                <button class="btn btn-default btn-xs left" @click="prev">prev</button>
                <div class="select">
                    <span>{{ currentYear }}</span>/<span>{{ currentMonth }}</span>
                </div>
                <button class="btn btn-default btn-xs right" @click="next">next</button>
            </div>
            <div class="content">
                <table>
                    <tr>
                        <th v-for="option in Week">{{ option }}</th>
                    </tr>
                </table>
                <ul>
                    <li v-for="day in lastdays"><span></span></li>
                    <li class="current" v-on:click="pick(day)" v-for ="day in days">
                        <span v-if="day.flag" class="active">{{ day.mm.getDate() }}</span>
                        <span v-else>{{ day.mm.getDate() }}</span>
                    </li>
                </ul>
            </div>
            <div class="clear">
              <button class="btn btn-default btn-xs left" @click="clearall">清空</button>
              <button class="btn btn-default btn-xs center" @click="today">今天</button>
    		  <button class="btn btn-default btn-xs right" @click="close">关闭</button>
          </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'calendar',
    props:['type'],
    data() {
        var date = new Date();
        return {
           show:false,
           Week:['周一','周二','周三','周四','周五','周六','周日'],
           currentDay:1,
           currentMonth:date.getMonth()+1,
           currentYear:date.getFullYear(),
           currentWeek: date.getDay(),
           lastdays:[],
           days:[],
           defaultDay:date.toLocaleDateString(),
           isCurrent:false,
       }
    },
    created:function(){
        this.initData(null);
    },
    methods: {
        toggle(){
            this.show = !this.show;
        },
        initData:function(cur){
            var date;
            if(cur){
                date = new Date(cur);
            }else{
                date = new Date(this.currentYear,this.currentMonth-1,this.currentDay);
            }
            this.currentDay = date.getDate();
            this.currentYear = date.getFullYear();
            this.currentMonth = date.getMonth()+1;
            this.currentWeek = date.getDay();
            if (this.currentWeek == 0) {
                this.currentWeek = 7;//把周日换为7
            }
            var str = this.formatDate(this.currentYear , this.currentMonth, this.currentDay);
            this.days.length = 0;
            this.lastdays.length = 0;
            //上个月的几天
            for (var i = this.currentWeek-1; i > 0; i--) {
                var d = new Date(str);
                d.setDate(d.getDate()-i);
                this.lastdays.push(d);
            }
            var m1 = this.lastdays.length;//上个月的天数
            //当前月
            var str2 = new Date(this.currentYear , this.currentMonth, 0);
            var m2 = str2.getDate();//当前月的天数
            var n = m1+m2;
            for(var i = 0;i <= n-this.currentWeek;i++){
                var d = new Date(str);
                d.setDate(d.getDate()+i);
                var date = new Date();
                this.isCurrent = date.getFullYear() == d.getFullYear() && date.getMonth() == d.getMonth() && date.getDate() == d.getDate();
                var obj = {
                    mm:d,
                    flag:this.isCurrent
                }
                this.days.push(obj);
            }
        },
        prev:function() {
            // setDate(0); 上月最后一天
            // setDate(-1); 上月倒数第二天
            // setDate(dx) 参数dx为 上月最后一天的前后dx天
            var d = new Date(this.formatDate(this.currentYear , this.currentMonth , 1));
            d.setDate(0);
            this.initData(this.formatDate(d.getFullYear(),d.getMonth() + 1,1));
       },
       //选择日期
       pick:function(date){
           var chooseDay = this.formatDate( date.mm.getFullYear() , date.mm.getMonth() + 1, date.mm.getDate());
           this.defaultDay = chooseDay;
           this.show = false;
           var day = date.mm.getDay();
           this.currentDay = day;
           this.$emit('hasPick',this.defaultDay,this.type,this.currentDay);

       },
       next:function(){
           var d = new Date(this.formatDate(this.currentYear,this.currentMonth,1));
           d.setDate(42);
           this.initData(this.formatDate(d.getFullYear(),d.getMonth() + 1,1));
       },
       clearall(){
           this.defaultDay = '';
       },
       today(){
           var date = new Date();
           var chooseDay = this.formatDate( date.getFullYear() , date.getMonth() + 1, date.getDate());
           this.initData(this.formatDate( date.getFullYear() , date.getMonth() + 1, 1));
           this.defaultDay = chooseDay;
           this.show = false;
           var day = date.getDay();
           this.currentDay = day;
           this.$emit('hasPick',this.defaultDay,this.type,this.currentDay);
      },
      close(){
          this.show = false;
      },

      // 返回 类似 2011-01-21 格式的字符串
      formatDate: function(year,month,day){
           var y = year;
           var m = month;
           if(m<10) m = "0" + m;
           var d = day;
           if(d<10) d = "0" + d;
           return y+"-"+m+"-"+d
      }
    }
}
</script>

<style lang="css" scoped>
    .calendar{
        height: 20px;
        width: 150px;
        position: relative;
    }
    #calendar{
        width:220px;
        height:224px;
        border:1px solid #999;
        position: absolute;
        left:0;
        top:20px;
        z-index: 1000;
        background-color:#fff;
    }
    .change{
        height:10%;
        width:100%;
    }
    .change .select{
        float:left;
        line-height: 22px;
    }
    .content{
        border-top: 1px solid #999;
        font-size: 10px;
        height: 79%;
        width:100%;
    }
    table{
        height:16%;
        width:100%;
        border-collapse:collapse;
    }
    ul{
        height: 84%;
        width: 100%;
    }
    .clear{
        height:11%;
        width:100%;
        position: relative;
        border-top: 1px solid #999;
    }
    .left{
        float:left;
        margin-right:50px;
        margin-left: 5px;
    }

    .right{
        float:right;
        margin-right: 5px;
    }
    th{
        text-align:center;
    }
    li{
        line-height:23px;
        list-style: none;
        float: left;
        width: 14.2%;
        text-align: center;
        height: 16.6%;
    }
    li .active{
        color: #00B8EC;
        display: block;
        width: 100%;
        height: 100%;
    }
    li .active:hover{
        color: #fff;
    }
    li .other-month{
        color: gainsboro;
    }
    ul .current:hover{
        background: #00B8EC;
        display: block;
        color:#fff;
    }
</style>
