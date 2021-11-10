<template>
    <div class="relative h-full w-full flex flex-col text-white h-screen">
        <header class="flex justify-between bg-gray-900 p-4">
            <span>Snake<strong class="text-green-400">Dev</strong></span>
            <button type="button" @click="play">Play (enter)</button>
            <button type="button" @click="reset">Reset (esc)</button>
            <div>
                <span>Pontos: <strong>{{ pontos }}</strong></span>
            </div>
        </header>
        <div id="campo" class="bg-gray-800 border-green-600 border-2 w-full h-full relative">
            <div id="snake" class="bg-green-800">
                <span :style="'top: '+ s.top +'px; left: '+s.left+'px;'" v-for="(s, index) in snake" :key="index"></span>
            </div>
            <span id="logo">Snake<strong class="text-green-400">Dev</strong></span>
        </div>
        <div class="flex justify-center items-center absolute w-full h-full bg-gray-900 flex-col z-10" v-show="dead">
            <span class="text-2xl">Snake<strong class="text-green-400">Dev</strong></span>
            <span class="mt-20 text-5xl text-red-400">You Dead!</span>
            <button type="button" class="mt-20 text-2xl" @click="playAgain">Play again</button>
        </div>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                snake: [],
                timer: 100,
                intervalo: null,
                pontos: 0,
                dead: false
            }
        },
        created () {
            this.reset();
        },
        mounted() {
            window.addEventListener("keyup", function(e) {
                this.comando(e.keyCode);
            }.bind(this));

            document.body.style.overflowY = "hidden";
            document.body.style.overflowX = "hidden";
        },
        methods: {
            play: function (){
                this.reset();
                this.intervalo = setInterval(() => {
                    this.run();
                }, this.timer);
                this.fruit();
                this.dead = false;
            },
            stop: function (){
                clearInterval(this.intervalo);
            },
            reset: function (){
                this.dead = false;
                
                var 
                    left = 50;

                this.snake = [];
                    
                for (let index = 0; index < 5; index++) {
                    left += 25;
                    this.snake.push({
                        comando: (index == 4) ? 'right' : '',
                        top: 50,
                        left: left
                    });
                }
            },
            playAgain: function (){
                if(document.querySelector('.fruits'))
                    document.querySelector('.fruits').remove();
                this.reset();
            },
            comando: function (event){
                switch (event) {
                    case 27:
                        this.reset();
                        break;
                    case 13:
                        this.play();
                        break;
                    case 40:
                        this.snake[this.snake.length - 1].comando = 'bottom';
                        break;
                    case 39:
                        this.snake[this.snake.length - 1].comando = 'right';
                        break;
                    case 38:
                        this.snake[this.snake.length - 1].comando = 'top';
                        break;
                    case 37:
                        this.snake[this.snake.length - 1].comando = 'left';
                        break;
                }
            },
            run: function (){
                this.snake.forEach((element, index) => {
                    var 
                        nextIndex   = index + 1;

                    if(nextIndex < this.snake.length){
                        element.top = this.snake[nextIndex].top;
                        element.left = this.snake[nextIndex].left;
                    }else{
                        switch (element.comando) {
                            case 'top':
                                element.top -= 25;
                                break;
                            case 'right':
                                element.left += 25;
                                break;
                            case 'bottom':
                                element.top += 25;
                                break;
                            case 'left':
                                element.left -= 25;
                                break;
                        }
                        if(this.checkEat())
                            this.eat();

                        this.checkSnake();
                        
                    }
                    
                });
            },
            checkSnake: function (){
                var 
                    campo   = document.getElementById('campo'),
                    top     = 0,
                    left    = 0,
                    bottom  = campo.offsetHeight,
                    right   = campo.offsetWidth,
                    head    = this.snake[this.snake.length - 1];

                if((head.top < top) ||
                    ((head.top + 25) > bottom) ||
                    (head.left < left) ||
                    ((head.left + 25) > right)
                ){
                    this.dead = true;
                    this.stop();
                }
            },
            fruit: function(){
                var 
                    fruits  = document.createElement('span'),
                    campo   = document.getElementById('campo'),
                    height  = campo.offsetHeight - 26,
                    width   = campo.offsetWidth - 26;

                if(document.querySelector('.fruits'))
                    document.querySelector('.fruits').remove();

                fruits.style.top    = this.getRandomInt (height)+'px';
                fruits.style.left   = this.getRandomInt (width)+'px';
                fruits.style.width  = '25px';
                fruits.style.height = '25px';

                fruits.classList.add('fruits', 'absolute', 'flex', 'bg-red-600');
                campo.append(fruits);
            },

            getRandomInt: function (limit) {
                var 
                    num     = 25,
                    random  = Math.random() * limit,
                    res     = Math.round( random / num ) * num;

                return res;
            },

            checkEat: function (){
                var 
                    el1 = document.querySelectorAll('#snake span')[document.querySelectorAll('#snake span').length -1],
                    el2 = document.querySelector('.fruits');

                if(!el1 || !el2)
                    return false;

                return this.isCollide(el1, el2);
            },

            isCollide: function (div1,div2) {
                let d1 = div1.getBoundingClientRect();
                let d2 = div2.getBoundingClientRect();
                let ox = Math.abs(d1.x - d2.x) < (d1.x < d2.x ? d2.width : d1.width);
                let oy = Math.abs(d1.y - d2.y) < (d1.y < d2.y ? d2.height : d1.height);
                return ox && oy;
            },

            eat: function (){
                document.querySelector('.fruits').remove();
                this.snake.unshift({
                    comando: this.snake[0].comando,
                    top: this.snake[0].top,
                    left: this.snake[0].left
                });
                this.fruit();
                this.pontos += 100;
            }

        }
    }
</script>
<style scoped>
    #snake span {
        display: flex;
        width: 25px;
        height: 25px;
        position: absolute;
        background-color: green;
        border: 1px solid rgb(1, 156, 1);
        z-index: 1;
    }

    #logo{
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        font-size: 15rem;
        opacity: .1;
    }
</style>

