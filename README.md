
# Autonomous Agents and Steering

## 1: Autonomous Agents and Steering

- Autonomous Agent
    - has **limited** ability to perceive its environment
        - e.g. can perceive anything with 25 pixel of itself , or only to see things that are in its field of view.
    - process the environment, **calculate an action**, it's going to result a **force**.
    - no global plan , or leader
- Vehicles
    - action/selection
        - desire velocity
    - steering
        - calculate a steering force based on desire velocity
    - locmotion
        - physics simulation

## 2: Steering Behaviors: Seek

- `steering = desired - velocity`
- Pseudo Seek Code
    ```java
    PVector desired = PVector.sub( target, location );
    // normalized first, and multiply `maxSpeed`
    desired.SetMag( maxSpeed );

    PVector steering = PVector.sub( desired - velocity );
    steering.limit( maxForce ); // not SetMag !!

    applyForce( steering );
    ```
- Vehicle
    ```java
    class Vehicle {
        float maxspeed;
        // how good is it at turning
        float maxforce;
    }
    ```
- simple seek, but this implementation has weird behavior, it's always sort of flying past the target and then it has to turn aroud and turn back.
    - we will solve this problem by introducing the `arriving` behavior what it means for that vehicle to slow down and stop when it reaches the target. We're going to change the desired velocity so that the closer to the target the smaller it is. That is, when I'm close to the target, I desire to be moving very very slowly at that point. 



<details>
<summary>
Seek
</summary>

```java
// Seeking "vehicle" follows the mouse position

// Implements Craig Reynold's autonomous steering behaviors
// One vehicle "seeks"
// See: http://www.red3d.com/cwr/

Vehicle v;

void setup() {
  size(640, 360);
  v = new Vehicle(width/2, height/2);
}

void draw() {
  background(255);

  PVector mouse = new PVector(mouseX, mouseY);

  // Draw an ellipse at the mouse position
  fill(200);
  stroke(0);
  strokeWeight(2);
  ellipse(mouse.x, mouse.y, 48, 48);

  // Call the appropriate steering behaviors for our agents
  v.seek(mouse);
  v.update();
  v.display();
}

```

</details>


<details>
<summary>
Vehicle Class
</summary>

```java
// Seek_Arrive

// The "Vehicle" class

class Vehicle {
  
  PVector position;
  PVector velocity;
  PVector acceleration;
  float r;
  float maxforce;    // Maximum steering force
  float maxspeed;    // Maximum speed

  Vehicle(float x, float y) {
    acceleration = new PVector(0,0);
    velocity = new PVector(0,-2);
    position = new PVector(x,y);
    r = 6;
    maxspeed = 4;
    maxforce = 0.1;
  }

  // Method to update position
  void update() {
    // Update velocity
    velocity.add(acceleration);
    // Limit speed
    velocity.limit(maxspeed);
    position.add(velocity);
    // Reset accelerationelertion to 0 each cycle
    acceleration.mult(0);
  }

  void applyForce(PVector force) {
    // We could add mass here if we want A = F / M
    acceleration.add(force);
  }

  // A method that calculates a steering force towards a target
  // STEER = DESIRED MINUS VELOCITY
  void seek(PVector target) {
    PVector desired = PVector.sub(target,position);  // A vector pointing from the position to the target
    
    // Scale to maximum speed
    desired.setMag(maxspeed);

    // Steering = Desired minus velocity
    PVector steer = PVector.sub(desired,velocity);
    steer.limit(maxforce);  // Limit to maximum steering force
    
    applyForce(steer);
  }

  void display() {
    // Draw a triangle rotated in the direction of velocity
    float theta = velocity.heading2D() + PI/2;
    fill(127);
    stroke(0);
    strokeWeight(1);
    pushMatrix();
    translate(position.x,position.y);
    rotate(theta);
    beginShape();
    vertex(0, -r*2);
    vertex(-r, r*2);
    vertex(r, r*2);
    endShape(CLOSE);
    popMatrix();
  }
}
```

</details>

## 3: Steering Behaviors: Arrive

- If I'm close to the target maybe I want my desired velocity to be not so fast. And if I'm on the target, my desired velocity should be zero. So how to do it ?
- one way that we can do it is we can think of the target as having an invisible circle around it, with some radius , let's say 100 , and when the vehicle is anywhere outside of the target's circle, its desired velocity magnitude is maximum speed. 
    - now when its right on the edge it still desires to go in max speed.
    - when it is right on the target, it desires its magnitude is 0.
    - when it is halfway in between circle edge and the target, it desires its magnitude is `maxspeed/2`.
    ```java
    float distance = PVector.dist( myLocaltion, target );
    float speed = map( distance, 0, 100 , 0, maxspeed );
    ```
    - this is a really fantastic solution and works very well. But the real question isn't just to say that this model is that what you should use. It's this way of thinking. 


<details>
<summary>
Seek()
</summary>

```java
  // deprecate seek(), use arrive() instead

  // A method that calculates a steering force towards a target
  // STEER = DESIRED MINUS VELOCITY
  void arrive(PVector target) {
    PVector desired = PVector.sub(target,position);  // A vector pointing from the position to the target
    float d = desired.mag();
    // Scale with arbitrary damping within 100 pixels
    if (d < 100) {
      float m = map(d,0,100,0,maxspeed);
      desired.setMag(m);
    } else {
      desired.setMag(maxspeed);
    }

    // Steering = Desired minus Velocity
    PVector steer = PVector.sub(desired,velocity);
    steer.limit(maxforce);  // Limit to maximum steering force
    applyForce(steer);
  }
```

</details>

---

- What if the target happens to be a vehicle and it's moving as well, and you're seeking it.
    - this behavior is called `pursuit`.
    - I know the target's velocity, I can predict that in a few moments where it's going to be.
    - and my desired velocity shouldn't be pointed towards its current location , it should be pointed towards where I believe its future location is going to be.


## 4: Steering Behaviors: Flow FIeld Following 


