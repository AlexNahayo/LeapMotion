# LeapMotion
This project uses the Leap Motion sensor and looks at how it can be used as a musical tool.
I used processing through Java which has allowed me to implement various audio techniques and combining that with a visual representation on a screen. 
The Cube showcases the leap motion potential in individual music performance.

import ddf.minim.*;
import ddf.minim.signals.*;
import ddf.minim.ugens.*;
import ddf.minim.effects.*;
import de.voidplus.leapmotion.*;

LeapMotion leap;
Minim minim;
AudioInput in;//for recording
AudioRecorder recorder; //for recording
AudioOutput output;
FilePlayer fileplayer; //in the background
Gain gain;
LowPassFS lpf;
HighPassSP hpf;

Boolean canPlay = true;
Boolean Ambientregion;
Boolean recorded = false; //for recording
String fileName = ("02. Lift Me Up.mp3");

void setup() {
    // initialize the drawing window 
    size(512, 200, P3D); 
    //initialize the minim,output,leap,gain,hpf/lpf,fileplayer and myDelay objects.
    minim = new Minim(this);
    in = minim.getLineIn();
    leap = new LeapMotion(this).allowGestures();  
    fileplayer = new FilePlayer( minim.loadFileStream(fileName) );
    recorder = minim.createRecorder(in, "myrecording.wav");
    gain = new Gain(0.f);
    output = minim.getLineOut();
    hpf = new HighPassSP(1000, output.sampleRate());
    // Patching objects into the fileplayer
    fileplayer.patch(hpf).patch(gain).patch(output);   
    leap = new LeapMotion(this);
    Ambientregion = false;
  
}



