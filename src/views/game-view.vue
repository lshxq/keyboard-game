<template>
    <div class="main-panel" :class="mainPanelClass" ref="mainPanel">
        <div class="points">{{point}}</div>
        <div v-for="char in chars" 
             :key="char.id" 
             :style="{ top: char.position.y + 'px', left: char.position.x + 'px' }"
            class="letter">
                <img src="../assets/7757ddc5-36ce-4022-b06b-f290b4a78ec4.png"/>
                <div class="char">{{ char.letter }}</div>
        </div>
        <div class="welcome">
            <div class="start-btn" @click="startGame">开始游戏</div>
        </div>
        
        <div class="bar"></div>
    </div>
</template>
  
<script>

export default {
    data() {
        return {
            chars: [],
            running: false,
            point: 0,
            miss: 0,
            gameover: false
        }
    },

    computed: {
        mainPanelClass() {
            const classes = {
                started: this.running
            }
            return classes
        },
        charSize() {
            return 164
        },
        barRange() {
            return 0.9
        }
    },

    watch: {
        miss(val) {
            if (val > 10) {
                this.running = false
                this.gameover = true
            }
        }
    },

    mounted() {


        const that = this
        that.fall()
        that.$refs.mainPanel.style.setProperty('--bar-range', this.barRange)

        window.addEventListener('keydown', this.handleKeyDown)
        window.addEventListener('keyup', this.handleKeyUp)

        setInterval(() => {
            if (that.running) {
                this.generateChar()
            }

        }, 1000)
    },

    unmounted() {
        window.removeEventListener('keydown', this.handleKeyDown)
        window.removeEventListener('keyup', this.handleKeyUp)
    },

    methods: {
        createAudio() {
            const audio = document.createElement('audio');
            audio.src = '/6509e4d527ac4a3e9deb4c5e50258e6d.mp3';
            document.body.appendChild(audio);
            audio.play();
            audio.addEventListener('ended', () => {
                document.body.removeChild(audio);
            });
        },
        createImage(left) {
            const image = document.createElement('img');
            image.src = '/94d0ffa55bfb32eec644a03bb78f4cd5.gif';
            image.style.position = 'absolute';
            image.style.bottom = 0;
            image.style.left = `${left}px`;
            image.style.transform = 'translate(-50%, 30%)'
            document.body.appendChild(image);

            setTimeout(() => {
            image.style.transition = 'opacity 0.5s';
            image.style.opacity = '0';
            setTimeout(() => {
                document.body.removeChild(image);
            }, 500);
            }, 1000);
        },
        startGame() {
            this.running = true
            this.gameover = false
        },
        generateChar() {
            // 生成一个随机字符
            const char = String.fromCharCode(Math.floor(Math.random() * 26) + 65)

            const speed = this.point * 0.1 + 1
            const left = .1 * window.innerWidth
            const range = .8 * window.innerWidth
            // 将生成的字符添加到chars数组中
            this.chars.push({
                id: this.chars.length + 1,
                letter: char,
                position: {
                    x: left + Math.random() * range,
                    y: 0
                },
                speed: speed
            })
        },

        fall() {
            const that = this
            if(that.running) {
                // 更新字母的位置和速度
                for (let i = 0; i < this.chars.length; i++) {
                    const char = this.chars[i]
                    char.position.y += char.speed

                    // 判断字母是否触底
                    if (char.position.y + this.charSize >= window.innerHeight) {
                        // 字母触底爆炸
                        that.chars.splice(i, 1)
                        i--
                        that.miss++
                        that.createImage(char.position.x)
                        that.createAudio()
                }
                }
            }
            

            // 使用requestAnimationFrame循环调用fall方法
            requestAnimationFrame(this.fall)
        },
        handleKeyUp() {
            this.$refs.mainPanel.style.setProperty('--bar-visibility', 'hidden')
        },
        handleKeyDown(event) {
            // 获取按下的键盘按键
            const key = event.key

            let minDistance = Infinity;
            let closestCharIndex = -1;

            for (let i = 0; i < this.chars.length; i++) {
                const char = this.chars[i];
                if (key === char.letter.toLowerCase()) {
                    const distanceToBottom = window.innerHeight - char.position.y - this.charSize;
                    if (distanceToBottom <= window.innerHeight * this.barRange && distanceToBottom < minDistance) {
                        minDistance = distanceToBottom;
                        closestCharIndex = i;
                    }
                }
            }

            if (closestCharIndex !== -1) {
                const char = this.chars[closestCharIndex]
                const {position} = char
                this.$refs.mainPanel.style.setProperty('--bar-left', `${position.x}px`)
                this.$refs.mainPanel.style.setProperty('--bar-visibility', 'visible')
                this.chars.splice(closestCharIndex, 1);
                this.point++
            }
        }
    }
}
</script>
  
<style scoped lang="sass">
  .main-panel.started
    .welcome
      top: -100%

  .main-panel
    --bar-visibility: hidden
    --bar-left: 500px
    --char-size: 64px
    --bar-range: 0.1

    height: 100%
    background: url(../assets/20130722175255.jpg)
    user-select: none
    position: relative

  .points
    position: absolute
    top: 20px
    right: 50px
    font-size: 80px
    color: white
    text-shadow: 2px 2px 0 black, -2px 2px 0 black, -2px -2px 0 black, 2px -2px 0 black

  .bar
    position: absolute
    bottom: 0
    height: calc(100vh * var(--bar-range))
    width: 100px
    background: linear-gradient(rgba(255, 255, 255, .1), red)
    left: var(--bar-left)
    z-index: 9999
    visibility: var(--bar-visibility)

  .welcome
    position: absolute
    height: 100%
    top: 0
    bottom: 0
    left: 0
    right: 0
    background: pink
    display: flex
    justify-content: center
    align-items: center
    transition: all 1s

    .start-btn
      color: white
      font-size: 60px
      padding: 10px
      border: 1px solid gray
      background: hsl(200, 100%, 64%)
      border-radius: 10px
      cursor: pointer

  .letter
    position: absolute
    left: 50%
    color: white
    text-shadow: 2px 2px 0 black, -2px 2px 0 black, -2px -2px 0 black, 2px -2px 0 black
    width: 100px
    height: 200px
    display: flex
    justify-content: center
    align-items: center

    img
      width: 100px
      height: 200px
      position: absolute
      top: 0
      left: 0
      
    .char
      font-size: var(--char-size)
      

  </style>