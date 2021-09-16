<template>
  <div>
    <el-tabs v-model="activeName" ref="jsonView">
      <el-tab-pane label="JSON数据" name="first">
        <div>
          <el-button type="primary" @click="format">格式化</el-button>
          <el-button type="primary" @click="format_reset">删除空格</el-button>
        </div>
        <div>
          <el-input type="textarea" v-model="jsonData_json"></el-input>
        </div>
      </el-tab-pane>
      <el-tab-pane label="JSON格式" name="second">
        <div style="overflow:auto">
          <JsonView :json="jsonData"  @changeTextarea="changeTextarea"></JsonView>
        </div>
        <el-row>
        <el-col :span="24" style="margin-top:15px">
          <el-input v-model="inputVal" placeholder="请输入内容"></el-input>
        </el-col>
        <el-col :span="24" style="text-align:center;margin-top:5px">
          <el-button type="primary" @click="setActiveVal">确定</el-button>
        </el-col>
      </el-row>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>
<script>
import JsonView from './JsonView'
export default {
  data(){
    return{
      activeName: 'second',
      jsonData:{
        "ads": [
          {
            "active": "1",
            "ads":[{
              "name":"111",
              "age":12
            },{
              "name":"222",
              "age":13
            }],
            "ce":{
              "name":"222",
              "age":13,
              "fdsfd":"qwer"
            },
            "ad_via_link": "http://selfserve.buysellads.com/checkout/new/?utm_source=vuejs-org-custom\u0026utm_medium=ad_via_link\u0026utm_campaign=in_unit\u0026utm_term=custom",
            "external_id": "81",
            "height": "0",
            "identifier": "ce8407b0f530ec0c464687c10654d7d1",
            "longimp": "T3FY65I3TTTTTTD3OERC4TTTTTTCDR3BK6TTTTTTKDBDWYVTTTTTT",
            "noincrement": "1",
            "num_slots": "1",
            "rendering": "custom",
            "statimp": "//srv.buysellads.com/ads/imp/x/GTND42JIC67D623IF6ALYKQNCKYI62QICYYD4Z3JCY7I4KJMCKSDTKQKFTBDP2QYCTBI42QMCYBD627ECYADV23KC6SDCKQLCW7IKK3EHJWNBADLKM7UCBZG2Y",
            "timestamp": "1631256572",
            "width": "0",
            "zoneid": "30255",
            "zonekey": "CKYD62QM"
          },
          {
            "active": "1",
            "ad_via_link": "http://selfserve.buysellads.com/checkout/new/?utm_source=vuejs-org-custom\u0026utm_medium=ad_via_link\u0026utm_campaign=in_unit\u0026utm_term=custom",
            "external_id": "81",
            "height": "0",
            "identifier": "ce8407b0f530ec0c464687c10654d7d1",
            "longimp": "T3FY65I3TTTTTTD3OERC4TTTTTTCDR3BK6TTTTTTKDBDWYVTTTTTT",
            "noincrement": "1",
            "num_slots": "1",
            "rendering": "custom",
            "statimp": "//srv.buysellads.com/ads/imp/x/GTND42JIC67D623IF6ALYKQNCKYI62QICYYD4Z3JCY7I4KJMCKSDTKQKFTBDP2QYCTBI42QMCYBD627ECYADV23KC6SDCKQLCW7IKK3EHJWNBADLKM7UCBZG2Y",
            "timestamp": "1631256572",
            "width": "0",
            "zoneid": "30255",
            "zonekey": "CKYD62QM"
          }
        ]
      },
      jsonData_json:'',
      isEnd:false,   //是否已经格式化
      inputVal:""
    }
  },
  watch:{
    jsonData_json(){
      if(this.jsonData_json){
        this.jsonData = JSON.parse(this.jsonData_json)
      }else{
        this.jsonData=""
      }
    },
  },
  computed:{
  },
  components:{JsonView},
  methods:{
    // 格式化
    format(){
        // if(this.isEnd){
        //   return
        // }
        var str=JSON.stringify(this.jsonData);
        var stack = []; //栈-用于括号匹配
        var tmpStr = '';    //新格式化JSON字符串
        var len = str.length;   //原始JSON长度
        //遍历每一个字符
        for (let i = 0; i < len; i++) {
            //当遇到结构块起始结构
            if (str[i] == '{' || str[i] === '[') {
                //起始结构后面直接换行  
                tmpStr += str[i] + "\n";
                //入栈
                stack.push(str[i]);
                //这里的意思是结构块起始的下一行开始就会有一个缩进，缩进量与遇到的结构块起始符个数成正比1:1
                tmpStr += "  ".repeat(stack.length);
            } 
            //当遇到结构块结束符
            else if (str[i] == ']' || str[i] === '}') {
                //因为本身JSON格式是固定的，所以括号一定是成对的，这里先不考虑错误的json数据
                //遇到结束符就退栈，
                stack.pop();
                //结束符本身输出到下一行，并减少一个缩进
                tmpStr += "\n"+"  ".repeat(stack.length) + str[i];
            } 
            //当遇到逗号的时候
            else if (str[i] == ',') {
                //逗号后方直接换行，以及下一行的缩进处理
                tmpStr += str[i] + "\n" + "  ".repeat(stack.length);
            } 
            else {
                //其他字符直接复制
                tmpStr += str[i];
            }
        }
        this.jsonData_json = tmpStr
        this.isEnd=true
    },
    format_reset(){
      this.jsonData_json = JSON.stringify(this.jsonData)
    },
    // 中间修改，同步修改左侧值
    changeTextarea(items){
      let aaa=JSON.parse(JSON.stringify(this.jsonData))
      if(Array.isArray(aaa)){
        aaa.forEach((v,i)=>{
          if(v==items['oldJson']){
            v=items['newJson']
          }
        })
      }else{
        for(var i in aaa) {
          if(aaa[i]==items['oldJson']){
            aaa[i]==items['newJson']
          }
        }
      }
      this.jsonData_json = JSON.stringify(aaa)
      if(this.isEnd){
        this.format()
      }
    },
    setActiveVal(){
      top.activeItem.activeThis.setActiveVal(this.inputVal)
    }
  },
  mounted(){
    this.jsonData_json = JSON.stringify(this.jsonData)
    top.JsonView=this
  }
}
</script>
<style>
  textarea{
    height:680px;
    width:100%
  }
</style>