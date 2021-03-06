# Week 16

## Adding Sound

### Task 0 - Local Server

- You will need to use a local server such as the python SimpleHTTPServer for this tutorial. Read through [this page](https://github.com/processing/p5.js/wiki/Local-server) from the lovely p5 peoples which explains more about local servers.

- Then, make sure you start your work (make a blank sketch with the p5 complete folder downloaded from p5js.org) on the local disk of the machine you are working on. This means working in the Documents folder in your username path. Once you have made a blank sketch in the Documents folder, then continue to the next point:

- OK, now open the terminal application by hitting cmd+spacebar to open spotlight, then type "terminal" and hit enter.

- In terminal, type the letters "cd" and then spacebar. "cd" means current directory, and you are telling the terminal to "look" at a particular file... Now drag your p5 folder that you just made into the terminal window. You will see that it populates with the file path of the folder.

- Hit enter, you are now telling the terminal application to "look" inside that folder.

- Still in terminal, now type ```python -m SimpleHTTPServer 8000``` and hit enter. You should see a response saying "Serving HTTP on 0.0.0.0 port 8000 ..."

- Now head open Chrome and in the address bar type localhost:8000/empty-example and it will load your page. You may not see anything yet if it's a blank sketch!

- Make sure that when you load your page, you are loading via the localhost address rather than the file path address.

### Task 1 - Soundfile

OK, first things first, let's load a sound file and play it. Make a new blank sketch with required folder structure - how many times have you done this now?! Muscle memory should hopefully be kicking in...

Let's define a variable called ```song``` and a function to preload a .mp3 file. You can find loads of sounds at www.freesound.org. Make sure you save it to the same folder as your sketch.

```javascript
let song;

function preload() {
  song = loadSound("PATH TO YOUR SOUND FILE");  
}

```

Now, let's make a setup function that initialises a bunch of stuff and plays our song.

```javascript
function setup() {
  createCanvas(480, 270);
  background(0);
  fill(255);
  textAlign(CENTER);
  text("click to play/pause", width/2, height/2);
  song.play();
  noLoop();
}
```

Try running your sketch by opening your index.html page via the localhost:YOUR PORT NUMBER address.

We don't actually need a draw function here.

So, if you're using Chrome, you may see that in the console it is coming up with a warning saying that AudioContext was not allowed to start. This is frustrating but we just need to get around it by adding (beneath setup()):

```javascript
function mousePressed() {
  getAudioContext().resume();
}
```

### Task 2 - Mouse Interaction 1

#### Soundfile

Let's add the ability to pause and play our song based on toggling with a mouse click.


```javascript

function mouseClicked() {
  if (song.isPlaying()) {
    song.stop();
  } else {
    song.play();
  }
}
```

#### Synthesis

With synthsis it's slighty different. We have to make an oscillator and make our own system of checking whether it's playing or not. So we start off with a couple of variables. The let ```playing``` is just going to go between true and false, what's that type of variable called? [HINT](https://en.wikipedia.org/wiki/Boolean_data_type).

Let's make a new sketch, call it sketchSynth.js or something. Remember what you have to do in index.html to get this file to load?

```javascript
let osc;
let playing = true;
```

Next in our setup function let's make our oscillator and start it with a frequency of concert pitch A:

```javascript
function setup() {
  createCanvas(200, 200);
  osc = new p5.Oscillator();
  osc.setType('sine');
  osc.freq(440);
  osc.start();
}
```

And finally let's toggle between fading the oscillator in and out. We have to use our ```playing``` flag to check if it is playing and act accordingly.

```javascript
function mouseClicked() {
  if (mouseX > 0 && mouseX < width && mouseY < height && mouseY > 0) { //check if we're in the canvas
    if (!playing) {// what does the ! operator mean?
      // ramp amplitude to 0.5 over 0.5 seconds
      osc.amp(0.5, 0.5);
      playing = true;
    } else {
      // ramp amplitude to 0 over 0.5 seconds
      osc.amp(0, 0.5);
      playing = false;
    }
  }
}
```

What happens if you want to change the tone of the oscillator? How would you do that? [HINT](https://p5js.org/reference/#/p5.Oscillator/setType)

### Task 3 - Mouse Interaction 2

Now let's try mapping some mouse interaction to the frequency of our sound generators.

#### Soundfile

Go back to your original sound file play back sketch. Comment out the ```noLoop()``` line.

Then change the ```song.play()``` to ```song.loop()```;

OK so we're going to map the Y position of the mouse to the playback rate of the audio file. Anything with a negative number will play the sound in reverse, which is pretty cool! Remember that Y=0 is at the top.

```javascript
function draw() {
  let speed = map(mouseY, 0.1, height, 2, -2);
  speed = constrain(speed, -2, 2);
  song.rate(speed);
}
```

#### Synthesis

We can do the same in the draw function of our synthesis patch:

```javascript
function draw() {
  let speed = map(mouseY, 0.1, height, 2000, 100);
  speed = constrain(speed, 100, 2000);
  osc.freq(speed);
}
```


#### Sound Effects

Let's try some reverberation. Reverb is an effect that mimics the way real world sound is reflected off objects and surfaces in the environment.

We need a global variable for our first audio effect, we're going to make a reverb:

```javascript
let reverb;
```

Then, in setup(), try adding this:

```javascript
//other code making oscillator.
reverb = new p5.Reverb();
osc.disconnect(); // so we'll only hear reverb...
reverb.process(osc, 10, 2);
```
Now try the same process for adding a delay. Delay is an echo that repeats the sound using a feedback loop. Have a look [here](https://p5js.org/reference/#/p5.Delay) for how to use it.

### Task 4 - Event Driven Sound

So let's pick up from the session where we created a [particle system with forces](../session_10/session_10.md). Make sure you start this off only making 5 particles, sound is very CPU intensive in the browser!

#### Synthesis

Let's declare some global variables. We need to create an array some MIDI notes; an array of strings with different waveforms that our oscillator can make; then some initial colours and a flag to check whether our mouse has been clicked.

```javascript
let scaleArray = [60, 62, 64, 67, 71, 72, 74, 77]; //array of MIDI note numbers
let waveArray = ['sine','square','sawtooth','triangle']; //sound wave sources
let bgColour = 0; //inital background colour
let particleColour = 255; //inital particle colour
let clicked = false; //flag to check whether mouse has been clicked

let delay, reverb; // our effects.
```

In our particle constructor. We're going to make an oscillator that is controlled by an envelope.

```javascript
constructor(startX, startY, startMass){
    this.mass = startMass;
    this.r = 10;
    this.pos = createVector(startX, startY);
    this.vel = createVector(random(0.5,2.5), random(0.5,2.5));
    this.acc = createVector(0, 0);
    /// new stuff
    this.osc =  new p5.Oscillator(waveArray[Math.round(random(0, waveArray.length-1))]); //make a new oscillator with a random waveform type
    this.envelope = new p5.Envelope(); // make a new envelope
    this.envelope.setADSR(0.001, 0.5, 0.05, 0.9); // set attackTime, decayTime, sustainLevel, releaseTime
    this.note = Math.round(random(0, scaleArray.length)); //select a random MIDI note from our scaleArray
    this.envelope.setRange(0.5, 0); //set volume range on the envelope
    this.osc.amp(this.envelope); //map amplitude of envelope to the oscillator
    this.freqValue = midiToFreq(scaleArray[this.note]); // convert our MIDI note to a frequency value for the oscillator
    this.osc.freq(this.freqValue); //set the oscillator frequency
    this.osc.start(); 
  }
```
Then in setup we need to update what's in there:

```javascript
function setup() {
  createCanvas(windowWidth, windowHeight); // create a canvas that fills the window
  delay = new p5.Delay(); // make delay
  reverb = new p5.Reverb(); // make reverb
  for (let i = 0; i < 10; i++) {
      particles[i] = new Particle(random(50,width-50),random(50,height-50),random(4,8));

      ////
      delay.process(particles[i].osc, .22, .75, 2900); // hook our particle oscillator up to the delay
      reverb.process(particles[i].osc, 10, 8); // hook our particle oscillator up to the reverb
    }

    for (let i = 0; i < 1; i++) {
      attractors[i] = new Attractor(random(0,width),random(0,height));
    }
}
```

And then, just because Google have been annoying, we need to just add a block that allows us to start the audio processing when there is some user interaction. For some reason, Google have made it so that, to access the Web Audio API you need to provide some positive interaction:

```javascript
function mousePressed() {
  getAudioContext().resume();
  }
```


Finally, we just need to update our checkEdges function to trigger our envelope and make a sound when the particle hits the edge of the canvas:

```javascript
checkEdges() {

    if (this.pos.x > (width-this.r)) {
      this.vel.x *= -1;
      this.pos.x = width-this.r;
      this.envelope.play(this.osc, 0, 0.1);
    } else if (this.pos.x < (0+this.r)) {
      this.vel.x *= -1;
      this.pos.x = 0+this.r;
      this.envelope.play(this.osc, 0, 0.1);
    }

    if (this.pos.y > (height-this.r)) {
      this.vel.y *= -1;
      this.pos.y = height-this.r;
      this.envelope.play(this.osc, 0, 0.1);
    } else if (this.pos.y < (0+this.r)) {
      this.vel.y *= -1;
      this.pos.y = 0+this.r;
      this.envelope.play(this.osc, 0, 0.1);
    }

  }
```



### Task 5 - State Change

One other thing, I want to just show you quickly how to change the state of your piece. This uses an if statement to toggle between two states. But you coud also use a switch statement to cycle between multiple states by using a counter. That is for you to figure out though! Let's add a toggle function to invert the colours of the piece:

```javascript
function mousePressed() {
  getAudioContext().resume();
   if (!clicked) {
    bgColour = 255;
    particleColour = 0;
    clicked = true;
  } else {
    bgColour = 0;
    particleColour = 255;
    clicked = false;
  }
}

```
Now we just need to set our particle colour in the display function of the particle:

```javascript
display() {
    stroke(particleColour);
    strokeWeight(2);
    noFill();
    ellipse(this.pos.x, this.pos.y,this.mass*2,this.mass*2);
  }
```

[Here](http://davemeckin.panel.uwe.ac.uk/Week_18_Demo/sketch_folder/) is my version of the final piece. This could be submitted to fit the brief...

Stretch goal: instead of only having 2 states and toggling between them, how can you make more? HINT: use a [switch statement](https://www.w3schools.com/js/js_switch.asp)..

### Task 6 - Parameter Mapping

What kind of parameters from the objects in your sketch can you map to sound?

What about:

- Speed to volume? [HINT](https://p5js.org/reference/#/p5.SoundFile/setVolume)
- X Position to pan? [HINT](https://p5js.org/examples/sound-pan-sound.html)
- Y Position to filter frequency? [HINT](https://p5js.org/reference/#/p5.Filter)

etc etc etc

