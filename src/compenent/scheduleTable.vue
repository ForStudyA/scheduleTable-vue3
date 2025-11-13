首列时间格式不标准，边框有重叠

<template>
    <table>
        <tr>
            <th class="firstcol">星期<br>时间</th>
            <th v-for="day in daylist" class="firstrow">{{ day }}</th>  
        </tr>
        <tr>
            <td>
                <table v-for="hour in 24" class="time">
                    <tr class="firstcolData"><td>{{ hour-1 }}:00</td></tr>
                </table>
            </td>
            <td v-for="(dayScheduleData, DayIndex) in allScheduleData.data" class="scheduleData">
                <div 
                    v-for="(ScheduleData, dataIndex) in dayScheduleData"
                    :style="getScheduleStyle(ScheduleData)"
                    class="schedule-item"
                    @contextmenu.prevent="showMyMenu($event, ScheduleData, DayIndex, dataIndex)"
                >
                {{ ScheduleData.title }}
                </div>
            </td>
        </tr>
    </table>

    <!-- 右键菜单 -->
    <div 
        v-if="showMenu"
        class="menu"
        :style="{position: 'absolute',left: menuPosition.x + 'px', top: menuPosition.y + 'px'}"
    >
        <div class="MenuDetail">
            <div @click="showDetail()">查看详细信息</div>
            <div @click="changeDetail()">修改信息</div>
            <div @click="deleteScheduleData()">删除日程</div>
            <div @click="showGetInput = true;">添加日程</div>
        </div>
    </div>

    <!-- 展示详细信息 -->
    <div v-if="showDetailData" 
        :style="{position: 'absolute',left: '372.5px', top: '100px', 'background-color': 'lightblue', padding: '10px 40px 30px 40px'}"
        @click="closeDetailData()"
    >
        <h2 :style="{display: 'inline-block'}">详细信息</h2><span :style="{display: 'inline-block'}">(点击以关闭)</span>
        <h3>标题：{{ seletedScheduleData.title }}</h3>
        <h3>星期：{{ daylist[seletedScheduleData.week-1] }}</h3>
        <h3>开始时间：{{ seletedScheduleData.startHour }}: {{ formattedStartHour }}</h3>
        <h3>结束时间：{{ seletedScheduleData.endHour }}: {{ formattedEndHour }}</h3>
        <div :style="{width: '160px', 'white-space': 'pre-wrap', 'word-wrap': 'break-word'}">详细信息：{{ seletedScheduleData.detail }}</div>
    </div>

    <br>
    <button @click="showGetInput = true;">添加事务</button>
    <!-- <input type="text" id="title" placeholder=""> -->

    <!-- 请求输入 -->
    <div v-if="showGetInput" class="getInput">
        <form @submit.prevent="addSchedule">
            <div>
                <label for="title"></label>
                标题:
                <input 
                    id="title"
                    type="text"
                    v-model="buffer.title"
                    required
                    placeholder="请输入标题"
                >
            </div>
            <div>
                <label for="week">星期:</label>
                <select v-model="buffer.week" id="week">
                    <option v-for="day in daylist">{{ day }}</option>
                </select>
            </div>
            <div>
                <label for="startHour"></label>
                开始时间:
                <input 
                    :style="{width: '30px'}"
                    id="startHour"
                    type="text"
                    v-model="buffer.startHour"
                    required
                    placeholder='小时'
                >
                : 
                <input 
                    :style="{width: '30px'}"
                    id="startMinute"
                    type="text"
                    v-model="buffer.startMinute"
                    placeholder='分钟'
                >
                <label for="endHour"></label>
                结束时间:
                <input 
                    :style="{width: '30px'}"
                    id="endHour"
                    type="text"
                    v-model="buffer.endHour"
                    required
                    placeholder='小时'
                >
                : 
                <input 
                    :style="{width: '30px'}"
                    id="endMinute"
                    type="text"
                    v-model="buffer.endMinute"
                    placeholder='分钟'
                >
            </div>
            <div>
                <label for="detdil"></label>
                详细信息:
                <input 
                    id="datail"
                    type="text"
                    v-model="buffer.detail"
                    placeholder="详细信息"
                >
            </div>
            <div :style="{'text-align': 'center'}">
                <button @click="cancelSubmit()" class="getInputButton">取消</button>
                <button type="submit" class="getInputButton">确认</button>
            </div>
        </form>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, handleError } from 'vue';
    // 数据
    let menuPosition = ref({x: 200, y: 100})
    let seletedScheduleData = ref();
    let seletedDayIndex = ref();
    let seletedDataIndex = ref();

    let showMenu = ref(false);
    let showDetailData = ref(false);
    let showGetInput = ref(false);

    let changeflag = ref(false);

    let daylist = ['星期一', '星期二', '星期三', '星期四', '星期五', '星期六', '星期日']

    let allScheduleData = ref({data:[[{
        title: "测试数据", 
        startHour: 5, 
        startMinute: 0, 
        endHour: 7, 
        endMinute: 0,
        week: 1,
        detail: "详细信息1123132123123123132132123213"
    }], [{
        title: "测试数据", 
        startHour: 11, 
        startMinute: 0, 
        endHour: 14, 
        endMinute: 0,
        week: 2,
        detail: "详细信息2"}], [], [], [], [], []]})

    let buffer = ref({
        title: '',
        week: '',
        startHour: '',
        startMinute: '',
        endHour: '',
        endMinute: '',
        detail: ''
    })

    // 方法
    function showMyMenu(event: { pageX: any; pageY: any; }, ScheduleData: {
        title: string, 
        startHour: number, 
        startMinute: number, 
        endHour: number, 
        endMinute: number,
        week: number,
    }, DayIndex: any, dataIndex: any){
        showMenu.value = true;
        showDetailData.value = false;
        menuPosition.value = {
            x: event.pageX,
            y: event.pageY
        }
        seletedScheduleData.value = ScheduleData;
        seletedDayIndex.value = DayIndex;
        seletedDataIndex.value = dataIndex;

        // 点击其他位置关闭页面
        document.addEventListener('click', ()=>{showMenu.value=false}, {once: true})
    }

    function showDetail(){
        console.log("展示详细信息")
        showDetailData.value = true;
    }

    function changeDetail(){
        showGetInput.value = true;
        buffer.value = JSON.parse(JSON.stringify(seletedScheduleData.value)); // 深拷贝
        buffer.value.week = daylist[seletedScheduleData.value.week - 1] as string;
        changeflag.value = true;
    }

    function closeDetailData(){
        showDetailData.value = false;
    }

    function deleteScheduleData(){
        allScheduleData.value.data[seletedDayIndex.value].splice(seletedDataIndex.value, 1);
    }

    function addSchedule(){
        if(!(Number(buffer.value.startHour)>=0 && Number(buffer.value.startHour)<=23 &&
            Number(buffer.value.startMinute || 0)>=0 && Number(buffer.value.startMinute || 0)<=59 &&
            Number(buffer.value.endHour)>=0 && Number(buffer.value.endHour)<=23 &&
            Number(buffer.value.endMinute || 0)>=0 && Number(buffer.value.endMinute || 0)<=59 &&
            (Number(buffer.value.startHour)<Number(buffer.value.endHour) ||
            (Number(buffer.value.startHour)==Number(buffer.value.endHour) && 
            Number(buffer.value.startMinute || 0)<Number(buffer.value.endMinute || 0)))
        )
        ){
            alert("请确保输入有效")
            return;
        }
        if(changeflag){
            deleteScheduleData()
        }
        allScheduleData.value.data[daylist.indexOf(buffer.value.week)].push({
            title: buffer.value.title, 
            startHour: Number(buffer.value.startHour), 
            startMinute: Number(buffer.value.startMinute || 0), 
            endHour: Number(buffer.value.endHour), 
            endMinute: Number(buffer.value.endMinute || 0),
            week: daylist.indexOf(buffer.value.week)+1,
            detail: buffer.value.detail,
        })
        console.log(allScheduleData);
        showGetInput.value = false;
        buffer.value = {
            title: '',
            week: '',
            startHour: '',
            startMinute: '',
            endHour: '',
            endMinute: '',
            detail: ''
        }
    }

    function cancelSubmit(){
        showGetInput.value = false;
        buffer.value = {
            title: '',
            week: '',
            startHour: '',
            startMinute: '',
            endHour: '',
            endMinute: '',
            detail: ''
        }
    }

    // CSS相关
    let colwidth = 100;
    let borderwidth = 1;
    let firstcolwidth = 45;
    const cssColWidth = colwidth + "px";
    const cssScheduleDataWidth = colwidth + 2 + 'px';    // 没明白为什么+2px才能刚好对齐
    const cssBorderWidth = borderwidth + 'px';
    const cssFirstColWidth = firstcolwidth + "px";
    const cssFirstColDataWidth = firstcolwidth + 2 + "px";

    const formattedEndHour = computed(()=>{
        return seletedScheduleData.value.endMinute <= 9 ? seletedScheduleData.value.endMinute + '0' : seletedScheduleData.value.endMinute
    });
    const formattedStartHour = computed(()=>{
        return seletedScheduleData.value.startMinute <= 9 ? seletedScheduleData.value.startMinute + '0' : seletedScheduleData.value.endMinute
    });

    function getScheduleStyle(time: {
        startHour: number, 
        startMinute: number, 
        endHour: number, 
        endMinute: number
    }){
        let startHour = Number(time.startHour);
        let startMinute = Number(time.startMinute);
        let endHour = Number(time.endHour);
        let endMinute = Number(time.endMinute);

        let topPersent = computed(()=>{
            return (startHour*3600 + startMinute*60)/864;
        })
        let hightPersent = computed(()=>{
            return (endHour*3600 + endMinute*60 - startHour*3600 - startMinute*60)/864;
        })
        console.log(topPersent.value+'%', hightPersent.value + '%');
        return{
            top: topPersent.value+'%',      
            height: hightPersent.value+'%',
            width: cssScheduleDataWidth,
            position: "absolute" as const,    // 断言为字面量类型
            // flex: 'justify-content',
        }
    }
