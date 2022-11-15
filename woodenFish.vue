<style lang="scss">
.wooden-fish {
    user-select: none;
    font-family: LiSu;
    height: 100%;
    position: relative;
    overflow-x: hidden;
    background-image: url(@/assets/images/foguang.jpg);
    background-size: 100% 100%;
    background-position: center;

    * {
        user-select: none;
    }

    .text-gold {
        color: rgba(248, 233, 46, 1);
    }

    .title {
        font-weight: bold;
        padding: 5px 0;
        display: none;
    }

    .muyu {
        background-image: url(@/assets/images/muyu.png);
        background-position: center;
        width: 300px;
        height: 300px;
        position: absolute;
        // bottom: 20%;
        top: 50%;
        // left: 50%;
        transform: translate(0, -50%);

        .mubang {
            background-image: url(@/assets/images/mubang.png);
            background-position: center;
            width: 300px;
            height: 300px;
            background-size: 100% 100%;
            position: absolute;
            // top: 50%;
            left: 180px;
            bottom: 100px;
            // transform: translate(-100px, -300px);
            // transition-duration: .3s;
            // background-color: rgba(255, 0, 0, .3);
            transform: rotate(45deg);

            &.fall {
                transform: rotate(0deg);
            }
        }
    }



    .content {
        position: absolute;
        bottom: 10px;
        left: 0;
        right: 0;
    }
}
</style>


<template>
    <div class="wooden-fish">
        <div class="center">
            <div class="title text-center text-gold">电子木鱼</div>
            <div class="padding-lr content">
                <div class="flex align-center padding-tb-xs">
                    <span class="text-gold padding-right">功德值</span>
                    <div class="text-gold">
                        <Bubble :watcher="meritSum">
                            <span ref="meritSumRef">
                                {{meritSum}}
                            </span>
                        </Bubble>

                    </div>
                </div>
                <div class="flex align-center padding-tb-xs">
                    <span class="text-gold padding-right">虔诚度</span>
                    <div class="flex-sub">
                        <van-progress :percentage="devoutSum" text-color="#111" pivot-color="rgba(248, 233, 46, 1)"
                            track-color="rgba(248, 233, 46, .2)"
                            color="linear-gradient(to right, rgba(248, 233, 46, .5), rgba(248, 233, 46, 1)" />
                    </div>
                </div>
            </div>
            <div class="muyu" @click.stop="singleKnock">
                <div class="mubang pointer" :class="{fall:isFall}" :style="`transition-duration:${timerDuration}ms;`"
                    @click="singleKnock"></div>
                <audio :src="audio" ref="knock"></audio>
            </div>
        </div>
    </div>
</template>


<script setup>
import { Howl, Howler } from 'howler';
import audio from "@/assets/audio/knock.mp3"
import {
    Notify,
} from 'vant'
import {
    getToken
} from '@/utils/auth'
import Bubble from "@/components/Bubble"
const { proxy } = getCurrentInstance();


var sound = new Howl({
    src: [audio]
});


let timer
let timerDuration = 150
const isFall = ref(false)
const isAnimating = ref(false)
// 功德值
const meritSum = ref(0)
// 虔诚度
const devoutSum = ref(70)
// 虔诚度衰减定时器
let devoutTimer
// 上一次敲击时间戳
const lastKnockTime = ref(new Date().getTime())





const infiniteKnock = () => {
    timer = setInterval(() => { singleKnock() }, timerDuration)
}

const singleKnock = () => {

    if (isAnimating.value) {
        //  console.log("防抖")
        // proxy.$modal.msg("防抖")
        return
    }
    isFall.value = true
    isAnimating.value = true
    // sound.play();

    // proxy.$refs["knock"].load()
    // proxy.$refs["knock"].play()
    setTimeout(() => {
        // 敲下动画结束后
        isFall.value = false
        sound.play();

        //  console.log("play")
    }, timerDuration)

    setTimeout(() => {
        // 动画结束后
        isAnimating.value = false
        meritSum.value++
        Notify({
            message: `功德值+1`,
            color: 'rgba(248, 233, 46, 1)',
            duration: timerDuration * 3,
            // background: 'rgba(248, 233, 46, .1)',
        })

        localStorage.setItem("meritSum", meritSum.value);


        let curKnockTime = new Date().getTime()

        //  console.log(curKnockTime - lastKnockTime.value)
        //  console.log(10000 / (curKnockTime - lastKnockTime.value))
        devoutSum.value += Number((300 / (curKnockTime - lastKnockTime.value)).toFixed(2))
        devoutSum.value > 100 ? devoutSum.value = 99.99 : devoutSum.value = Number(devoutSum.value.toFixed(2))
        lastKnockTime.value = curKnockTime
        handleMeritSumUpdate()
    }, timerDuration * 2)
}

const handleMeritSumUpdate = () => {
    //  console.log(proxy.$refs["meritSumRef"])
    //  console.log(proxy.$refs["meritSumRef"].offsetTop)
    //  console.log(proxy.$refs["meritSumRef"].clientWidth)
    //  console.log(proxy.$refs["meritSumRef"].offsetLeft + proxy.$refs["meritSumRef"].clientWidth)

    let span = document.createElement("span")
    span.innerHTML = "+1"
    span.style = `top: ${proxy.$refs["meritSumRef"].offsetTop - 20};left: ${proxy.$refs["meritSumRef"].offsetLeft + proxy.$refs["meritSumRef"].clientWidth};position: "absolute";color: "#E94F06"`
    proxy.$refs["meritSumRef"].innerHTML = proxy.$refs["meritSumRef"].innerHTML + span

}

onMounted(() => {
    // infiniteKnock()
    if (getToken()) {
        /* has token*/

    } else {
        // 没有token
        meritSum.value = parseInt(localStorage.getItem("meritSum")) | 0
    }

    devoutTimer = setInterval(() => {
        let attenuation = Number(Math.random().toFixed(2))
        ////  console.log(attenuation)
        devoutSum.value -= attenuation
        devoutSum.value > 0 ? devoutSum.value = Number(devoutSum.value.toFixed(2)) : devoutSum.value = 0
    }, timerDuration * 10)
})

onUnmounted(() => {
    clearInterval(timer)
    clearInterval(devoutTimer)
})
</script>
