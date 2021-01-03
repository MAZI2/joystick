<template>
  <div id="dJ" class="draggable-joystick" @mousedown="startDrag" @mouseup="stopDrag" @mousemove="move" >
    <svg class="bezier" width = "500px" height = "500px">
      <path :d="bezierPath" :fill="color" :visibility="intccl1"/>
      <path :d="bezierPath1" :fill="color" :visibility="intccl1"/>
      <path :d="circle" :fill="color" />
      <circle :cx="x1" :cy="y1" :r="width" :fill="color" :visibility="intccl"/>
      <path :d="triangle" :fill="color" />
      <line :x1="X" :y1="Y" :x2="Xs" :y2="Ys" :stroke="color" stroke-width="1" :visibility="intccl1"/>
      <line :x1="X" :y1="Y" :x2="c.c3x" :y2="c.c3y" :stroke="color" stroke-width="1" :visibility="intccl1"/>
      <line :x1="c.c3x" :y1="c.c3y" :x2="Xs" :y2="Ys" :stroke="color" stroke-width="1" :visibility="intccl1"/>

      <circle v-for="entry in entries" v-bind:key="entry.count" :cx="entry.x" :cy="entry.y" :r="entry.r" :fill="color" :opacity="entry.opacity"/>
      <text v-for="entry in entries" v-bind:key="entry.count" :x="entry.x" :y="entry.y + 5" text-anchor="middle" :font-size="fontSize">{{entry.value}}</text>
    </svg>
  
  <p>Selected:</p>
  {{left}}
  
  </div>
</template>

<script>
import dynamics from 'dynamics.js'

//number of entries
var num = 6;
//width of entries circle
var width = 200;

var angle = (2 * Math.PI)/(num); 
var count = 0;

function Dot(x1, y1) {
  this.angle = angle * count;
  count++;
  this.i = count;
  this.x = Math.cos(this.angle+(Math.PI/num)) * width + x1; 
  this.y = Math.sin(this.angle+(Math.PI/num)) * width + y1;
  this.r = 0;
  this.opacity;
  this.value;
}