void draw() {
   
 lights();
 // TRIGGER INITIALIZED
 if (Ambientregion == true) {
     Ambientregion = false;
  }   
   //This function below will help locate the postion of my hands.
      for (Hand hand : leap.getHands ()) {
          PVector hand_position = hand.getPosition();//This will help recognise hand position via the leap.
          PVector hand_stabilized     = hand.getStabilizedPosition();
        //needed for the cube to rotate according to the wrist's motion.
          float   hand_pitch       = radians(hand.getPitch());//angle around the x-axis
          float   hand_yaw         = radians(hand.getYaw());//angle around the y-axis
          float   hand_roll        = radians(hand.getRoll()); //angle around the z-axis
      //This helped me get colours in the background.    
      for (Finger finger : hand.getFingers()) {
          int     touch_zone        = finger.getTouchZone(); //Zone which is in the switch state below diiferent background colours
          float   touch_distance    = finger.getTouchDistance();// Differences between the difference zones can be seen below.
            // Each int value here represents a colour range.
          int x = (int)(abs(touch_distance*1000))%255;  
          int y = (int)(abs(touch_distance*1000))%255;
          int z = (int)(abs(touch_distance*1000))%255; 
      switch(touch_zone) {
      case -1:
        background(0,50,z);
      break;
      case 0: // Hovering
        background(x,50,0);
      break;
      case 1:
        background(50,y,0);
      break;
       }
      }

 
   //****MULTI EFFECTS CODE****.
   
   //Panning/Cubecontroller(colour, rotation and position)/Gain and Lpf/Delay and chorus effect  with the right hand exclusively.
  if ( output.hasControl(Controller.PAN) && hand.isRight()){
     
      String handType = hand.isRight() ? "Right hand" : "Left hand";
      println(handType + " position: " + hand_position, 50, hand.isLeft() ? 50 : 100);
      // map the hand position to the range of the pan with hand position.
      float val = map(hand_position.x, 0, width, -1, 1); //moving rom left to right with ones hand (one hand at a time)
      output.setPan(val); 
    
      // The interchanging colour of the cube mapped to individual coordinates.
      float Red = hand_position.x; //assigning hand position with a colour of RGB.
      float Blue = hand_position.y;
      float Green = hand_position.z;
      
      smooth();
      int x = (int)(abs(Red))%255 ; //THIS WILL MAKE SURE THAT THESE COLOURS AREN'T NEGATIVE OR OUT OF THE REGION FOR EACH COLOUR.
      int y = (int)(abs(Blue*10))%255;//increment x 10 to add a bit more colpur
      println("Blue"+ y);//increment x 100 to add a bit more colour variation
      int z = (int)(abs(Green*100))%255;

    
      ///Code for the cube   
      pushMatrix();   //saves the cuurent coordinate system wiht matrices.
      translate(hand_stabilized.x, hand_stabilized.y, hand_stabilized.z);// moves the cube in the x , y and z coordinates
      rotateX(hand_roll+PI);
      rotateY(hand_pitch);
      rotateZ(-hand_yaw);
      
      fill(x,y,z);//TRYING TO MAP EACH COLOUR TO THE DIFFERENT CORDINATES.
      box(200,200,200);
      popMatrix();
      
       //FOR THE Right hand for amplitude. By moving from up and down.
      float dB = map(hand_position.y, height, 0, -6, 6);
      gain.setValue(dB);
      
      //LOWPASS FILTER, which is initiated with back and forth movement of the right hand. 
      float cutoff = map(hand_position.z, 0, width, 1000, 14000);
      hpf.setFreq(cutoff);

    }
 
     
  } 
}


        //KEY TAP GESTURE FOR  STARTING THE MUSIC. 
    void leapOnKeyTapGesture(KeyTapGesture g){
    
      PVector position         = g.getPosition(); 

      for (Hand hand : leap.getHands ()) {
          String hand_Type = hand.isLeft()  ? "Left hand" : "Right hand"; //left Hand can be used to trigger the stop the music with listed gesture.
     
      if (position.x>0&& position.x<600 && position.y>0 && position.y<600 && position.z>0 && position.z<600 && hand.isLeft() && Ambientregion == false){
          //just raise my arm abit more to start.
          Ambientregion = true;
          fileplayer.play();   
        // stating the recorder to true allows the Audiorecorder to be activated.
      if (!recorded){
          fileplayer.play();
          recorder.beginRecord();
          
          }
        }
    
      else {
          println("notintheregion "+Ambientregion);// beside "your able to check true or false statements
          Ambientregion = false;
        
      }

      if (canPlay && Ambientregion && hand.isLeft() ) {
          fileplayer.play();
          fileplayer.rewind();
          fileplayer.loop();
          canPlay = false;
      }
    
      if (!Ambientregion); {
          canPlay = true;
    }
  }
}
    //STOP MUSIC WITH SWIPE GESTURE.(LEFT/RIGHT MOTION) 
    void leapOnSwipeGesture(SwipeGesture g, int state){ 
      
      PVector position         = g.getPosition();

      for (Hand hand : leap.getHands ()) {
          String hand_Type = hand.isLeft()  ? "Left hand" : "Right hand"; //left Hand can be used to start the music with listed gesture.
          
      if (position.x>0 && position.x<600 && position.y>0 && position.y<600 && position.z>0 && position.z<600 && hand.isLeft() && Ambientregion == false){
          Ambientregion = true;
      }
      else {
          Ambientregion = false;
        // stating the recorder to true allows the Audiorecorder to be activated.
      if (recorder.isRecording()){
          recorded = true;
          recorder.endRecord();
          recorder.save(); 
        
        }
      }
 
      if (canPlay && Ambientregion && hand.isLeft()) {
          fileplayer.pause();
          canPlay = false;
        }

      if (!Ambientregion);{
          canPlay = true;

    }
  }
     
} 

        //STOP MUSIC WITH MOUSE PRESSED.(IMCASE OF EMERGENCY DURING)
      void mousePressed(){
        fileplayer.pause();
      }

          //BACK AND FORWARD SKIP WITH CLOCKWISE/ANTICLOCKWISE MOTION.
      void leapOnCircleGesture(CircleGesture g, int state){
        
        int     id               = g.getId();
        int     direction        = g.getDirection();

        for (Hand hand : leap.getHands ()) {
            String hand_Type = hand.isLeft()  ? "Left hand" : "Right hand"; // Left Hand can be used to trigger the skip the back and forward parameters exclusively.
          
          
            switch(direction){
              case 0: // Anticlockwise/Left gesture
                if (hand.isLeft()){
                  fileplayer.skip(-5000);
            }
              break;
              case 1: // Clockwise/Right gesture
                if (hand.isLeft()){
                  fileplayer.skip(5000);
            }
              break;
        }
  }
}
