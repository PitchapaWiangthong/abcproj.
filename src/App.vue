<template>
  <div>
    <!-- When in naming State -->
    <div v-if="state === 'nameState'">
      <div>
        
      </div>
        <div class="learn-more">
          <input type="text" v-model="level" placeholder="Enter level">
          <input type="text" v-model="whatword" placeholder="Enter word num">
            <button @click="startState()" class="learn-more" type="button">Start</button>
        </div>
    </div>

    <!-- When in running State -->
    <div v-if="state === 'runState'"> 
        <div>
            <timer @takeEnd="takeEnd" @takeDifficult="takeDifficult"  v-bind:start-app="startApp"  v-bind:firstall='all'/>
        </div>
        <!-- display Real text -->
        <div v-bind:style="styleObject">
          
            <div id="para" style="display: inline; top: 0px; width: 100% ;  "></div>
        </div>
        <!-- display Temp text -->
        <div style="text-align: center; position:relative" >
            <div v-if="!isPlay">
              <p id="plus" style="display: inline ; top: 0px; width: 100%"></p>
            </div>
            <div id="tempText" style="display: inline ; top: 0px; width: 100%"></div> 
        </div>
        
        
        
        <div v-if="isRecord && !isPlay" style="position: relative; margin: 0 auto; text-align: center">
            <img class = "image-style" src="https://www.flaticon.com/svg/vstatic/svg/945/945185.svg?token=exp=1617564095~hmac=14fe0a401d86eaa536b3213040afa148" >
        </div>
    </div>

    <!-- When in end State -->
    <div v-if="state ==='endState'" >
        <p class="popout">
          <span>E</span><span>N</span><span>D</span><span>I</span><span>N</span><span>G</span>
        </p>
        <div class="wrap">
            
                <a href="#" class="underlined underlined--reverse">
                  "{{studentName}}" got {{arrayPool.length}}
                </a>
            
        </div>  
    </div>


  </div>
</template>

<script>
import json from  "@/assets/letter/level0.json";
import json2 from "@/assets/letter/level1.json";
import json3 from "@/assets/letter/level2.json";
import json4 from "@/assets/letter/level3.json";
import json5 from "@/assets/letter/level4.json";
import json6 from "@/assets/letter/level5.json";
import json7 from "@/assets/letter/level6.json";
import json8 from "@/assets/letter/level7.json";
import json9 from "@/assets/letter/level8.json";
import json10 from "@/assets/letter/level9.json";



import timer from "@/components/layout/timer.vue";

