<template>
    <canvas width="600" height="400" @contextmenu.prevent></canvas>

    <div class="configuracao-jogo">
        <div class="caixa-configuracao-jogo">
            <div class="caixa-configuracao-jogo__nivel">
                <select name="" id="nivelOpcoes" @change="escolhaNivel">
                    <option value="1">Fácil</option>
                    <option value="2">Médio</option>
                    <option value="3">Difícil</option>
                </select>
            </div>
            <div class="caixa-configuracao-jogo_comportamento-jogo">
                <button @click="jogarJogo()">
                    <img src="@/img/play.svg" alt="Jogar" title="Jogar">
                    <span>Jogar</span>
                </button>
            </div>
        </div>
        <div class="caixa-configuracao-exibicao">
            <div class="caixa-configuracao-exibicao__pontuacao">
                <p>Pontuação</p>
                <span class="--pontuacao" v-text="pontuacao"></span>
            </div>
            <div class="caixa-configuracao-exibicao__temporizador">
                <p>Temporizador</p>
                <span class="--temporizador" name="temporizador">00: {{ segundos < 10 ? '0' + segundos : segundos }}</span>
            </div>
        </div>
        <div class="container-pontuacao-tabela">
            <table class="pontuacao-tabela">
                <tbody>
                    <tr class="pontuacao-tabela--titulo">
                        <th colspan="2"><b>Tabela de pontos</b></th>
                    </tr>
                    <tr class="pontuacao-tabela--cabecalho">
                        <th>Dificuldade</th>
                        <th>Pontuação</th>
                    </tr>
                     <tr v-if="verifiquePontuacao.length < 1">
                        <td>Fácil</td>
                        <td>0</td>
                    </tr>
                    <tr v-else v-for="(item, pos) in verifiquePontuacao" :key="`pontos--${pos}`">
                        <td v-text="item.nivel"></td>
                        <td v-text="item.pontuacao"></td>
                    </tr>
                </tbody>
            </table>
        </div>

        <div class="caixa-configuracao-exibicao__regras">
            <p class="como-jogar">Como Jogar:</p>

            <p>Para jogar o <strong>Acerte ao Alvo</strong> basta apertar o <strong>Play</strong> ou se quiser uma dificuldade
                maior pode escolher um nível.</p>
            <p>Para pontuar é necessário acertar o alvo em amarelo, a cada ponto seu score é aumentado.</p>
            <p>O timer é iniciado quando o jogo começa e a cada 30 segundos, seu score é gravado e zerado.</p>
            <p>Caso você mude o nível no meio do game o timer e o score são zerados também.</p>
        </div>
    </div>