</script>

<style scoped>
    .MenuDetail{
        background-color: cornflowerblue;
        text-align: center;
    }

    .firstrow{
        width: v-bind(cssColWidth);
        background-color: rgb(241, 140, 57);
        /* border-collapse: collapse; */
        /* display: inline-block; */
        position: sticky;
        border: solid black v-bind(cssBorderWidth);
    }
    .firstcol{
        width: v-bind(cssFirstColWidth);
        background-color: rgb(241, 140, 57);
        text-align: center;
        /* outline: solid black v-bind(cssBorderWidth); */
        border-collapse: collapse;
    }
    
    .time{
        width: v-bind(cssFirstColDataWidth);
        background-color: rgb(241, 140, 57);
        text-align: center;
        /* border: solid black v-bind(cssBorderWidth); */
        border-collapse: collapse;
    }

    .firstcolData{
        width: v-bind(cssFirstColDataWidth);
        border: solid black v-bind(cssBorderWidth);
        border-collapse: collapse;
    }

    table{
        border: solid black v-bind(cssBorderWidth);
        border-collapse: collapse;
    }

    td{
        padding: 0px;
        position: relative;
        box-sizing: border-box;
    }

    .scheduleData{
        box-sizing: border-box;
        border: solid v-bind(cssBorderWidth);
        margin: 0;
        padding: 0;
        display: flexbox;
        flex-direction: column;
        background-color: rgb(228, 226, 127);
    }

    .schedule-item{
        background-color: rgb(76, 203, 74);
        display: flex;
        flex: 1;
        align-items: center;
        justify-content: center;
    }

    .getInput{
        top: 20%;
        left: 50%;
        position: absolute;
        display: block;
        background-color: rgb(218, 236, 19);
        transform: translateX(-50%);
        padding: 20px 40px 10px 40px;
    }

    .getInputButton{
        margin: 10px;
    }
</style>