export default {
  name: "app",
  components: {
    timer,
  },

  data() {
    return {
      level:null,
      whatword:null,

      all:1800,
      currentDifficult:1,
      difficult: 1,
      state: "nameState",
      studentName: '',
      myTrack: new Audio(),
      letter: json.level4,     //Import json file
      i: 0,                     //This integer for what number of alphabet in word              
      j: 0,                     //This integer for what number of word
      previusJ:0,
      thisString: [],           //this is a main string of word
      coloredString: [],        //this is a color string of word
      isStop: false,            //this boolean will turn to true when finish render 1 word
      isPlay: false,            //this boolean will turn to false when user press keyboard   
      isRecord: false,
      isEnd: false,
      arrayPool: [],            //Use to collect rendered word
      startApp:false,           //Boolean for sent data back to timer
      currentString: "",

      //Text style 
      fontSize:300,             //text size
      styleObject: {
        top: '80px',
        color: 'blue',
        position: 'fixed',
        left: '10px', 
      },
    };
  },

  methods: {
    takeEnd(value){
        this.isEnd = value;
        this.state = "endState"
    },
    takeDifficult(value){
        this.difficult = value;
        console.log(this.difficult)
    },
    showSliderValue() {
      
    },
    //To get random number from word pool
    random(min,max,arrayPool) {
    var i = Math.floor(Math.random()*(max-min))+min;
      for (let j = 0 ; j < arrayPool.length ; ){
          if(i === arrayPool[j]){
              i = Math.floor(Math.random()*(max-min))+min;
              j = 0;
          }
          else{
              j++;
          }
          
      }
      arrayPool.push(i);
      return i;
    },
    // calculate percentage
    percent(input , percent){
      return input*percent/100
    },
    //Core function
    playSound() {
      this.makeString();
      if (this.letter[this.j][this.i].source !== "") {            //If there have source in file
        this.isPlay = true;
        this.isStop = false;
        this.myTrack = new Audio();
        if(!this.letter[this.j][this.i].source ){
          this.myTrack.src = "consonant/กอ.mp3"
        }
        else{
          
          this.myTrack.src = this.letter[this.j][this.i].source; 
        }
        
           //path


        this.myTrack.play();
        this.myTrack.addEventListener(                            //when *1* alphabet ended
          "ended",
          function() {
            this.i++;
            if (this.i > this.letter[this.j].length - 1) {        //If end of 1 word 
              this.i = 0;
              this.thisString = [];                               //Reset both string and color
              this.coloredString = [];
              this.isPlay = false;
              this.isStop = true;
              console.log(this.currentString)
              console.log(this.arrayPool)
              this.state = 'nameState'
            }
            if (!this.isStop) {
              setTimeout(()=>{
                this.playSound();
              },1)
            }
          }.bind(this)
        );
      } 
      else {
        this.i++;
        if (!this.isStop) this.playSound();
      }
    },

    //make String for position
    makeTempString(){
      var tempText = document.getElementById("tempText");
      var invisString = [];
      var lastInvis = [];
      //******This for invisiblestring*******
      for(let i = 0 ; i< this.letter[this.j].length ; i++){     //making array of invisible string
        if (this.letter[this.j][i].swap === "0") {            
          if (this.letter[this.j][i].char !== "") {
            invisString.push(this.letter[this.j][i].char);
          }
        } 
        else if (this.letter[this.j][i].swap !== "0") {
          if (this.letter[this.j][i].char !== "") {
            invisString = this.swap(
              invisString,
              this.letter[this.j][i].char,
              this.letter[this.j][i].swap
            );
            
          }
        }
      }
      for (let index = 0; index < invisString.length; index++) {    //makeing array of html text 
        
          let lastInvis1 =
            "<strong class='strongID'  style='font-size:"+(this.percent(this.fontSize , 100.5 )) +"px; opacity:0% ; font-weight: 401 !important; color: " +
            this.coloredString[index] +
            ";'>" +
            invisString[index] +
            "</strong>";
          lastInvis.push(lastInvis1);
        
      }
      tempText.innerHTML = lastInvis.join("")
      this.currentString = invisString.join("")
      this.styleObject.left = tempText.offsetLeft + "px"
        

    },
    
    //main fucntion for making string
    makeString: function() {
      var para = document.getElementById("para");
      
      //******This for RealString*******
      var lastString = [];
      if (this.letter[this.j][this.i].swap === "0") {
        if (this.letter[this.j][this.i].char !== "") {
          this.thisString.push(this.letter[this.j][this.i].char);
          this.coloredString.push(this.letter[this.j][this.i].color);
        }
      } else if (this.letter[this.j][this.i].swap !== "0") {
        if (this.letter[this.j][this.i].char !== "") {
          this.thisString = this.swap(
            this.thisString,
            this.letter[this.j][this.i].char,
            this.letter[this.j][this.i].swap
          );
          this.coloredString = this.swap(
            this.coloredString,
            this.letter[this.j][this.i].color,
            this.letter[this.j][this.i].swap
          );
        }
      }
      for (let index = 0; index < this.thisString.length; index++) {

        //ถ้าตัวนั้นเป็นวรรณยุกต์
        if (this.coloredString[index] == "blue") {
            if( this.thisString[index-1] === 'ั' ||
                this.thisString[index-1] === 'ิ' ||
                this.thisString[index-1] === 'ี' ||
                this.thisString[index-1] === 'ึ' || 
                this.thisString[index-1] === 'ื' ){

                if( this.thisString[index] === "๋" ||
                    this.thisString[index] === "่" ||
                    this.thisString[index] === "้" ||
                    this.thisString[index] === "๊" ){  //ถ้าเกิดเป็นไม้จัตวา คงต้องให้ด้านหลังมันขยับมาด้านหน้านิสนึง
                    if( this.thisString[index-2] === 'ป' || //แล้วถ้าข้างหน้าแม่งเจือกเป็น ป ฟ ฝ ฬ
                        this.thisString[index-2] === 'ฟ' ||
                        this.thisString[index-2] === 'ฝ' ||
                        this.thisString[index-2] === 'ฬ' ){
                      let lastStrings =
                        "<sup class='supID2' style='font-size:"+(this.percent(this.fontSize , 85 )) +"px; color: " +
                        this.coloredString[index] +
                        ";  '>" +
                        this.thisString[index] +
                        "</sup>";
                      lastString.push(lastStrings);
                    }
                    else{
                      let lastStrings =
                        "<sup class='supID' style='font-size:"+(this.percent(this.fontSize , 85 )) +"px; color: " +
                        this.coloredString[index] +
                        ";  '>" +
                        this.thisString[index] +
                        "</sup>";
                      lastString.push(lastStrings);

                    }
                }
                else{
                  let lastStrings =
                    "<sup class='supID' style='font-size:"+(this.percent(this.fontSize , 85 )) +"px; color: " +
                    this.coloredString[index] +
                    ";  '>" +
                    this.thisString[index] +
                    "</sup>";
                  lastString.push(lastStrings);
                }
              
            }
            else if(  this.thisString[index-1] ==='ป' ||
                      this.thisString[index-1] ==='ฝ' ||
                      this.thisString[index-1] ==='ฬ' ||
                      this.thisString[index-1] ==='ฟ'
                ){
                  if(this.thisString[index] === '๋'){
                    let lastStrings =
                    "<b class='bID3' style='font-size:"+(this.percent(this.fontSize , 96.25 )) +"px; color: " +
                    this.coloredString[index] +
                    "; font-weight: 389 ;'>" +
                    this.thisString[index] +
                    "</b>";
                    lastString.push(lastStrings);
                  }
                  else{
                    let lastStrings =
                    "<b class='bID2' style='font-size:"+(this.percent(this.fontSize , 96.25 )) +"px; color: " +
                    this.coloredString[index] +
                    "; font-weight: 389 ;'>" +
                    this.thisString[index] +
                    "</b>";
                    lastString.push(lastStrings);
                  }
            }
            else if(this.thisString[index] ==='๋'){
                console.log("hehehheheh")
                let lastStrings =
                "<b class='bID5' style='font-size:"+(this.percent(this.fontSize , 96.25 )) +"px; color: " +
                this.coloredString[index] +
                "; font-weight: 389 ;'>" +
                this.thisString[index] +
                "</b>";
                lastString.push(lastStrings);
            }
            else{
                let lastStrings =
                "<b class='bID' style='font-size:"+(this.percent(this.fontSize , 96.25 )) +"px; color: " +
                this.coloredString[index] +
                "; font-weight: 389 ;'>" +
                this.thisString[index] +
                "</b>";
                lastString.push(lastStrings);
            }
          
        } 


        //เช็คสำหรับตัวอักษรนั้นๆเป็นสระ
        else if (this.coloredString[index] == "green") {
          if( this.thisString[index] === 'ุ' ||            //สระอุ
              this.thisString[index] === 'ู'){             //สระอุ
              let lastStrings =
                "<span class='spanID' style='font-size:"+(this.percent(this.fontSize , 100 )) +"px; color: " +
                this.coloredString[index] +
                ";'>" +
                this.thisString[index] +
                "</span>";
              lastString.push(lastStrings);
          }
          else if(  this.thisString[index-1] ==='ป' ||   //ไว้ดักตอนที่ข้างหน้าเป็น ป ฝ ฟ ฬ
                    this.thisString[index-1] ==='ฝ' ||
                    this.thisString[index-1] ==='ฬ' ||
                    this.thisString[index-1] ==='ฟ'){
              if(  this.thisString[index] === "ิ" || this.thisString[index] === "ี" ||
                   this.thisString[index] === "ึ" || this.thisString[index] === "ื" ||
                   this.thisString[index] === "ั" || this.thisString[index] === "่" || 
                   this.thisString[index] === "้" || this.thisString[index] === "๊" || 
                   this.thisString[index] === "๋"  ){
                     if(  this.thisString[index+1] === "๋" ||
                          this.thisString[index+1] === "่" ||
                          this.thisString[index+1] === "้" ||
                          this.thisString[index+1] === "๊" ){
                        let lastStrings =
                          "<b class='bID4' style='font-size:"+(this.percent(this.fontSize , 100 )) +"px; color: " +
                          this.coloredString[index] +
                          ";'>" +
                          this.thisString[index] +
                          "</b>";
                        lastString.push(lastStrings);
                     }
                     else{
                        let lastStrings =
                          "<b class='bID2' style='font-size:"+(this.percent(this.fontSize , 100 )) +"px; color: " +
                          this.coloredString[index] +
                          ";'>" +
                          this.thisString[index] +
                          "</b>";
                        lastString.push(lastStrings);
                     }
              }
              else{
                let lastStrings =
                  "<b class='bID' style='font-size:"+(this.percent(this.fontSize , 100 )) +"px; color: " +
                  this.coloredString[index] +
                  ";'>" +
                  this.thisString[index] +
                  "</b>";
                lastString.push(lastStrings);
              }

          }
          else if(this.thisString[index+1] === "ุ" || this.thisString[index+1] === "ู"
          || this.thisString[index+1] === "ิ" || this.thisString[index+1] === "ี"
          || this.thisString[index+1] === "ึ" || this.thisString[index+1] === "ื"
          || this.thisString[index+1] === "่" || this.thisString[index+1] === "้"
          || this.thisString[index+1] === "๊" || this.thisString[index+1] === "๋")
          {
            let lastStrings =
              "<b style='font-size:"+(this.percent(this.fontSize , 100 )) +"px; color: " +
              this.coloredString[index] +
              ";'>" +
              this.thisString[index] +
              "</b>";
            lastString.push(lastStrings);
          }
          else{
            let lastStrings =
              "<b class='bID' style='font-size:"+(this.percent(this.fontSize , 100 )) +"px; color: " +
              this.coloredString[index] +
              ";'>" +
              this.thisString[index] +
              "</b>";
            lastString.push(lastStrings);
          }
        } 


        //เช็คสำหรับตัวอักษรนั้นๆเป็นพยัญชนะ,ตัวสะกด
        else {
          //ถ้าเกิดตัวที่อยู่ข้างหน้าเป้นไม้จัตวาหละ
          if( this.thisString[index+1] === "๋"
          ){
            if( this.thisString[index] ==='ป' ||
                this.thisString[index] ==='ฝ' ||
                this.thisString[index] ==='ฬ' ||
                this.thisString[index] ==='ฟ'
                ){
                  let lastStrings =
                    "<strong class='strongID4' style='font-size:"+(this.percent(this.fontSize , 100.5 )) +"px; font-weight: 401 !important; color: " +
                    this.coloredString[index] +
                    ";'>" +
                    this.thisString[index] +
                    "</strong>";
                  lastString.push(lastStrings);

                }
                else{
                  
                  let lastStrings =
                    "<strong class='strongID5' style='font-size:"+(this.percent(this.fontSize , 100.5 )) +"px; font-weight: 401 !important; color: " +
                    this.coloredString[index] +
                    ";'>" +
                    this.thisString[index] +
                    "</strong>";
                  lastString.push(lastStrings);
                }
          }
          else if( //ถ้าเกิดตัวด้านหน้าเป็นตัวที่อยู่ด้านบน หรืออยู่ด้านล่าง
              this.thisString[index+1] === "้" ||
              this.thisString[index+1] === "่" ||
              this.thisString[index+1] === "๊" ||
              this.thisString[index+1] === "ิ" ||
              this.thisString[index+1] === "ี" ||
              this.thisString[index+1] === "ึ" ||
              this.thisString[index+1] === "ื" ||
              this.thisString[index+1] === "ั" ||
              this.thisString[index+1] === "็"
              ){
                if( this.thisString[index] ==='ป' ||  //แล้วถ้าตัวนนั้นๆเสือกเป็นพยัญชนะยาวๆ ก็ให้ระยะห่างมันติดลบสะหน่อย
                    this.thisString[index] ==='ฝ' ||
                    this.thisString[index] ==='ฬ' ||
                    this.thisString[index] ==='ฟ'
                  ){
                  let lastStrings =
                    "<strong class='strongID3' style='font-size:"+(this.percent(this.fontSize , 100.5 )) +"px; font-weight: 401 !important; color: " +
                    this.coloredString[index] +
                    ";'>" +
                    this.thisString[index] +
                    "</strong>";
                  lastString.push(lastStrings);

                }
                else{                                   //แต่ถ้าเป็นตัวอักษรปกติที่ไม่ใช่ ป ฟ ฝ ฬ ก็ให้มันระยะห่างเท่ากะเพื่อนคือ 0 px
                  let lastStrings =
                    "<strong class='strongID2' style='font-size:"+(this.percent(this.fontSize , 100.5 )) +"px; font-weight: 401 !important; color: " +
                    this.coloredString[index] +
                    ";'>" +
                    this.thisString[index] +
                    "</strong>";
                  lastString.push(lastStrings);
                }

          }
          else if(this.thisString[index+1] === "ุ" || 
              this.thisString[index+1] === "ู"){
                let lastStrings =
                    "<strong class='strongID2' style='font-size:"+(this.percent(this.fontSize , 100.5 )) +"px; font-weight: 401 !important; color: " +
                    this.coloredString[index] +
                    ";'>" +
                    this.thisString[index] +
                    "</strong>";
                  lastString.push(lastStrings);
          }

          else{ //ถ้าเกิดไอพวกตัวด้านหลังเป็นพยัญชนะโง่ๆ ก็ให้มันขยับไป 40 px 
            let lastStrings =
              "<strong class='strongID' style='font-size:"+(this.percent(this.fontSize , 100.5 )) +"px; font-weight: 401 !important; color: " +
              this.coloredString[index] +
              ";'>" +
              this.thisString[index] +
              "</strong>";
            lastString.push(lastStrings);
            
          }
        }
      }
      para.innerHTML = lastString.join("");
    },
    //this function use to swap array of string
    swap(thisstring, character, num) {
      thisstring.splice(thisstring.length - num, 0, character);
      return thisstring;
    },
    
    //Call this function when want to start to run
    runningFunction(){
      if     (this.level == 1){
        this.letter = json.level0;
      }
      else if(this.level == 2){
        this.letter = json2.level1;
      }
      else if(this.level == 3){
        this.letter = json3.level2;
      }
      else if(this.level == 4){
        this.letter = json4.level3;
      }
      else if(this.level == 5){
        this.letter = json5.level4;
      }
      else if(this.level == 6){
        this.letter = json6.level5;
      }
      else if(this.level == 7){
        this.letter = json7.level6;
      }
      else if(this.level == 8){
        this.letter = json8.level7;
      }
      else if(this.level == 9){
        this.letter = json9.level8;
      }
      else if(this.level == 10){
        this.letter = json10.level9;
      }
      
      if(this.currentDifficult !== this.difficult){
        this.arrayPool = [];
        this.currentDifficult = this.difficult
      }

      if(this.state === 'endState'){
        var para = document.getElementById("endScore");
        para.innerHTML = "sads";
      }
      if(!this.startApp) this.startApp = true //To return that game start
            if(this.isRecord === true){
              this.isRecord = false;
            }
            if(!this.isPlay){
              
              var para = document.getElementById("plus");
              var para2 = document.getElementById("para");
              var para3 = document.getElementById("tempText");
              this.thisString = []  
              this.coloredString= [],
              para.innerHTML = "<p style='font-size:"+ (this.percent(this.fontSize, 120))+"px ;color:black ;'>+</p>";
              para2.innerHTML = "";
              para3.innerHTML = "";
              this.j= this.whatword
              this.previusJ = this.j
            setTimeout(()=>{
            if (!this.isPlay) {
                this.makeTempString()
                this.playSound();
              }
            },2000)
            }
    },
    
    turnRecord(){
      if(!this.isPlay && this.isRecord === false){
            this.isRecord = true;
        }
    },

    startState(){
      this.state = 'runState' ; 
      setTimeout(()=>{
            this.turnRecord() ; 
            this.runningFunction();
            },100)
    }

  },
  mounted: function() {
    //When app in name state
    window.addEventListener("keypress", (event) => {
      
      console.log(this.state)
      if(this.state === 'nameState'){
        if(event.code === 'Enter')
        this.startState();
      }
      else if(this.state === 'runState'){
        if(event.code === 'Space'){
          this.turnRecord()
        } 
      }
      else if(this.state === 'endState'){
      }
    });
    
      //add event when user interact with keyboard
    window.addEventListener("keyup", (event) => {
      console.log(event.code)
      //When in nameState
      if(this.state === 'nameState'){

      }
      //When in runningState
      else if(this.state === 'runState' ){     
                        
          if(event.code === 'KeyR'){
            if(!this.isPlay){      
              this.j = this.previusJ
              this.playSound();
            }
          }
          if(event.code === 'Space' ){
            this.runningFunction();
          }
      }
      //When in endState
      else if(this.state === 'endState'){

      }
    });
    
    
  },
};
</script>

