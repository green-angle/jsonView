<template>
  <div class="bgView">
    <div :class="['json-view', length ? 'closeable' : '']" :style="'font-size:' + fontSize+'px'">
    <span @click="toggleClose" :class="['angle', innerclosed ? 'closed' : '']" v-if="length">
    </span>
      <div class="content-wrap">
        <p class="first-line">
          <span v-if="jsonKey" class="json-key">"{{jsonKey}}": </span>
          <span v-if="length">
           {{prefix}}
           {{innerclosed ? ('...' + subfix) : ''}}
           <span class="json-note">
            {{innerclosed ? (' // count: ' + length) : ''}}
           </span>
         </span>
           <span v-if="!length">{{isArray ? '[]' : '{}'}}</span>
         </p>
         <div v-if="!innerclosed && length" class="json-body">
           <template v-for="(item, index) in items">
             <json-view :closed="closed" v-if="item.isJSON" :key="index" :json="item.value" @changeTextarea="changeTextarea"
                        :jsonKey="item.key" :isLast="index === items.length - 1"></json-view>
             <p class="json-item" v-else :key="index" @click="setActive($event,item,index)">
             <span class="json-key">
                 {{(isArray ? '' : '"' + item.key + '"')}}
             </span>
               :
               <span class="json-value">
                 {{item.value + (index === items.length - 1 ? '' : ',')}}
             </span>
                <input v-model="item.value" v-if="choose==item.key" @blur="changeTextarea11"></input>
             </p>
           </template>
           <span v-show="!innerclosed" class="body-line"></span>
         </div>
         <p v-if="!innerclosed && length" class="last-line">
           <span>{{subfix}}</span>
         </p>
       </div>
     </div>
   </div>
 </template>
 
 <script>
    import $ from "jquery" //在需要使用的页面中
   export default {
     name: 'jsonView',
     props: {
       json: [Object, Array],
       jsonKey: {
         type: String,
         default: ''
       },
       closed: {
         type: Boolean,
         default: false
       },
       isLast: {
         type: Boolean,
         default: true
       },
       fontSize: {
         type: Number,
         default: 13
       }
     },
     created() {
       this.innerclosed = this.closed
       this.$watch('closed', () => {
         this.innerclosed = this.closed
       })
     },
     data() {
       return {
         innerclosed: true,
         choose:'',
         itemIii:''
       }
     },
     methods: {
       setActive(e,item,index){
        $(".json-item").removeClass("chooseItem")
         if($(e.target).hasClass("json-item")){
           $(e.target).addClass("chooseItem")
         }else{
          $(e.target).parent(".json-item").addClass("chooseItem")
         }
          top.activeItem={       //当前选中信息暴露到最外层
           activeItems:this.items,
           activeP:this.item,
           activeIndex:index,
           activeThis:this
         }
         this.itemIii=item
         top.JsonView.inputVal=item.value  //输入框内显示选中的值
       },
      //  修改选中值
      setActiveVal(value){
        this.$set(top.activeItem.activeItems[top.activeItem.activeIndex],'value',value)
        this.$set(this.items[top.activeItem.activeIndex],'value',value)
        this.$set(this.itemIii,'value',value)
          this.$nextTick(v=>{
            this.changeTextarea11()
          })
      },
       isObjectOrArray(source) {
         const type = Object.prototype.toString.call(source)
         const res = type === '[object Array]' || type === '[object Object]'
         return res
       },
       toggleClose() {
         if (this.innerclosed) {
           this.innerclosed = false
         } else {
           this.innerclosed = true
         }
       },
       //  输入改变左侧内容
       changeTextarea(items){
          // let aaa=JSON.parse(JSON.stringify(this.json))
          let aaa=this.json
          if(Array.isArray(this.json)){
            for(let i= 0;i<aaa.length;i++){
              if(this.isObjectValueEqual(aaa[i],items['oldJson'])){
                aaa[i]=items['newJson']
              }
            }
          }else{
            for(var i in aaa) {
              if(this.isObjectValueEqual(aaa[i],items['oldJson'])){
                aaa[i]=items['newJson']
              }
            }
          }
        let jsonObj={
            oldJson:this.json,
            newJson:aaa
          }
         this.$emit("changeTextarea",jsonObj)
       },
       changeTextarea11(){
        //  this.choose=''
         console.log(this.json)
        //  判断是对象或者数组
          let newJson=null
          if(Array.isArray(this.json)){
            // newJson=[]
            // this.items.forEach(v=>{
            //   newJson[v.key]=v.value
            // })
          }else{
            newJson={}
            this.items.forEach(v=>{
              newJson[v.key]=v.value
            })
          }
         let jsonObj={
            oldJson:this.json,
            newJson:newJson
          }
         this.$nextTick(v=>{
            this.$emit("changeTextarea",jsonObj)
         })
       },
       //  判断两个对象是否相等
       isObjectValueEqual(aa,bb){
         let a=JSON.parse(JSON.stringify(aa))
         let b=JSON.parse(JSON.stringify(bb))
        //如果a和b本来就全等
        if(a===b){
          //判断是否为0和-0
          return a !== 0 || 1/a ===1/b;
        }
        //判断是否为null和undefined
        if(a==null||b==null){
          return a===b;
        }
        //接下来判断a和b的数据类型
        var classNameA=toString.call(a),
          classNameB=toString.call(b);
        //如果数据类型不相等，则返回false
        if(classNameA !== classNameB){
          return false;
        }
        //如果数据类型相等，再根据不同数据类型分别判断
        switch(classNameA){
          case '[object RegExp]':
          case '[object String]':
          //进行字符串转换比较
          return '' + a ==='' + b;
          case '[object Number]':
          //进行数字转换比较,判断是否为NaN
          if(+a !== +a){
            return +b !== +b;
          }
          //判断是否为0或-0
          return +a === 0?1/ +a === 1/b : +a === +b;
          case '[object Date]':
          case '[object Boolean]':
          return +a === +b;
        }
        //如果是对象类型
        if(classNameA == '[object Object]'){
          //获取a和b的属性长度
          var propsA = Object.getOwnPropertyNames(a),
            propsB = Object.getOwnPropertyNames(b);
          if(propsA.length != propsB.length){
            return false;
          }
          for(var i=0;i<propsA.length;i++){
            var propName=propsA[i];
            //如果对应属性对应值不相等，则返回false
            if(typeof a[propName] == 'object'  && typeof b[propName] == 'object'){
              for(let i= 0;i<a[propName].length;i++){
                this.isObjectValueEqual(a[propName][i],b[propName][i])
              }
            }else{
              if(a[propName] != b[propName]){
                return false;
              }
            }
          }
          return true;
        }
        //如果是数组类型
        if(classNameA == '[object Array]'){
          if(a.toString() == b.toString()){
            return true;
          }
          return false;
        }
      },
     },
     computed: {
       isArray() {
         return Object.prototype.toString.call(this.json) === '[object Array]'
       },
       length() {
         return this.isArray ? this.json.length : Object.keys(this.json).length
       },
       subfix() {
         return (this.isArray ? ']' : '}') + (this.isLast ? '' : ',')
       },
       prefix() {
        return this.isArray ? '[' : '{'
      },
      items() {
        if (this.isArray) {
          return this.json.map(item => {
            const isJSON = this.isObjectOrArray(item)
            return {
              value: isJSON ? item : item,
              isJSON,
              key: ''
            }
          })
        }
        const json = this.json
        return Object.keys(json).map(key => {
          const item = json[key]
          const isJSON = this.isObjectOrArray(item)
          return {
            value: isJSON ? item : item,
            isJSON,
            key
          }
        })
      }
    }
  }
