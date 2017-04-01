<template>
    <canvas id="showing" style="border: 1px solid #999;"></canvas>
    <div id="answer-box">
        <span>Answer: </span>
        <input id="answer" type="text">
        <button id="submit">提交</button>
    </div>
</template>

<script>
    'use strict'
    export default {
        ready() {
            const ws = new WebSocket('ws://' + window.location.hostname + ':8090')
            const canvas = document.getElementById('showing')
            const ctx = canvas.getContext('2d')
            let moveToSwitch = 1
            ws.onmessage = (msg) => {
              let pathObj = msg.data.split('.')
              ctx.strokeStyle = "#000"
              
              if (moveToSwitch && msg.data != 'stop' && msg.data != 'clear') {
                  ctx.beginPath()
                  ctx.moveTo(pathObj[0], pathObj[1])
                  moveToSwitch = 0
              } else if (!moveToSwitch && msg.data == 'stop') {
                  ctx.beginPath()
                  ctx.moveTo(pathObj[0], pathObj[1])
                  moveToSwitch = 1
              } else if (moveToSwitch && msg.data == 'clear') {
                  ctx.clearRect(0, 0, this.canvas.width, this.canvas.height)
              } else if (msg.data == '答对了！！') {
                  alert('恭喜你答对了！！')
              }

              ctx.lineTo(pathObj[2], pathObj[3])
              ctx.stroke()
            }

            ws.onopen = () => {
                let submitBtn = document.getElementById('submit')
                submitBtn.onclick = () => {
                    let keyword = document.getElementById('answer').value
                    ws.send(keyword)
                }
            }
        }
    }
</script>

<style lang="less">
    #showing {
        background: lightblue;
    }
    #answer-box {
        margin: 10px 0;
    }
</style>