<style >
@import url('https://fonts.googleapis.com/css2?family=Sarabun&display=swap');
@import 'style1.css';
.image_style{
  position: fixed;
  width: 20%;
  display: block;
  margin-left: auto;
  margin-right: auto;
  color: rebeccapurple;
  
}

* {
  padding: 0;
  margin: 0;
  
  font-family: 'Sarabun', sans-serif;
}

.center {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 1080px;
  border: 3px solid green; 
}

.image-style{
  position: fixed;
  top: 85%;
  left: 50%;
  width: 10%;
  height: auto;
  transform: translate(-50%, -50%);
}

b{
  
  font-weight: 399 !important;
}
strong.strongID{
  letter-spacing: 40px;
}
strong.strongID2{
  letter-spacing: 0px;
}
strong.strongID3{
  letter-spacing: -70px;
}
strong.strongID4{
  letter-spacing: -90px;
}
strong.strongID5{
  letter-spacing: -40px;
}
span.spanID{
letter-spacing: 40px;
}
b.bID{
letter-spacing: 40px;
}
b.bID2{
letter-spacing: 110px;
}
b.bID3{
letter-spacing: 130px;
}
/* for สระที่ข้าวหน้าเป็น ป ฟ ฝ ฬ */
b.bID4{ 
  letter-spacing: -90px;
}
b.bID5{ 
  letter-spacing: 80px;
}

sup.supID{
  letter-spacing: 40px;
}
sup.supID2{
  letter-spacing: 110px;
}


</style>