</script>

<style>
  .bgView {
    background-color: #fafafa;
  }

  .json-view {
    position: relative;
    display: block;
    width: 100%;
    height: 100%;
    white-space: nowrap;
    padding-left: 20px;
    box-sizing: border-box;
  }

  .json-note {
    color: #909399;
  }

  .json-key {
    color: rgb(147, 98, 15);
  }

  .json-value {
    color: rgb(24, 186, 24);
  }

  .json-item {
    margin: 0;
    padding-left: 20px;
  }

  .first-line {
    padding: 0;
    margin: 0;
  }

  .json-body {
    position: relative;
    padding: 0;
    margin: 0;
  }

  .json-body .body-line {
    position: absolute;
    height: 100%;
    width: 0;
    border-left: dashed 1px #bbb;
    top: 0;
    left: 2px;
  }

  .last-line {
    padding: 0;
    margin: 0;
  }

  .angle {
    position: absolute;
    display: block;
    cursor: pointer;
    float: left;
    width: 20px;
    text-align: center;
    left: 0;
  }

  .angle::after {
    content: "";
    display: inline-block;
    width: 0;
    height: 0;
    vertical-align: middle;
    border-top: solid 4px #333;
    border-left: solid 6px transparent;
    border-right: solid 6px transparent;
  }

  .angle.closed::after {
    border-left: solid 4px #333;
    border-top: solid 6px transparent;
    border-bottom: solid 6px transparent;
  }
 .chooseItem{
    background:yellow
  }
  .json-item:hover{
    background:#f2f2f2
  }
</style>
