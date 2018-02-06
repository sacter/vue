<template>
    <div class="pos">
        <div>
            <el-row>
                <el-col :span="7" class="pos-order" id="order-list">
                    <el-tabs>
                        <el-tab-pane label="点餐">
                            <el-table :data="tableData" border style="width: 100%" >                           
                                <el-table-column prop="goodsName" label="商品名称"></el-table-column>
                                <el-table-column prop="count" label="数量" width="50"></el-table-column>
                                <el-table-column prop="price" label="单价" width="70"></el-table-column>
                                <el-table-column  label="操作" width="100" fixed="right">
                                    <template slot-scope="scope">
                                        <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                                        <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>           
                                    </template>
                                </el-table-column>
                            </el-table> 
                            <div class="total">
                                <span>总量：{{totalCount}}</span>
                                <span>总价：{{totalMoney}}元</span>
                            </div>
                            <div class="div-btn">
                                <el-button type="warning" >挂单</el-button>
                                <el-button type="danger" @click="delAllGoodsy">删除</el-button>
                                <el-button type="success" >结账</el-button>
                            </div> 
                        </el-tab-pane>
                        <el-tab-pane label="挂单">
                        挂单
                        </el-tab-pane>
                        <el-tab-pane label="外卖">
                        外卖
                        </el-tab-pane>
                    </el-tabs>
                </el-col>
                <el-col :span="17">
                    <div class="often-goods">
                        <div class="title">常用商品</div>
                        <div class="often-goods-list"> 
                            <ul>
                                <li v-for="oftenGood of oftenGoods" @click="addOrderList(oftenGood)" :key="oftenGood.goodsId">
                                    <span>{{oftenGood.goodsName}}</span>
                                    <span class="o-price">￥{{oftenGood.price}}元</span>
                                </li>                    
                            </ul>
                        </div>
                    </div>
                    <div class="goods-type">
                        <el-tabs>
                            <el-tab-pane label="汉堡">
                                <ul class='cookList'>
                                    <li v-for="goods of type0Goods" @click="addOrderList(goods)" :key="goods.goodsId">
                                        <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                                        <span class="foodName">{{goods.goodsName}}</span>
                                        <span class="foodPrice">￥{{goods.price}}元</span>
                                    </li>
                                </ul>
                            </el-tab-pane>
                                <el-tab-pane label="小食">
                                <ul class='cookList'>
                                    <li v-for="goods of type1Goods" @click="addOrderList(goods)" :key="goods.goodsId">
                                        <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                                        <span class="foodName">{{goods.goodsName}}</span>
                                        <span class="foodPrice">￥{{goods.price}}元</span>
                                    </li>
                                </ul>
                            </el-tab-pane>
                            <el-tab-pane label="饮料">
                                <ul class='cookList'>
                                    <li v-for="goods of type2Goods" @click="addOrderList(goods)" :key="goods.goodsId">
                                        <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                                        <span class="foodName">{{goods.goodsName}}</span>
                                        <span class="foodPrice">￥{{goods.price}}元</span>
                                    </li>
                                </ul>
                            </el-tab-pane>
                            <el-tab-pane label="套餐">
                                <ul class='cookList'>
                                    <li v-for="goods of type3Goods" @click="addOrderList(goods)" :key="goods.goodsId">
                                        <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                                        <span class="foodName">{{goods.goodsName}}</span>
                                        <span class="foodPrice">￥{{goods.price}}元</span>
                                    </li>
                                </ul>
                            </el-tab-pane>                  
                        </el-tabs>
                    </div>
                </el-col>
            </el-row>
        </div>
    </div> 
</template>

<script>
    import axios from 'axios';
    export default{
        name:'Pos',
        data(){
            return{
                totalCount:0, //商品总数量
                totalMoney:0,  //商品总价
                tableData: [],
                oftenGoods:[],
                type0Goods:[],
                type1Goods:[],
                type2Goods:[],
                type3Goods:[]
            }
        },
        created(){
            axios.get('http://jspang.com/DemoApi/oftenGoods.php')
            .then(response=>{
                this.oftenGoods=response.data;
            }).catch(error=>{
                console.log(error);               
            });
            axios.get('http://jspang.com/DemoApi/typeGoods.php')
            .then(response=>{
                let result=response.data;
                this.type0Goods=result[0];
                this.type1Goods=result[1];
                this.type2Goods=result[2];
                this.type3Goods=result[3];
            }).catch(error=>{
                console.log(error);          
            })
        },
        mounted(){
            var orderHeight=document.body.clientHeight;
            document.getElementById('order-list').style.height=orderHeight+'px';
        },
        methods:{
            addOrderList(goods){   
                let isHave=false;
                //判断是否这个商品已经存在于订单列表
                for(let i=0;i<this.tableData.length;i++){
                    if(this.tableData[i].goodsId===goods.goodsId){
                        isHave=true;
                    }
                };
                //根据isHave的值判断订单列表中是否已经有此商品
                if(isHave){
                    let arr=this.tableData.filter(o=>o.goodsId===goods.goodsId);
                    arr[0].count++;
                }else{
                    let newGoods={goodsId:goods.goodsId,goodsName:goods.goodsName,price:goods.price,count:1};
                    this.tableData.push(newGoods);
                };     
                this.getTotal();        
            },           
            delSingleGoods(goods){ //删除单个商品
                this.tableData=this.tableData.filter(o => o.goodsId !=goods.goodsId);
                this.getTotal();
            },
            getTotal(){ //汇总数量和金额
                this.totalCount=0; //商品总数量
                this.totalMoney=0;  //商品总价
                if(this.tableData){
                    //进行数量和价格的汇总计算
                    this.tableData.forEach((element)=>{
                        this.totalCount+=element.count;
                        this.totalMoney=this.totalMoney+(element.price*element.count);
                    }) 
                }
            },
            delAllGoods(){
                this.tableData=[];
                this.totalCount=0;
                this.totalMoney=0;
            }
        }
    }
</script>
<style scoped>
.pos-order {
  background-color: #f9fafc;
  border-right: 1px solid #c0ccda;
  height: 100%;
  text-align: center;
}
.div-btn {
  margin-top: 10px;
}
.title {
  height: 20px;
  border-bottom: 1px solid #d3dce6;
  background-color: #f9fafc;
  padding: 10px;
}
.often-goods-list ul li {
  list-style: none;
  float: left;
  border: 1px solid #e5e9f2;
  padding: 10px;
  margin: 5px;
  background-color: #fff;
}
.o-price {
  color: #58b7ff;
}
.goods-type {
  clear: both;
  padding: 10px;
}
.cookList li {
  list-style: none;
  width: 23%;
  border: 1px solid #e5e9f2;
  height: auot;
  overflow: hidden;
  background-color: #fff;
  padding: 2px;
  float: left;
  margin: 2px;
}
.cookList li span {
  display: block;
  float: left;
}
.foodImg {
  width: 40%;
}
.foodName {
  font-size: 18px;
  padding-left: 10px;
  color: brown;
}
.foodPrice {
  font-size: 16px;
  padding-left: 10px;
  padding-top: 10px;
}
.total {
  text-align: right;
  background: #fff;
}
.total span {
  margin-right: 20px;
  line-height: 30px;
}
</style>
