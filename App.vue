<template>
  <div class='mainContainer fog'>
    <div class='mainContainer_phenomenon'>
        <canvas id="bg-img" width='900' height='400'></canvas>
        <div class="fog__img"></div>
        <div class="fog__img"></div>
        <div class='cloud'></div>
    </div>
      <div class='frontendPart'>
        <div class='inputCity'>
            <input type='text' placeholder='Введите город' @click='city=""' @keydown='enter($event)' v-model='city' id='enterCity'>
            <div class='inputCity__btn' @click='getWeather'></div>
        </div>
        <div class='dateInfo'>
            <div class='dateInfo__time' id='time'>{{time}}</div>
            <div class='dateInfo__date'>{{date}}</div>
        </div>
        <div class='weatherContainer'>
            <div class='weatherContainer_city'>{{title}}</div>
            <div class='temp_container'>
                <div class='weatherAttributes_container'>
                    <div class='temp_container_label'>Фактическая</div>
                    <div class='weatherContainer_temperature'>{{temperature}}</div>
                </div>
                <div class='weatherAttributes_container'>
                    <div class='temp_container_label'>Ощущается</div>
                    <div class='weatherContainer_temperature'>{{realfeel}}</div>
                </div>
            </div>
            <div class='weatherContainer_phenomenon'>{{phenomenon}}</div>
        </div>
        <div class='weatherAttributes'>
            <div class='weatherAttributes_container'>
                <div class='weatherAttributes_container__info'>{{windspeed}}</div>
                <div class='weatherAttributes_container__label'>Ветер</div>
            </div>
            <div class='weatherAttributes_container'>
                <div class='weatherAttributes_container__info'>{{visibility}}</div>
                <div class='weatherAttributes_container__label'>Видимость</div>
            </div>
            <div class='weatherAttributes_container'>
                <div class='weatherAttributes_container__info'>{{humidity}}</div>
                <div class='weatherAttributes_container__label'>Влажность</div>
            </div>
             <div class='weatherAttributes_container'>
                <div class='weatherAttributes_container__info'>{{pressure}}</div>
                <div class='weatherAttributes_container__label'>Давление</div>
            </div>
        </div>
    </div>
    <div class='footer'>
        <div>
            <span>© 2020 VelumWeb - </span>
            <a href = 'https://tlgg.ru/Maxorik'>@Maxorik</a>
        </div>
    </div>
  </div>
</template>

<script>  
    
function setnull(num){
    if(num.toString().length<2){
        return '0' + num
    }   
    else{
        return num;
    }
}
    
function setTime(){
    let t = setTimeout(function testtime(){
        let today = new Date();
        time.innerHTML = setnull(today.getHours()) + ':' + setnull(today.getMinutes());
        t = setTimeout(testtime, 1000); 
    },1000)
}
    
