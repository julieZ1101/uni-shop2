<template>
	<view>
		<view class="search-box">
<uni-search-bar  @input="input" cancelButton="none" :radius="100"></uni-search-bar> 
		</view>
    <view class="sugg-list" v-if="searchResult.length!== 0">
      <view class="sugg-item" v-for="(item,i) in searchResult" :key="i" @click="gotoDetail(item)">
        <view class="goods-name">{{item.goods_name}}</view>
        <uni-icons type="forward" size="16"></uni-icons>
</view>
 </view>
  <view class="history-box" v-else>
 <view class="history-title"">
 <text>搜索历史</text> 
 <uni-icons type="trash" size="16" @click="clean"></uni-icons> 
  </view>
 <view class="history-list">
   <uni-tag :text="item" v-for="(item, i) in histories" :key="i" @click='gotoGoodslist(item)'></uni-tag>
 </view>
  </view>
   </view>
  
</template>

<script>
	export default {
		data() {
			return {
        timer :null,
        kw :'',
        searchResult:[],
        historyList:[]
				
			};
		},
    methods:{
      input(e){
        // console.log(e.value)
        clearTimeout(this.timer)
        this.timer = setTimeout(()=>{
          // console.log(e.value)
          this.kw = e.value
          this.getSearchResult()
        },500)
      },
     async getSearchResult(){
       if (this.kw==='') {
         this.searchResult=[]
         return
       }
       const {data:res}=await uni.$http.get('/api/public/v1/goods/qsearch',{query:this.kw}) 
         if(res.meta.status!==200) return uni.$showMsg() 
         this.searchResult=res.message
       this.saveSearchHistory()
      },
      gotoDetail(item){
        // console.log(item.goods_id)
        uni.navigateTo({
          url:'/subpkg/goods_detail/goods_detail?goods_id='+'item.goods_id'
        })
      },
     saveSearchHistory(){
   
       const set =new Set(this.historyList)
       set.delete(this.kw)
       set.add(this.kw)
      this.historyList = Array.from (set)
      uni.setStorageSync('kw',JSON.stringify(this.historyList))
     },
     clean(){
       this.historyList=[]
       uni.setStorageSync('kw','[]')
     },
     gotoGoodslist(kw){
       uni.navigateTo({
          url: '/subpkg/goods_list/goods_list?query =' +kw
       })
      
     }
    },
    computed:{
      histories(){
        return [...this.historyList].reverse()
      }
    },
    onload(){
      this.historyList= JSON.parse(uni.getStorageSync('kw')||'[]')
    }
	}
</script>

<style lang="scss">
.search-box{
  position: sticky;
  top: 0;
  z-index: 999;
}
.sugg-list{
  padding: 0 5px;
  .sugg-item{
    display: flex;
    font-size: 12px;
    align-items: center;
    justify-content: space-between;
    padding: 13px 0;
    border-bottom: 1px solid #efefef;
    .goods-name{
      white-space: nowrap;
      text-overflow: ellipsis;
      overflow: hidden;
      margin-right: 3px;
      
    }
  }
}
.history-box{
  padding: 0 5px;
  .history-title{
    display: flex;
    justify-content: space-between;
    font-size: 13px;
    height: 40px;
    align-items: center;
     border-bottom: 1px solid #efefef;
     }
    .history-list{
      display: flex;
      flex-wrap: wrap;
      .uni-tag{
        margin-top: 5px;
        margin-right: 5px;
      }
    }
  
}
</style>
