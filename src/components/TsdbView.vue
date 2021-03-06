<template>
    <div>
        <EntityCascaderView 
            :multiplenable="true" 
            :entity="entity"
            :buckets="bucktets"
            @selected="((data)=>{ onSelectedAttr(data) })" 
            ref="bucketKeys"></EntityCascaderView>
        
        <div style="background: #f2f2f2;padding: 5px 20px 20px 20px;margin-top:20px;">
            <h4>已选指标：
                <el-tag
                    :key="index" v-for="(tag,index) in selectedBuckets"
                    closable
                    style="margin:2px;">
                    {{tag[0]}}/{{tag[1]}}
                </el-tag>
            </h4>
            <div><span style="font-weight:600;">时间区间：</span>
                <el-date-picker
                    v-model="kpi.time"
                    :picker-options="kpi.options"
                    type="datetimerange"
                    value-format="timestamp"
                    range-separator="至"
                    start-placeholder="开始日期"
                    end-placeholder="结束日期"
                    ref="datePicker">
                </el-date-picker>
            </div>

            <grid-layout 
                    :layout.sync="kpi.list"
                     :col-num="layout.colNum"
                     :row-height="40"
                     :is-draggable="layout.draggable"
                     :is-resizable="layout.resizable"
                     :vertical-compact="true"
                     :use-css-transforms="true"
                     style="width:100%;display:flex">

                <grid-item :static="item.static"
                       :x="item.x"
                       :y="item.y"
                       :w="item.w"
                       :h="item.h"
                       :i="item.i"
                        v-for="(item,index) in kpi.list"
                        :key="index"
                        @resize="onGridItemResizeEvent"
                        @move="onGridItemMoveEvent"
                        @resized="onGridItemResizedEvent"
                        @container-resized="onGridItemContainerResizedEvent"
                        @moved="onGridItemMovedEvent"
                        drag-ignore-from=".no-drag"
                        :ref="'item'+item.i">
                    <el-card style="height:100%;">
                        <div slot="header" class="clearfix" style="padding:5px;">
                            <span v-if="item.subKeys != null">
                                {{item.id}} / {{item.bucket}} / <small>{{item.key}}</small>
                                <el-select v-model="item.defaultSubKey" placeholder="请选择子对象" @change="onChange">
                                    <el-option
                                        v-for="key in item.subKeys"
                                        :key="key.value"
                                        :label="key.name"
                                        :value="key.value">
                                    </el-option>
                                </el-select>
                            </span>
                            <span v-else>{{item.id}} / {{item.bucket}} / <small>{{item.key}}</small></span>
                        </div>
                        <ChartView  :model="item" class="no-drag"></ChartView>
                    </el-card>
                    <el-button type="text" icon="el-icon-full-screen" @click="onFullScreen(item.i)" style="position: absolute;top: 10px;right: 30px;font-weight: 900;color: #b2b2b2;"></el-button>
                <el-button type="text" icon="el-icon-close" @click="onRemoveItem(item.i)" style="position: absolute;top: 10px;right: 10px;font-weight: 900;color: #b2b2b2;"></el-button>
                </grid-item>

            </grid-layout>
        
        </div>

    </div>
</template>
<script>
import _ from 'lodash';
import Cookies from 'js-cookie'
import ChartView from './chart/ChartView.vue';
import EntityCascaderView from './EntityCascaderView';
import { GridLayout, GridItem } from "vue-grid-layout";

