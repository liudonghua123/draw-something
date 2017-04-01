<template>
    <canvas id="canvas" style="border: 1px solid #999;"></canvas>
    <div id="keyword-box">
        <span>Keyword: </span>
        <span id="keyword"></span>
    </div>
    <button id="btn">清空画布</button>
</template>

<script>
'use strict'

class Draw {
    constructor(el) {
        this.el = el
        this.canvas = document.getElementById(this.el)
        this.ctx = this.canvas.getContext('2d')
        this.stage_info = canvas.getBoundingClientRect()
        this.path = {
            beginX: 0,
            beginY: 0,
            endX: 0,
            endY: 0
        }
    }

    init(ws, btn) {
        let onMove = false;

        this.canvas.onmousedown = this.canvas.ontouchstart = () => {
            onMove = true
            console.info(event.type, "triggered");
            event.preventDefault()
            this.drawBegin(event, ws)
        }

        this.canvas.onmousemove = this.canvas.ontouchmove = () => {
            if(onMove) {
                console.info(event.type, "triggered");
                event.preventDefault()
                this.drawing(event, ws)
            }
        }
        //
        this.canvas.onmouseup = this.canvas.ontouchend = this.canvas.ontouchcancel = () => {
            onMove = false
            console.info(event.type, "triggered");
            event.preventDefault()
//            this.drawEnd()
            ws.send('stop')
        }
        this.clearCanvas(ws, btn)
    }
    drawBegin(e, ws) {
//        window.getSelection() ? window.getSelection().removeAllRanges() : document.selection.empty()
        this.ctx.strokeStyle = "#000"

        this.ctx.beginPath()

        let x = (e.clientX ? e.clientX : e.touches[0].clientX) - this.stage_info.left;
        let y = (e.clientY ? e.clientY : e.touches[0].clientY) - this.stage_info.top;

        this.ctx.moveTo(x, y)

        this.path.beginX = x
        this.path.beginY = y
    }
    drawing(e, ws) {

        let x = (e.clientX ? e.clientX : e.touches[0].clientX) - this.stage_info.left;
        let y = (e.clientY ? e.clientY : e.touches[0].clientY) - this.stage_info.top;

        this.ctx.lineTo(x, y)

//        this.path.startX = this.path.endX
//        this.path.startY = this.path.endY
        this.path.endX = x
        this.path.endY = y

        let sendData = parseInt(this.path.beginX) + '.' + parseInt(this.path.beginY) + '.' + parseInt(this.path.endX) + '.' + parseInt(this.path.endY);
        console.info("send data", sendData);
        ws.send(sendData)

        this.ctx.stroke()
    }
    drawEnd() {
        document.onmousemove = document.onmouseup = this.canvas.ontouchmove = this.canvas.ontouchend = null
    }
    clearCanvas(ws, btn) {
        btn.onclick = () => {
            this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height)
            ws.send('clear')
        }
    }
}

export default {
    ready() {
        const ws = new WebSocket('ws://' + window.location.hostname + ':8090')
        let draw = new Draw('canvas')
        let btn = document.getElementById('btn')
        ws.onopen = () => {
            draw.init(ws, btn)
        }
        ws.onmessage = (msg) => {
            msg.data.split(':')[0] == 'keyword' ?
                document.getElementById('keyword').innerHTML = msg.data.split(':')[1] :
                false
        }
    }
}
</script>

<style lang="less">
    #canvas {
        background: pink;
        cursor: default;
    }
    #keyword-box {
        margin: 10px 0;
    }
</style>