export default {
    name: 'app',
    data(){
        return{ 
            city:'',
            jsonWeather:{},
            temperature:'',
            phenomenon:'',
            pressure:'',
            windspeed:'',
            visibility:'',
            humidity:'',
            title:'',
            realfeel:'',
            phenomenonType:'snow',
            phenomenonIntense: 200,
            phenomenonSpeed: 150
        }
    },
    computed:{
        date(){
            let today = new Date();
            let month=['января','февраля','марта','апреля','мая','июня','июля','августа','сентября','октября','ноября','декабря']
            return today.getDate() + ' ' + month[today.getMonth()] + ' ' + today.getFullYear() + ' г.';
        },
        time(){
            let today = new Date();
            return setnull(today.getHours()) + ':' + setnull(today.getMinutes());
        }
    },
    methods:{
        getWeather(){
            let city=this.city;
            localStorage.setItem('city', this.city);
            let cite = 'https://api.worldweatheronline.com/premium/v1/weather.ashx?key=a4e1dfcb895c46bf8aa63725202501&q=';
            let citeend = '&format=json&num_of_days=1&lang=ru&timezone=yes';
            if(this.city == 'москва'){
                city = 'moscow';
            }
            
            if(this.city.indexOf('анкт', 0)!= -1){
                city = 'Saint-Petersburg';
                this.city = 'Санкт-Петербург';
            }
            fetch(cite + city + citeend)
            .then(res => res.ok ? res : Promise.reject(res))
            .then(res => {
                return res.json();
            }).then(this.setResults)
            .catch(function(){
                this.clear();
            });
        },
        
        setResults (results) {
            if(!results.data.error){
                this.jsonWeather = results;
                this.temperature = this.jsonWeather.data.current_condition[0].temp_C + '°';
                this.phenomenon = this.jsonWeather.data.current_condition[0].lang_ru[0].value;
                this.pressure = this.jsonWeather.data.current_condition[0].pressure + 'mb';
                this.windspeed = parseInt(+this.jsonWeather.data.current_condition[0].windspeedKmph / 3.6) + 'м/с';
                this.visibility = this.jsonWeather.data.current_condition[0].visibility + 'км';
                this.humidity = this.jsonWeather.data.current_condition[0].humidity + '%';
                this.realfeel = parseInt(37-((37-this.jsonWeather.data.current_condition[0].temp_C)/(0.68-0.0014*this.jsonWeather.data.current_condition[0].humidity+(1/(1.76+1.4*Math.pow((this.jsonWeather.data.current_condition[0].windspeedKmph/3.6/3.6), 0.75)))))-0.29*this.jsonWeather.data.current_condition[0].temp_C*(1-(this.jsonWeather.data.current_condition[0].humidity/100))) + '°';
                this.title = this.toTitle(this.city);
                this.setPhenomenon(this.phenomenon);
            }
        },
        
        delPhenomenon(){
            let mainContainer = document.querySelector('.mainContainer');
            let cloud = document.querySelector('.cloud');
            mainContainer.firstElementChild.childNodes[0].classList.remove('fog__img1');
            mainContainer.firstElementChild.childNodes[2].classList.remove('fog__img2');
            cloud.innerHTML='';
            this.phenomenonType = 'nothing';
            this.phenomenonIntense = 1;
        },
        
        setPhenomenon(p){
            let mainContainer = document.querySelector('.mainContainer');
            let rain = document.querySelector('#bg-img');
            this.delPhenomenon();
            
            if(p.indexOf('дождь', 0)!= -1){
                rain.style.display = 'block';
                this.phenomenonType = 'rain';
                if(p.indexOf('ильный', 0)!= -1){
                    this.phenomenonIntense = 400;
                }
                else if(p.indexOf('ебольшой', 0)!= -1 ){
                    this.phenomenonIntense = 100;
                }
                else{
                    this.phenomenonIntense = 200;
                }
            }
            
            else if(p.indexOf('снег', 0)!= -1){
                rain.style.display = 'block';
                this.phenomenonType = 'snow';
                if(p.indexOf('ильный', 0)!= -1){
                    this.phenomenonIntense = 200;
                }
                else if(p.indexOf('ебольшой', 0)!= -1 ){
                    this.phenomenonIntense = 50;
                    this.phenomenonSpeed=90;
                }
                else{
                    this.phenomenonIntense = 100;
                    this.phenomenonSpeed=100;
                }
            }
            
            else if(p.indexOf('уман', 0)!= -1 || p.indexOf('ымка', 0)!= -1){
                mainContainer.firstElementChild.childNodes[0].classList.add('fog__img1');
                mainContainer.firstElementChild.childNodes[2].classList.add('fog__img2');
            }
            
            else if(p.indexOf('блачн', 0)!= -1){
                rain.style.display = 'none';
                let cloud = document.querySelector('.cloud');
                if(window.navigator.userAgent.toLowerCase().includes("mobi")){
                    cloud.innerHTML = '<img src="./assets/img/cloud-02-mobile.png" class="cloud2"><img src="./assets/img/cloud-04-mobile.png" class="cloud4">';
                }
                else cloud.innerHTML = '<img src="./assets/img/cloud-02.png" class="cloud2"><img src="./assets/img/cloud-04.png" class="cloud4">';
            }
            
            else if(p.indexOf('асмурн', 0)!= -1){
                rain.style.display = 'none';
                let cloud = document.querySelector('.cloud');
                if(window.navigator.userAgent.toLowerCase().includes("mobi")){
                    cloud.innerHTML = '<img src="./assets/img/graycloud-mobile.png" class="cloud2">';
                }       
                else cloud.innerHTML = '<img src="./assets/img/graycloud.png" class="cloud2">';
            }
            
            else{
                this.phenomenonType = 'nothing';
            }
                    
            this.callPhenomenon();
        },
        
        callPhenomenon(){
            let c = document.getElementById("bg-img");
            let n = this.phenomenonIntense;
            let ctx = c.getContext("2d");
            ctx.clearRect(0, 0, c.width, c.height);
            let cw = (c.width = c.offsetWidth);
            let ch = (c.height = 500);
            let duration = ch/this.phenomenonSpeed;
            let img = new Image();
            let particles = [];
            let particleNumber = 0;
                
            let Particle = function() {
                this.alpha = rand(0.1, 0.4);
                this.size = rand(20, 30);
                this.draw = function() {
                    ctx.globalAlpha = this.alpha;
                    if(window.navigator.userAgent.toLowerCase().includes("mobi")){
                        ctx.drawImage(img, this.x, this.y, this.size, this.size/2);
                    }
                    else ctx.drawImage(img, this.x, this.y, this.size, this.size);
                }
            };
            
            function setParticle(p) {
                particleNumber++;
                let _dur = rand(duration/2, duration),
                _tl = new TimelineMax()
                .fromTo(p, _dur, {
                    x:rand(-p.size, cw+ch/2),
                    y:-p.size
                    },
                {
                    x:'-='+rand(ch/2-50,ch/2), 
                    y:ch+p.size,
                    ease:Power0.easeNone,
                    onComplete:function(){ setParticle(p); }
                });
                
                if (particleNumber<n) _tl.seek(_dur*rand());
            }
            
            
            for (let i=0; i<n; i++) {
                particles.push(new Particle());
                setParticle(particles[i]);      
            }
            
            TweenMax.ticker.addEventListener("tick", function(){
                ctx.clearRect(0, 0, cw, ch);
                for (let i=0; i<n; i++) {
                    particles[i].draw();
                }
            });
            
            window.addEventListener('resize', function() {
                particleNumber = 0;  
                cw = (c.width = c.offsetWidth);
                ch = (c.height = 500);
                for (let i=0; i<n; i++) {
                    TweenMax.killTweensOf(particles[i]);
                    setParticle(particles[i]);
                }
            });
            
            function rand(min=0, max=1) {
                return min + (max-min)*Math.random();
            }
            
            img.src = './assets/img/'+this.phenomenonType+'.png';
        },
        
        toTitle(str){
            str = str.toLowerCase();
            let f = str.charAt(0).toUpperCase();
            return f + str.slice(1, str.length);
        },
        
        enter(e){
            if(e.key == 'Enter'){
                this.hideKeyboard(enterCity);
                this.getWeather();
            }
        },
        
        hideKeyboard(element) {
            element.setAttribute('readonly', 'readonly'); 
            element.setAttribute('disabled', 'true'); 
            setTimeout(function() {
                element.blur();  
                element.removeAttribute('readonly');
                element.removeAttribute('disabled');
            }, 100);
        },
        
        clear(){
            this.city = '';
            this.temperature = '';
            this.phenomenon = '';
        }
    },
    
    created(){
        this.city = localStorage.getItem('city');
        if (YMaps.location){
            this.city = YMaps.location.city;
        }
        if(this.city){
            this.getWeather()
        }
        setTime();
    }
}
</script>