</template>
<script>
export default {
    name: 'JogoPagina',

    data() {
        return {
            segundos: 30,
            contagemDecrescente: true,
            ehPrimeiroJogo: true,
            ehPrimeiraContagem: true,
            canvas: null,
            pincel: null,
            meuIntervalo: null,
            velocidadeAlvo: 1000,
            pontuacao: 0,
            raio: 10,
            valorX: 0,
            valorY: 0,
            alvoAleatorio: false,
            nivelNomeado: '',
            selecioneNivel: '1',
            verifiquePontuacao: []
        }
    },
    methods: {
        verifiqueNivel() {

            if (this.selecioneNivel == '1') {
                this.nivelNomeado = 'Fácil'

            } else if (this.selecioneNivel == '2') {
                this.nivelNomeado = 'Médio'

            } else {
                this.nivelNomeado = 'Difícil'
            }
        },
        escolhaNivel() {

            const selecioneNivel = document.getElementById('nivelOpcoes')
            const valorSelecionado = selecioneNivel.options[selecioneNivel.selectedIndex].value

            this.selecioneNivel = valorSelecionado

            if (this.selecioneNivel == '0') {
                this.selecioneNivel == '1'
            }

            switch(this.selecioneNivel) {

                case '1':
                    this.velocidadeAlvo = 1000
                    break;
                case '2': 
                    this.velocidadeAlvo = 800
                    break;
                case '3':
                    this.velocidadeAlvo = 600
                    break;
                default:
                    break;
            }
            clearInterval(this.meuIntervalo);
            this.segundos = 0
            this.pontuacao = 0
            this.contagemDecrescente = false 
            this.ehPrimeiroJogo = true
        },
        jogarJogo() {

            console.log("Jogando...");

            if (this.ehPrimeiroJogo) {
                
                this.segundos = 30
                this.contagemDecrescente = true 
                this.alvoAleatorio = true
                this.verifiqueNivel()

                if (this.ehPrimeiraContagem) {
                    this.contagemRegressiva()
                    this.ehPrimeiraContagem = false
                    
                }

                this.canvas.onclick = this.verifiqueSeVoceAcertou
                this.meuIntervalo = setInterval(this.criarAlvo, this.velocidadeAlvo)
                this.ehPrimeiroJogo = false
            }
        },
        contagemRegressiva() {
            setInterval(() => {

                if (this.segundos == 1) {
                    this.verifiquePontuacao.push({
                        pontuacao: this.pontuacao,
                        nivel: this.nivelNomeado
                    })
                    this.pontuacao = 0
                }
                if (this.segundos > 1 && this.contagemDecrescente) {
                    this.segundos -= 1

                } else {
                    this.segundos = 30
                }
            }, 1000)
        },
        verifiqueSeVoceAcertou($event) {

            var x = $event.pageX - this.canvas.offsetLeft
            var y = $event.pageY - this.canvas.offsetTop

            if (x >= this.valorX - this.raio
               && this.valorX <= x + this.raio
               && y >= this.valorY - this.raio
               && y <= this.valorY + this.raio
               && this.contagemDecrescente) {
                
                this.pontuacao += 1
            }
        },
        criarAlvo() {

            if (this.alvoAleatorio) {

                this.limparCanvas()

                var x = Math.round(Math.random() * 600)
                var y = Math.round(Math.random() * 400)

                this.drawCircle(x, y, this.raio + 20, 'blue') // maior circulo
                this.drawCircle(x, y, this.raio + 10, 'red')
                this.drawCircle(x, y, this.raio, 'yellow') // menor circulo 
            }
        },
        limparCanvas() {

            this.pincel.clearRect(0, 0, 600, 400)
            this.pincel.fillStyle = '#222222'
            this.pincel.fillRect(0, 0, 600, 400)
        },
        drawCircle(x, y, raio, cor) {
            this.pincel.fillStyle = cor 
            this.pincel.beginPath()
            this.pincel.arc(x, y, raio, 0, 2 * Math.PI)
            this.pincel.fill()
            this.valorX = x
            this.valorY = y
        },
        criarFerramentas() {

            const $canvas = document.querySelector('canvas')
            const pincel = $canvas.getContext('2d')

            this.canvas = $canvas
            this.pincel = pincel 

            this.pincel.fillStyle = '#343434'
            this.pincel.fillRect(0, 0, 600, 400)
        }
    },
    mounted() {

        this.criarFerramentas()
    }
}
</script>
<style lang="scss">
canvas {
    display: block;
    margin: 0 auto;
    border-radius: 10px;
    box-shadow: 0 0 1em #000;
}
.configuracao-jogo {
    display: block;
    margin: 0 auto;
    max-width: 600px;

    .caixa-configuracao-jogo__nivel,
    .caixa-configuracao-jogo_comportamento-jogo {

        select {
            width: 180px;
            height: 58px;
            padding: 1em;
            text-align: center;
            border-radius: 10px;
            font-size: 1.2em;
            font-family: 'Gluten', cursive;
            background-color: #00a1ff;
            color: #FFFFFF;
            border: none;
            outline: none;
            transition: .3s;
            -webkit-box-sizing: border-box;
            -moz-box-sizing: border-box;
            box-sizing: border-box;
            -webkit-appearance: none;
            -moz-appearance: none;
            background-image: linear-gradient(45deg, transparent 50%, white 50%), linear-gradient(135deg, white 50%, transparent 50%);
            background-position: calc(100% - 26px) calc(1em + 4px), calc(100% - 21px) calc(1em + 4px), calc(100% - 2.5em) 0.5em;
            background-size: 10px 6px, 6px 10px, 1px 1.5em;
            background-repeat: no-repeat;

            &:hover {
                cursor: pointer;
                background-color: #00a1ffbf;
                transition: ease-in-out .3s;
            }

            option {
                background-color: #FFF;
                color: #000;
            }
        }
        button {
            width: 180px;
            height: 58px;
            padding: 1em;
            text-align: center;
            border-radius: 10px;
            font-size: 1.2em;
            font-family: 'Gluten', cursive;
            display: flex;
            place-items: center;
            justify-content: center;
            background-color: #0a8300;
            color: #FFFFFF;
            transition: .3s;
            border: none;
            outline: none;

            &:hover {
                cursor: pointer;
                background-color: #0a8300b5;
                transition: ease-in-out .3s;
            }

            img {
                width: 20px;
                height: auto;
                margin-right: 1em;
            }
        }
    }
    .caixa-configuracao-exibicao__pontuacao,
    .caixa-configuracao-exibicao__temporizador {
        p{
            margin: 0;
            padding-bottom: 1em;
            font-size: 1.325em;
            color: #6e0c0c;
            font-weight: 500;
        }
        span {
            font-size: 1.4em;
            color: #000;
            padding: 8px 1em;
        }
        .--pontuacao {
            color: red;
        }
        .--temporizador{
            border-radius: 10px;
            background-color: #f1f1f1;
            color: #ff7b00;
        }
    }
    .caixa-configuracao-exibicao,
    .caixa-configuracao-exibicao__temporizador {

        p {
            margin: 0;
            padding-bottom: 1em;
            font-size: 1.325em;
            color: #6e0c0c;
            font-weight: 500;
        }

        span {
            font-size: 1.4em;
            color: #000;
            padding: 8px 1em;
        }

        .--pontuacao {
            color: red;
        }
        .--temporizador {
            border-radius: 10px;
            background-color: #f1f1f1;
            color: #ff7b00;
        }
    }
    .container-pontuacao-tabela {
        width: 340px;
        max-height: 220px;
        overflow-y: scroll;
        overflow-x: hidden;
        padding-right: 6px;
        margin: 3em auto 0 auto;

        &::-webkit-scrollbar {
            width: 6px;
        }

        &::-webkit-scrollbar-thumb {
            background-color: #a7a7a7;
            border-radius: 20px;
        }
        .pontuacao-tabela {
            width: inherit;

            td,
            th {
                border: 1px solid #000000;
                border-radius: 5px;
                text-align: center;
                padding: 10px;
                text-align: center;
            }
            tr:nth-child(even) {
                background-color: #e7e7e7;
                color: #000000;
            }

            tr:nth-child(odd) {
                color: #000000;
            }

            tr.pontuacao-tabela--titulo {
                background-color: #460072;
                color: #ffffff
            }
            tr.pontuacao-tabela--cabecalho {
                background-color: #000000;
                color: #ffffff;
            }
        }
    }

    .caixa-configuracao-jogo,
    .caixa-configuracao-exibicao {
        display: grid;
        grid-template-columns: repeat(2, 1fr);
        justify-items: center;
        padding-top: 1.5em;
        text-align: center;
    }

    .caixa-configuracao-exibicao__regras {
        padding: 1.5em 1em 0 1em;

        .como-jogar {
            font-size: 1.5em;
            font-weight: 500;
            box-shadow: #000;
        }
        p {
            font-size: 1em;
            line-height: 1.425em;
        }
    }
}
</style>