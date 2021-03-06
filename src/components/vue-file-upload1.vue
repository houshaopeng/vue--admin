<style lang="scss" scoped>
  .fileupload-button{
    position: relative;
    overflow: hidden;
    display:inline-block;
    color:#fff;
    padding:6px 12px;
    background-color:#5cb85c;
    border-color: #4cae4c;
    margin: 0;
    font-size: 14px;
    font-weight: 400;
    line-height: 1.42857143;
    text-align: center;
    white-space: nowrap;
    vertical-align: middle;
    cursor: pointer;
  }
  input[type=file]{
    position: absolute;
    top: 0;
    right: 0;
    margin: 0;
    opacity: 0;
    -ms-filter: 'alpha(opacity=0)';
    font-size: 200px;
    direction: ltr;
    cursor: pointer;
  }

</style>
<template>
<span class="fileupload-button">
  <i v-if='icon != null' v-bind:class='rendIcon'></i>
  {{label}}
  <input ref="fileInput" type="file" name="file" disabled="true">
</span>
</template>
<script>
  /*!
   * vue-file-upload
   * 基于vue.js的文件上传组件
   * 为了更好的给业务组件提供api，就将fileuploader类隐藏在内部，不对外开放；
   * 带来的问题也有很多：比如队列文件的同步
   * @param {String:url} ［上传图片地址］
   * @param {Number:max} [上传图片数量]
   * @param {String:label} [按钮显示名称]
   * @param {String:name} [上传图片name属性]
   * @param {String:icon} [图标class]
   * @param {Boolean:autoUpload} [自动上传：true：是，false：否]
   * @param {Boolean:multiple} [多文件上传：true：是，false：否]
   * @param {Array:files} [多文件队列，通常需要和开发组件双向绑定]
   * @param {Object:filters} [过滤器，对象包含name和fn属性，可用于限制上传文件类型，大小]
   * @param {Object:requestOptions} [请求附加参数:  formData:{},headers:{},responseType:'json',withCredentials:false]
   * @param {Object:events} [回调函数:onProgressUpload,onCompleteUpload,onErrorUpload,onSuccessUpload,onAbortUpload,onAddFileFail,onAddFileSuccess]
   */
  import _ from './public.js';
  import UploadActions from './msg.js';
  import FileUploader from './fileUploader.js';

  var noop = ()=>{};
  export default{
      props:{
          url:{
              type:String,
              required:true
          },
          max:{
              type:Number,
              default:Number.MAX_VALUE
          },
          label:{
              type:String,
              default:'无法上传'
          },
          outletNum:{
              type:String,
          },
          name:{
              type:String,
              default:'file'
          },
          icon:{
              type:String,
              default:null
          },
          autoUpload:{
              type:Boolean,
              default:false
          },
          multiple:{
              type:Boolean,
              default:true
          },
          onAdd:{
              type:Function,
              default:noop
          },
          filters:{
              type:Array,
              default:()=>{
                  return new Array();
              }
          },
          requestOptions:{
              type:Object,
              default:()=>{
                  return{
                      formData:{'applicationNo':this.outletNum},
                      headers:{"x-sljr-session-token": JSON.parse(sessionStorage.getItem("userInfo")).userToken},
                      responseType:'json',
                      withCredentials:false
                  }
              }
          },
          events:{
              type:Object,
              default:()=>{
                  return {
                      onProgressUpload:noop,
                      onCompleteUpload:noop,
                      onErrorUpload:noop,
                      onSuccessUpload:noop,
                      onAbortUpload:noop,
                      onAddFileFail:noop,
                      onAddFileSuccess:noop
                  }
              }
          }
      },
      data(){
          return {
              fileUploader:null
          }
      },
      computed:{
          rendIcon(){
              return this.icon.split(/\s/g);
          },
          bFiles:{
              get(){
                  return this.files;
              },
              set(value){
                  this.files = value;
              }

          }
      },

      created(){
          this.fileUploader = new FileUploader({
              vm:this,
              url:this.url,
              name:this.name,
              maxItems:this.max,
              filters:this.filters,
              multiple:this.multiple,
              autoUpload:this.autoUpload,
              ...this.requestOptions,
              ...this.events
          });
      },
      mounted(){
          if(this.$refs.fileInput && this.multiple){
              this.$refs.fileInput.setAttribute('multiple',this.multiple);
          }
          _.on(this.$refs.fileInput,"change",this._onChange);
          console.log(this.requestOptions)
      },
      beforeDestroy(){
          _.off(this.$refs.fileInput,"change");
          this.fileUploader.clearQueue();
      },
      methods:{
          uploadAll(){
              this.fileUploader.uploadAll();
          },
          clearAll(){
              this.fileUploader.clearQueue();
          },
          _abortUpload(file){
              this.fileUploader.abortItem(file);
          },
          _onChange(){
              //文件已数组为单位，因为可能存在mutiple；如果是单个文件fileuploader会自动转换为数组类型！
              var elTargetFiles = _.isHTML5() ? this.$refs.fileInput.files :this.$refs.fileInput;
              this.fileUploader.addToQueue(elTargetFiles);
              this.$emit('onAdd',this.fileUploader.getAll());
              this._resetInput();
          },
          _resetInput(){
              //重置input值，可以上传重复
              this.$refs.fileInput.value = '';
          }
      }
  }
</script>