export default{
    name: "TsdbView",
    props: {
        bucket: Object,
        entity: Array
    },
    components:{
        ChartView,
        EntityCascaderView,
        GridLayout,
        GridItem
    },
    computed:{
        bucktets(){
            try{
                return _.map(this.bucket.children,(v)=>{
                    let children = _.map(v.param,(r,index)=>{
                        return { value: r, title: v['remark'][index], ptype: v['ptype'][index], precision: v['precision'][index], unit: v['unit'][index],subKeys:[] };
                    });
                    return _.extend(v, {value: v.field, title: v.field, children:children, subKeys:v.crc32} );
                })
            }catch(err){
                return [];
            }
        }
    },
    data(){
        return {
            layout: {
                draggable: true,
                resizable: true,
                colNum: 12,
                index: 0
            },
            kpi: {
                list: [],
                time: [this.moment().add(-1,'hour').format('YYYY-MM-DD HH:mm'),this.moment().format('YYYY-MM-DD HH:mm')],
                options: {
                    shortcuts:[
                        {
                            text: '最近15分钟',
                            onClick(picker) {
                                const end = new Date();
                                const start = new Date();
                                start.setTime(start.getTime() - 900 * 1000);
                                picker.$emit('pick', [start, end]);
                            }
                        }, {
                            text: '最近30分钟',
                            onClick(picker) {
                                const end = new Date();
                                const start = new Date();
                                start.setTime(start.getTime() - 1800 * 1000);
                                picker.$emit('pick', [start, end]);
                            }
                        }, {
                            text: '最近一小时',
                            onClick(picker) {
                                const end = new Date();
                                const start = new Date();
                                start.setTime(start.getTime() - 3600 * 1000 * 1);
                                picker.$emit('pick', [start, end]);
                            }
                        }, {
                            text: '最近一天',
                            onClick(picker) {
                                const end = new Date();
                                const start = new Date();
                                start.setTime(start.getTime() - 3600 * 1000 * 24);
                                picker.$emit('pick', [start, end]);
                            }
                        }, {
                            text: '最近七天',
                            onClick(picker) {
                                const end = new Date();
                                const start = new Date();
                                start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
                                picker.$emit('pick', [start, end]);
                            }
                        }, {
                            text: '最近30天',
                            onClick(picker) {
                                const end = new Date();
                                const start = new Date();
                                start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
                                picker.$emit('pick', [start, end]);
                            }
                        }, {
                            text: '最近60天',
                            onClick(picker) {
                                const end = new Date();
                                const start = new Date();
                                start.setTime(start.getTime() - 3600 * 1000 * 24 * 60);
                                picker.$emit('pick', [start, end]);
                            }
                        }, {
                            text: '最近半年',
                            onClick(picker) {
                                const end = new Date();
                                const start = new Date();
                                start.setTime(start.getTime() - 3600 * 1000 * 24 * 180);
                                picker.$emit('pick', [start, end]);
                            }
                        }, {
                            text: '最近1年',
                            onClick(picker) {
                                const end = new Date();
                                const start = new Date();
                                start.setTime(start.getTime() - 3600 * 1000 * 24 * 365);
                                picker.$emit('pick', [start, end]);
                            }
                        }, {
                            text: '最近2年',
                            onClick(picker) {
                                const end = new Date();
                                const start = new Date();
                                start.setTime(start.getTime() - 3600 * 1000 * 24 * 365 * 2);
                                picker.$emit('pick', [start, end]);
                            }
                        }, {
                            text: '最近5年',
                            onClick(picker) {
                                const end = new Date();
                                const start = new Date();
                                start.setTime(start.getTime() - 3600 * 1000 * 24 * 365 * 5);
                                picker.$emit('pick', [start, end]);
                            }
                        }
                ]}
            },
            selectedBuckets:null
        }
    },
    filters:{
        formatTimeRegion(val){
            return `开始：${ new Date(val[0]).toLocaleString() } 结束：${ new Date(val[1]).toLocaleString() }`;
        }
    },
    watch: {
        'kpi.time'(val){
            _.forEach(this.kpi.list,(v)=>{
                this.$set(v,'time',val);
            })
            
            Cookies.set('m3performance-time',val);
        }
    },
    created(){
        this.kpi.time = JSON.parse(Cookies.get('m3performance-time'));
    },
    methods: {
        onShowDatePicker(){
            this.$refs.datePicker.focus();
        },
        onChange(val){
            _.forEach(this.kpi.list,(v)=>{
                this.$set(v,'defaultSubKey',val);
            })
        },
        onSelectedAttr(val){
            
            this.selectedBuckets = val.bucketKeys;
            this.kpi.list = [];
            _.forEach(this.entity,e=>{
                
                _.forEach(val.bucketKeys,(v)=>{
                    
                    let subKeys = this.entity.crc32;
                    let defaultSubKey = "";
                    
                    // let selectedBucket = _.find(val.options,{value:v[0]}).subKeys;
                    
                    // if(!_.isEmpty(selectedBucket)){
                    //     subKeys = selectedBucket;
                    //     defaultSubKey = _.head(subKeys).value;
                    // }
                    let layout = {
                                    x: ( this.kpi.list.length * 6) % (this.layout.colNum || 12),
                                    y: this.kpi.list.length + (this.layout.colNum || 12), // puts it at the bottom
                                    w: 6,
                                    h: 6,
                                    i: this.layout.index,
                                };
                    this.layout.index++;
                    this.kpi.list.push( _.extend(layout,{id: e.id, class: e.class, bucket: v[0], key: v[1], time: this.kpi.time, subKeys: subKeys, defaultSubKey: defaultSubKey}) );
                })
            })
        },
        onFullScreen(val){
            this.m3.fullScreenByEl(this.$refs['item'+val][0].$el);
        },
        onRemoveItem(val) {
            const index = this.kpi.list.map(item => item.i).indexOf(val);
            this.kpi.list.splice(index, 1);
        },
        onGridItemResizeEvent(){
            this.eventHub.$emit("WINDOW-RESIZE-EVENT");
        },
        onGridItemMoveEvent(){
            this.eventHub.$emit("WINDOW-RESIZE-EVENT");
        },
        onGridItemResizedEvent(){
            this.eventHub.$emit("WINDOW-RESIZE-EVENT");
        },
        onGridItemContainerResizedEvent(){
            this.eventHub.$emit("WINDOW-RESIZE-EVENT");
        },
        onGridItemMovedEvent(){
            this.eventHub.$emit("WINDOW-RESIZE-EVENT");
        }
    }
}
</script>


<style scoped>
   

   .columns {
    -moz-columns: 120px;
    -webkit-columns: 120px;
    columns: 120px;
}
/*************************************/
.remove {
    position: absolute;
    right: 5px;
    top: 0;
    padding: 5px;
    cursor: pointer;
}
.vue-grid-layout {
    background: #eee;
}
.vue-grid-item:not(.vue-grid-placeholder) {
    box-shadow: 0 2px 4px rgba(0, 0, 0, .12), 0 0 6px rgba(0, 0, 0, .04);
}
.vue-grid-item .resizing {
    opacity: 0.9;
}
.vue-grid-item .static {
    background: #cce;
}
.vue-grid-item .text {
    font-size: 24px;
    text-align: center;
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    margin: auto;
    height: 100%;
    width: 100%;
}
.vue-grid-item .no-drag {
    height: 100%;
    width: 100%;
}
.vue-grid-item .minMax {
    font-size: 12px;
}
.vue-grid-item .add {
    cursor: pointer;
}
.vue-draggable-handle {
    position: absolute;
    width: 20px;
    height: 20px;
    top: 0;
    left: 0;
    background: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='10' height='10'><circle cx='5' cy='5' r='5' fill='#999999'/></svg>") no-repeat;
    background-position: bottom right;
    padding: 0 8px 8px 0;
    background-repeat: no-repeat;
    background-origin: content-box;
    box-sizing: border-box;
    cursor: pointer;
}
</style>