export default {
  name: 'Joystick',

  created () {    
    for (var i = 0; i < num; i++) {
      this.entries[i] = new Dot(this.x1, this.y1)
    }
    dynamics.animate(this, {
      //width of center circle
      width: 50,
      //font size of entries
      fontSize: 15
    }, {
      type: dynamics.easeOut,
      duration: 350,
      friction: 1
    })
  
    //here add entries and lin urls as so
    this.entries[0].value = "Cute Cat!"
    this.entries[0].linkurl = "https://cdnuploads.aa.com.tr/uploads/Contents/2020/05/14/thumbs_b_c_88bedbc66bb57f0e884555e8250ae5f9.jpg?v=140708"
    this.entries[1].value = "Entry 2"
    this.entries[2].value = "Entry 3"
    this.entries[3].value = "Entry 4"
    this.entries[4].value = "Entry 5"
    this.entries[5].value = "Entry 6"
  
    window.addEventListener('scroll', this.scrolldistance);
  },
  unmounted () {
    window.removeEventListener('scroll', this.scrolldistance);
  },
  data: function () {
    return {
      dragging: false,
      intccl: 'visible',
      intccl1: 'hidden',
      startx: window.pageXOffset,
      starty: window.pageYOffset,

      //center circle
      x1: 250,
      y1: 250,
      width: 0,
      color: '#ff9933',

      entries: [],
      selection: 0,
      selected: '',

      fontSize: 0,
      angle: 0,
      A: {x: 0, y: 0},
      B: {x: 0, y: 0},
      C: {x: 0, y: 0},

      //side circle point
      X: 250,
      Y: 250,
      
      //pointer location
      x2: 0,
      y2: 0,
            
      c: {c3x: 0, c3y: 0, c2x: 0, c2y: 0, c5x: 0, c5y: 0},
      c1: {c1x: 0, c1y: 0, c4x: 0, c4y: 0},

      len: 0,
      lock: false,
      leng: 0,

      distanceX: 0,
      distanceY: 0,
    }
  },
  computed: {
    bezierPath: function () { return 'M' + ' ' + this.X + ' ' + this.Y + ' C ' + this.c1.c1x + ' ' + this.c1.c1y + ', ' + this.c.c2x + ' ' + this.c.c2y + ', ' + this.c.c3x + ' ' + this.c.c3y },
    bezierPath1: function () { return 'M' + ' ' + this.Xs + ' ' + this.Ys + ' C ' + this.c1.c4x + ' ' + this.c1.c4y + ', ' + this.c.c5x + ' ' + this.c.c5y + ', ' + this.c.c3x + ' ' + this.c.c3y },
    circle: function () { return 'M' + this.Xs + ' ' + this.Ys +  'A 1 1 0 0 1 ' + this.X + ' ' + this.Y },
    triangle: function () { return 'M ' + this.Xs + ' ' + this.Ys +  ' L ' + this.c.c3x + ' ' + this.c.c3y + ' L ' + this.X + ' ' + this.Y },
    
    Xs: function () { return -(this.X - this.x1) + this.x1 },
    Ys: function () { return -(this.Y - this.y1) + this.y1 },
    nx: function () { return this.x2 - this.x1 },
    ny: function () { return this.y2 - this.y1 },

    const: function () { return Math.sqrt(Math.pow(this.width, 2)/(Math.pow((this.nx), 2) + Math.pow((this.ny), 2))) },
    k: function () { return 0.011045694996616 * 50 },

    //component position cursor correction
    top: function () { return 0},
    left: function () { return 0}
  },
  methods: {
    move: function (e) {   

      this.x2 = e.clientX + this.distanceX - this.left - 50;
      this.y2 = e.clientY + this.distanceY - this.top - 50; 

      if (this.dragging && !this.lock) { 
        //initial visibility
        this.intccl1 = 'visible'
       
        this.X = this.ny * this.const + this.x1;
        this.Y = -this.nx * this.const + this.y1;
      
        //dragging length
        var long = Math.pow(Math.max(Math.sqrt(Math.pow((this.nx), 2) + Math.pow((this.ny), 2)) - this.len, 0), 0.9)
        //lenght after aplying some modifyers
        this.leng = long*(long/500) < long/2 || long == 0 ? long - long*(long/500) : 125

        //control point top
        this.c.c3x = this.nx * (this.const + Math.sqrt((this.leng*this.leng) / (this.nx*this.nx + this.ny*this.ny))) + this.x1
        this.c.c3y = this.ny * (this.const + Math.sqrt((this.leng*this.leng) / (this.nx*this.nx + this.ny*this.ny))) + this.y1               
        //control point bezier 1.1                
        this.c1.c1x = this.nx * this.const * this.k + this.X
        this.c1.c1y = this.ny * this.const * this.k + this.Y
        //control point bezier 1.2
        this.c.c2x = (this.X - this.x1) * this.k + this.c.c3x
        this.c.c2y = (this.Y - this.y1) * this.k + this.c.c3y
        //control point bezier 2.1
        this.c1.c4x = this.nx * this.const * this.k + this.Xs
        this.c1.c4y = this.ny * this.const * this.k + this.Ys
        //control point bezier 2.2
        this.c.c5x = -(this.X - this.x1) * this.k + this.c.c3x
        this.c.c5y = -(this.Y - this.y1) * this.k + this.c.c3y
        
        //stop drag if dragging more than 250
        if (Math.sqrt(Math.pow((this.nx), 2) + Math.pow((this.ny), 2)) > 250) {
          this.stopDrag()
        }

        for (var i = 0; i < num; i++) {
          var x = this.entries[i].x + this.entries[i].y - this.y1
          var y = this.entries[i].y - this.entries[i].x + this.x1
      
          var distance1 = Math.abs((y - this.entries[i].y)*this.c.c3x - (x - this.entries[i].x)*this.c.c3y + x*this.entries[i].y - y*this.entries[i].x) / Math.sqrt(Math.pow(y - this.entries[i].y, 2) + Math.pow(x - this.entries[i].x, 2))

          if (distance1 <= width + 5) {
            var distance = Math.abs((this.entries[i].y - this.y1)*this.c.c3x - (this.entries[i].x - this.x1)*this.c.c3y + this.entries[i].x*this.y1 - this.entries[i].y*this.x1) / Math.sqrt(Math.pow(this.entries[i].y - this.y1, 2) + Math.pow(this.entries[i].x - this.x1, 2))
            this.entries[i].r = this.leng / 2 / Math.max(distance/40, 1) 
          } else {
            this.entries[i].r = 0
          } 

          this.entries[i].opacity = this.entries[i].r/this.width
        }   
      }
    },
    findAngle: function(A,B,C) {
    var AB = Math.sqrt(Math.pow(B.x-A.x,2)+ Math.pow(B.y-A.y,2));    
    var BC = Math.sqrt(Math.pow(B.x-C.x,2)+ Math.pow(B.y-C.y,2)); 
    var AC = Math.sqrt(Math.pow(C.x-A.x,2)+ Math.pow(C.y-A.y,2));
    return Math.acos((BC*BC+AB*AB-AC*AC)/(2*BC*AB));
    },
    startDrag: function () {
      this.len = Math.sqrt(Math.pow((this.nx), 2) + Math.pow((this.ny), 2))

      if (this.len < this.width && !this.lock) {
        this.dragging = true
      }
    },
    stopDrag: function () {
      var leng = this.leng

      if (this.dragging && !this.lock && this.leng > 0.1) {
        this.intccl = 'hidden'
        this.dragging = false
        this.lock = true
        this.leng = 0
        
        setTimeout(() => { this.intccl = 'visible'
                           this.lock = false }, 600);

        dynamics.animate(this.c, {
          c3x: this.nx * this.const + this.x1,
          c3y: this.ny * this.const + this.y1,

          c5x: (this.Xs - this.x1) * this.k + (this.nx * this.const + this.x1),
          c5y: (this.Ys - this.y1) * this.k + (this.ny * this.const + this.y1),
          c2x: (this.X - this.x1) * this.k + (this.nx * this.const + this.x1),
          c2y: (this.Y - this.y1) * this.k + (this.ny * this.const + this.y1) 
        }, {
          type: dynamics.spring,
          duration: 700,
          friction: 280
        })

        for (var i = 0; i < num; i++) {
          dynamics.animate(this.entries[i], {
            r: 0,
            opacity: 0
          }, {
            type: dynamics.easeOut,
            duration: 350,
            friction: 1
          })
        }
      
        if (leng > width / 4) {
          this.A.x = width + this.x1; 
          this.A.y = this.y1;
          this.B.x = this.x1;
          this.B.y = this.y1;

          this.C.x = this.x2
          this.C.y = this.y2;

          if (this.y2 > this.y1) {
            this.angle = this.findAngle(this.A, this.B, this.C) * (180/Math.PI)
          } else {
            this.angle = 360 - this.findAngle(this.A, this.B, this.C) * (180/Math.PI)
          }
      
          this.selection = Math.floor(this.angle/(360/num))
          this.selected = this.entries[this.selection].value
        
          if (this.entries[this.selection].linkurl) {
          window.open(this.entries[this.selection].linkurl)
          }
        }
      } else if (this.leng < 0.1) {
        this.dragging = false
      }
    },
    scrolldistance: function() { 
    //scroll offset correction
      window.clearTimeout(isScrolling);

      var isScrolling = setTimeout(() => {
        var endy = window.pageYOffset;
        var endx = window.pageXOffset;
        
        this.distanceX = (endx - this.startx);
        this.distanceY = (endy - this.starty);
      }, 66);
    }
  }
}
</script>

<style>
div {
  padding-left: 50px;
  height: 600px;
  width: 600px;
  font-family: 'B612';
}
.bezier {
  padding-top: 50px;
  padding-left: 50px;
}
</style>
