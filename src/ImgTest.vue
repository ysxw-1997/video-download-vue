<script setup>
import { ref } from 'vue'
const input = ref('')
const imageArr = ref([])
async function download(event) {
  try{
    const response = await fetch(input.value)
    const blob = await response.blob()
    imageArr.value.push(URL.createObjectURL(blob))
  }catch(e){
    console.error("下载失败",e)
  }

}
</script>
<template>
  <el-container>
    <el-aside style="width:180px;">
    </el-aside>
    <el-container>
      <el-header>XX管理系统</el-header>
      <el-main>
        <el-input v-model="input" style="width: 60%" placeholder="请输入视频地址" />
        <el-button type="primary" @click="download">下载</el-button>
        <div id="results">
          <div class="img" v-for="(item,index) in imageArr" :key="index" >
            <el-image style="max-width: 100%;" :src="item"></el-image>
          </div>
        </div>
      </el-main>
      <el-footer>版权所有</el-footer>
    </el-container>
  </el-container>
</template>
<style scoped>
#results{
  display: flex;
  flex-wrap: wrap;
}
.img{
  width: 20%;
}
</style>