<style lang='scss'>
    @import './scss/toWeather/fog.scss';
    @import './scss/toWeather/cloud.scss';
    $textcolor: #ecd5c9;
    $inputColor: #fac4e0;
    
    .mainContainer{
        margin: 0;
        padding: 0;
        width: 100%;
        height: 100vh;
        display: flex;
        flex-direction: column;
        align-items: center;
        background: url(img/city3.jpg);
        background-attachment: fixed;   
        background-repeat: no-repeat;
        background-size:cover;
        position: relative;
    }
    .frontendPart{
        position: absolute;
        top:0;
    }
    .inputCity{
        width:90%;
        max-width: 600px;
        position: relative;
        margin: 20px auto;
        #enterCity{
            width:100%;
            border-radius: 10px;
            padding: 1px 10px;
            line-height: 2em;
            font-size: 1.6em;
            background-color: $inputColor;
            font-weight: 400;
            border: 1px solid $inputColor;
            opacity: 0.7;
        }
        .inputCity__btn{
            cursor: pointer; 
            width:45px;
            height: 25px;
            background: url(img/cloud_btn.png);
            background-size: cover;
            position: absolute;
            top: 30%;
            right:2%;
        }
    }
    
    .dateInfo, .weatherContainer{
        font-size: 2em;
        font-weight: bold;
        text-shadow: #000 1px 1px 0, #000 -1px -1px 0, #000 -1px 1px 0, #000 1px -1px 0;
        color:$textcolor;
        text-align: center;
    }
    
    .temp_container{
        display: flex;
        align-items: center;
        justify-content: space-around;
        flex-direction: row;
        .weatherAttributes_container{
            display: flex;
            flex-direction: column;
            .temp_container_label{
                font-size: 0.7em;
                margin: 0;
            }
        }
    }
    
    
    .weatherContainer{
        .weatherContainer_temperature{
            font-size: 3em;
            background-color:$inputColor;
            border-radius: 20px;
            opacity: 0.7;
            width: 16vw;
            margin: auto;
        }
    }
    
    .weatherAttributes{
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        .weatherAttributes_container{
            margin: 10px;
            .weatherAttributes_container__info{
                display: flex;
                align-items: center;
                justify-content: center;
                font-size: 1.6em;
                font-weight: bold;
                text-shadow: #000 1px 1px 0, #000 -1px -1px 0, #000 -1px 1px 0, #000 1px -1px 0;
                color:$textcolor;
                border-radius: 100%;
                width:8vw;
                height:8vw;
                background-color:$inputColor;
                opacity: 0.7;
            }   
            .weatherAttributes_container__label{
                font-size: 1em;
                text-shadow: #000 1px 1px 0, #000 -1px -1px 0, #000 -1px 1px 0, #000 1px -1px 0;
                color:$textcolor;
                text-align: center;
            }
        }
    }
    
    .footer{
        position: absolute;
        bottom: 10px;
        font-size: 0.8em;
        text-shadow: #000 1px 1px 0, #000 -1px -1px 0, #000 -1px 1px 0, #000 1px -1px 0;
        color:$textcolor;
        left: 5px;
        opacity: 0.7;
        a{
            color:aquamarine;
            text-decoration: none;
            &:hover{
                color: #93ff64;
            }
        }
    }
    
    #bg-img{
        width: 99vw;
        height: 99vh;
    }
    
    @media screen and (max-width: 1110px){
        .mainContainer{
            background: url(img/city4.jpg);
            background-attachment: fixed;   
            background-repeat: no-repeat;
            background-size:cover;
            position: relative;
        }
        .inputCity{
            max-width: 76vw;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 0;
            margin-top: 20px;
            #enterCity{
                line-height: 2em;
                font-size: 2.6em;
            }
            .inputCity__btn{
                width:70px;
                height: 40px;
                top: 30%;
                right:3%;
            }
        }
         .dateInfo, .weatherContainer{
            font-size: 4em;
        }
        .weatherContainer{
            .weatherContainer_temperature{
                font-size: 3em;
                line-height: 1em;
                width: 40vw;
                opacity: 0.7;
            }
        }
        .footer{
            font-size: 1.8em;
            text-align: center;
        }
        
        .cloud{
            height: 100vh;
            width: 250vw;
        }
        
        .weatherAttributes{
            .weatherAttributes_container{
                .weatherAttributes_container__info{            
                    width:20vw;
                    height:20vw;
                    font-size: 3em;
                    background-color:$inputColor;
                    opacity: 0.7;
                }   
                .weatherAttributes_container__label{
                    font-size: 2.2em;
                }
            }
        }
    }
</style>


