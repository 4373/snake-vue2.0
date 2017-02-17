<style>
*{
  margin: 0;
  padding:0;
}
#app{
  margin: 10px auto;
  background: darkkhaki;
}
.control{
  display: flex;
  justify-content: center;
  margin: 100px auto 0;
}
.btn{
  padding:15px 20px;
  min-width: 90px;
  background:rgba(0, 0, 0, 0.5);
  color:lavenderblush;
  font-weight: bold;
  font-family: '微软雅黑';
  cursor: not-allowed;
}
.btn.active{
  background:palevioletred;
}
/*.btn:active{
  background:palevioletred;
  box-shadow: 0 0 10px rgba(0, 0, 0, 1)
}*/
.cube{
  height:20px;
  width:20px;
  display: inline-block;
  color:white;
  box-sizing: border-box;
  border: 1px slategray solid;
  vertical-align: bottom;
}
.cube.active{
  background:cadetblue;
}
.cube.first{
  background:brown;
}
.cube.food{
  background:greenyellow;
}
</style>
<template>
  <div class="box">
      <div class="control">
        <div class="btn" :class='{active:isDie}'>重来(enter)</div>
        <div class="btn" :class='{active:throughwall}'>穿墙(ctrl)</div>
        <div class="btn" :class='{active:!runing}'>暂停(space)</div>
        <div class="btn score">分数：{{long-5}}</div>
        <div class="btn score">最高分{{high}}</div>
        <div class="btn speed">速度：{{130-speed}}</div>
        <!--<input type="number" v-model='size[0]' placeholder="长">
        <input type="number" v-model='size[1]' placeholder="宽">-->
      </div>
      <div id="app" :style='{width:size[0]*20 + "px",height:size[1]*20 + "px"}' >
        <div class="cube" v-for='i in box':class='{active:include(i),first:eq(i,snake[long-1]),food:eq(i,food)}'></div>
      </div>
  </div>
</template>
<script>
let timer = null
let canTurn = true
let origin = [[12,18],[13,18],[14,18],[15,18],[16,18]]
export default {
  name: 'app',
  data () {
    return {
      size: [70, 30], // 画布大小
      snake: [[12,18],[13,18],[14,18],[15,18],[16,18]], 
      food:[-1,-1], // 食物位子
      direaction: 'ArrowRight', // 方向
      isDie:false, // 是否死亡
      runing: false, // 是否暂停
      speed:120, // 速度
      throughwall: false //是否能穿墙
    }
  },
  methods: {
    include(arr) { // snake 是否包含元素
      return window.JSON.stringify(this.snake).indexOf(window.JSON.stringify(arr)) !== -1
    },
    eq(a, b) { // 两个数组是否相等
      return window.JSON.stringify(a) === window.JSON.stringify(b)
    },
    copy(arr) { // 复制数组
      return window.JSON.parse(window.JSON.stringify(arr))
    },
    turn(d) { // 转向
      if (!canTurn) return
      canTurn = false
      let b = this.direaction === 'ArrowLeft' && d === 'ArrowRight' || this.direaction === 'ArrowRight' && d === 'ArrowLeft' || this.direaction === 'ArrowUp' && d === 'ArrowDown' || this.direaction === 'ArrowDown' && d === 'ArrowUp'
      if (b) {
        this.speed = this.speed === 120 ? 120 : this.speed + 10
      } else {
        if (d === this.direaction) {
          this.speed = this.speed === 10 ? 10 : this.speed - 10
        }
        this.direaction = d
      }
    },
    start() { // 重来
        this.isDie = false
        this.snake = this.copy(origin)
        this.direaction = 'ArrowRight'
        this.runing = false
    },
    run(b) { // 运动
      if (b) {
        timer = setInterval(this.snakeMove, this.speed)
      } else {
        clearInterval(timer)
      }
    },
    grow() { // 变长
      this.snake.unshift(this.snake[0])
    },
    foodChange() { // 随机生成食物位子
      let x,y
      do{
        x = Math.floor(Math.random()*this.size[0])
        y = Math.floor(Math.random()*this.size[1])
        this.$set(this.food,0,x)
        this.$set(this.food,1,y)
      }while(this.include(this.food))
    },
    snakeMove() { // 走一步
      if (this.isDie) return
      let old = window.JSON.parse(window.JSON.stringify(this.snake))
      let header = this.snake[old.length-1]
      switch(this.direaction) {
        case 'ArrowDown': header[1]++ // 下
        break
        case 'ArrowLeft': header[0]-- // 左
        break
        case 'ArrowRight': header[0]++ // 右
        break
        case 'ArrowUp': header[1]-- // 上
        break
      }

      let l = header[0] === -1
      let r = header[0] === this.size[0]
      let t = header[1] === -1
      let b = header[1] === this.size[1]

      if (this.throughwall) { // 穿墙而过
        l && (header[0] = this.size[0]-1)
        r && (header[0] = 0)
        t && (header[1] = this.size[1]-1)
        b && (header[1] = 0)
      } else {
        if (l || r || t || b) { // 穿墙而亡
          this.isDie = true
          this.snake = old
          return
        }
      }
      let arr = []
      for (let i = 0,j=this.long-1 ;i < j; i++) {
        if(this.eq(old[i],header)){ //碰到自身死亡
          this.isDie = true
          this.snake = old  
          return
        }
        this.$set(this.snake,i,old[i+1])
      }
      if (this.eq(header,this.food)) { //吃食物
        this.foodChange()
        this.grow()
      }
      this.$set(this.snake,this.long-1,header)
      canTurn = true
    }
  },
  watch: { // 暂停，开始
    runing:function(n){
      this.run(n)
    },
    speed:function(){
      this.run(false)
      this.run(true)
    }
  },
  computed: {
    box() { //所有方格
      let arr = []
      for (let i = 0; i < this.size[1]; i++) {
        for (let j = 0 ; j < this.size[0]; j++) {
          arr.push([j, i])
        }
      }
      return arr
    },
    long(){// 蛇长度
      return this.snake.length
    },
    high() { // 最高分
      let h = window.localStorage.getItem('high')
      h = h === null ? 0 : (this.long > h ? (this.long - origin.length) : h)
      window.localStorage.setItem('high', h)
      return h
    }
  },
  mounted() { 
    this.foodChange()
    document.addEventListener('keydown',(e) => {
      if(e.keyCode === 13) {
        this.start()
      } else if(e.keyCode > 36 && e.keyCode < 41){
        this.turn(e.code)
      } else if(e.keyCode === 32) {  // 空格 
        if(this.isDie) return
        this.runing = !this.runing
      } else if (e.keyCode === 17){
        this.throughwall = ! this.throughwall
      }
    })
  }
}
</script>

