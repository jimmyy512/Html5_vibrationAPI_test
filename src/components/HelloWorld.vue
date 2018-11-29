<template>
    <div id="main" class="v2">
        <div class="rowBlock0">
            <span>請先點擊屏幕一下</span>
        </div>
        <div class="rowBlock1">
            <span>
                支援陀螺儀：
            </span>
            <span :class="isSupportDeviceMotionCSS()">
                {{isSupportDeviceMotionLabel()}}
            </span>
        </div>
        <div class="rowBlock2">
            <span>
                支援震動：
            </span>
            <span :class="isSupportVibrateCSS()">
                {{isSupportVibrateLabel()}}
            </span>
        </div>
        <div class="rowBlock3">
            <div class="demoBox" :class='demoBoxBGJudge()'>
            </div>
        </div>

        <div class="rowBlock4">
            <div>
              <span>
                上次搖動最大值{{maxShake.toFixed(2)}}
              </span>
              <div>
                <input class="clearButton" type="button" value="清除最大值" @click="maxShake=0">
              </div>
            </div>
            <div>
              X or Y 超過{{this.speed}}就會震動
            </div>
            <div>
              <span>Ｘ: {{shakeX.toFixed(2)}}</span>
            </div>
            <div>
              <span>Ｙ: {{shakeY.toFixed(2)}}</span>
            </div>
            <div>
              <span>Ｙ: {{shakeZ.toFixed(2)}}</span>
            </div>
            <div>
              <input class="button" type="button" value="點我強制震動" @click="forceShake()">
            </div>
            <div>
              <span>
                震動觸發值
              </span>
              <input class="input" type="text" v-model="speed" >
            </div>
        </div>
        
    </div>
</template>

<script>
export default {
    created(){
        this.addShakeEvent();
        if("vibrate" in navigator)
        {
          this.isSupportVibrate=true;
          navigator.vibrate(0);
        }
    },
    data(){
        return{
          isSupportDeviceMotion:false,
          isSupportVibrate:false,
          boxColorIndex:0,
          speed:10,  // 用来判定的加速度阈值，太大了则很难触发
          maxShake:0,
          shakeX:0,
          shakeY:0,
          shakeZ:0,
          x:0,
          y:0,
          z:0,
          lastX:0,
          lastY:0,
          lastZ:0,
          lastAcc:{
            alpha:0,
            beta:0,
            gamma:0,
          },
        }
    },
    methods:{
        forceShake(){
          if("vibrate" in navigator){
            navigator.vibrate(1000);
          }
        },
        demoBoxBGJudge()
        {
          if(this.boxColorIndex==1)
            return {"_1":true};
          else if(this.boxColorIndex==2)
            return {"_2":true};
          else if(this.boxColorIndex==3)
            return {"_3":true};
          else if(this.boxColorIndex==4)
            return {"_4":true};
          else if(this.boxColorIndex==5)
            return {"_5":true};
        },
        isSupportDeviceMotionLabel(){
            if(this.isSupportDeviceMotion)
                return "支援";
            else
                return "不支援";
        },
        isSupportDeviceMotionCSS(){
            if(this.isSupportDeviceMotion)
                return {"green":true};
            else
                return {"red":true};
        },
        isSupportVibrateCSS(){
            if(this.isSupportVibrate)
                return {"green":true};
            else
                return {"red":true};
        },
        isSupportVibrateLabel(){
            if(this.isSupportVibrate)
                return "支援";
            else
                return "不支援";
        },
        addShakeEvent(){
            if(window.DeviceMotionEvent) {
              window.addEventListener('devicemotion',this.shakeDevicemotion, false);
              // window.addEventListener('deviceorientation',this.shakeDeviceorientation, false);
            }
        },
        shakeDevicemotion()
        {
            this.isSupportDeviceMotion=true;
            let acceleration = event.accelerationIncludingGravity;
            let acc = event.acceleration;
            this.x = acc.x;
            this.y = acc.y;
            this.shakeX=Math.abs(this.x-this.lastX);
            this.shakeY=Math.abs(this.y-this.lastY);
            if(this.shakeX>this.maxShake)
              this.maxShake=this.shakeX;
            if(this.shakeY>this.maxShake)
              this.maxShake=this.shakeY;
            if(this.shakeX > this.speed ||  this.shakeY> this.speed) {
                // 用户设备摇动了，触发响应操作
                // 此处的判断依据是用户设备的加速度大于我们设置的阈值
                this.boxColorIndex++;
                if(this.boxColorIndex==6)
                  this.boxColorIndex=1;
                if("vibrate" in navigator){
                  navigator.vibrate(700);
                }
            }
            this.lastX = this.x;
            this.lastY = this.y;
        },
        shakeDeviceorientation(event){
          let delA = Math.abs(event.alpha - this.lastAcc.alpha);    // alpha轴偏转角
          let delB = Math.abs(event.beta - this.lastAcc.beta);    // beta轴偏转角
          let delG = Math.abs(event.gamma - this.lastAcc.gamma);    // gamma轴偏转角
        
          if ((delA > this.speed && delB > this.speed) || 
              (delA > this.speed && delG > this.speed) || 
              (delB > this.speed || delG > this.speed)) {
            // 用户设备摇动了，触发响应操作
            // 此处的判断依据是用户设备的加速度大于我们设置的阈值
            this.boxColorIndex++;
            if(this.boxColorIndex==6)
              this.boxColorIndex=1;
            if("vibrate" in navigator){
              navigator.vibrate(700);
            }
          }
          this.shakeX=delA;
          this.shakeY=delB;
          this.lastAcc = event;    // 存储上一次的event
        },
    }
}
</script>

<style lang="scss" scoped>
.red{
    color:red;
}
.green{
    color:green;
}
._1
{
  background-color:gold !important;
}
._2{
  background-color:firebrick !important;
}
._3{
  background-color:deepskyblue !important;
}
._4{
  background-color:greenyellow !important;
}
._5{
  background-color:hotpink !important;
}
#main{
    width:100%;
    .rowBlock0{
      width:100%;
      position: absolute;
      top: 10%;
      left: 50%;
      transform: translate(-50%,-50%);
    }
    .rowBlock1{
      width:100%;
      position: absolute;
      top: 20%;
      left: 50%;
      transform: translate(-50%,-50%);
    }
    .rowBlock2{
      width:100%;
      position: absolute;
      top: 30%;
      left: 50%;
      transform: translate(-50%,-50%);
    }
    .rowBlock3{
      width:100%;
      position: absolute;
      top: 45%;
      left: 50%;
      transform: translate(-50%,-50%);
      .demoBox{
        width: 70px;
        height:70px;
        margin: 10px auto;
        background-color:gray;
      }
    }
    .rowBlock4{
      width:100%;
      position: absolute;
      top: 75%;
      left: 50%;
      transform: translate(-50%,-50%);
      .clearButton
      {
        background-color: skyblue;
        margin-top:5px;
        margin-bottom:5px;
      }
      .button{
        margin-top:10px;
        background-color: skyblue;
      }
      .input{
        margin-top:10px;
        width:25px;
        background-color: skyblue;
      }
    }
}
</style>


