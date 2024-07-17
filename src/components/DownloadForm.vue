<template>
    <div class="download-form">
      <h1>m3u8视频下载器</h1>
      <div class="form-group">
          <label for="m3u8-url">m3u8 URL:</label>
          <input
            type="text"
            id="m3u8-url"
            v-model="m3u8Url"
            placeholder="输入视频地址"
            required
          />
      </div>
      <button type="submit" :disabled="isDownloading" @click="downloadVideo">点击下载</button>

      <div v-if="isDownloading">
        <p style="color: green;">{{ downloadProcess }}</p>
      </div>
      <div v-if="downloadUrl">
        <p>下载完成！点击下面的链接下载视频:</p>
        <a :href="downloadUrl" target="_blank">下载视频</a>
      </div>
      <div v-if="error">
        <p style="color: red;">{{ error }}</p>
      </div>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  import config from '@/config.js';

  export default {
    data() {
      return {
        m3u8Url: '',
        downloadUrl: '',
        error: '',
        isDownloading: false,
        downloadProcess: '' //下载进度
      };
    },
    methods: {
      async downloadVideo() {
        
        if(!this.m3u8Url){
          this.error = '请输入视频地址';
          return;
        }
        this.downloadUrl = '';
        this.error = '';
        this.isDownloading = true;
        this.downloadProcess = '正在准备下载...';
        try {
          const response = await fetch(`${config.apiBaseUrl}/download`, {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify({ m3u8_url: this.m3u8Url })
          });
          const data = await response.json();
          if (response.ok) {
            let file_uuid = data.file_uuid;
            var eventSource = new EventSource(`${config.apiBaseUrl}/stream/${file_uuid}`);
            eventSource.addEventListener("progress",(event) => {
                console.log(event.data);
                this.isDownloading = true;
                this.downloadProcess = event.data;
            })
            eventSource.addEventListener("success",(event) => {
                console.log(event.data);
                this.downloadUrl = event.data;
                this.isDownloading = false;
                eventSource.close();

            })
            eventSource.addEventListener("error",(event) => {
                console.log(event.data);
                this.error = event.data;
                this.isDownloading = false;
                eventSource.close();
            })


          } else {
            this.error = data.error || '下载失败';
          }
        } catch (err) {
          this.error = '请求失败';
          this.isDownloading = false;

        } 
      },
    }
  };
  </script>
  
  <style>
  .download-form {
    max-width: 100%;
    margin: 50px auto;
    padding: 20px;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
  .form-group {
    margin-bottom: 15px;
  }
  label {
    display: block;
    margin-bottom: 5px;
  }
  input {
    width: 100%;
    padding: 8px;
    box-sizing: border-box;
  }
  button {
    padding: 10px 20px;
  }
  </style>
  