
//---------------------------------------------------------------
//class define
class Point{
  float x, y;
  Point(float nx, float ny){
    x = nx;
    y = ny;
  }
  void draw(){
    drawPoint(this);
  }
}

class Vec extends Point {
  Vec(float nx, float ny){
    super(nx, ny);
  }
}

class Line{
  Point p1, p2;
  Vec pv, vv;
  Line(Point np1, Point np2){
    p1 = np1;
    p2 = np2;
    float l = sqrt((p2.x - p1.x) * (p2.x - p1.x) + (p2.y - p1.y) * (p2.y - p1.y));
    pv = new Vec((p2.x - p1.x)/l, (p2.y - p1.y)/l);
    vv = new Vec(pv.y, -pv.x);
  }
  Line(float x1, float y1, float x2, float y2){
    this(new Point(x1, y1), new Point(x2, y2));
  }
  void draw(){
    stroke(0, 0, 0);
    strokeWeight(1);
    drawLine(this);
  }
  Point delta(){
    return subPoint(p2, p1);
  }
  float squareDistance(){
    return (p2.x - p1.x) * (p2.x - p1.x) + (p2.y - p1.y) * (p2.y - p1.y);
  }
  Line clone(){
    return new Line(p1.x, p1.y, p2.x, p2.y);
  }
}

class Player{
  Point pos, vel;
  Line move;
  
  Player(){
    pos = new Point(250, 250);
    vel = new Point(0, 0);
    move = new Line(0, 0, 0, 0);
  }
  
  void update(ArrayList<Line> ls){
    vel.x = 0;
    vel.y = 0;
    if(isPressLeft){
      vel.x -= 3;
    }
    if(isPressRight){
      vel.x += 3;
    }
    if(isPressUp){
      vel.y -= 3;
    }
    if(isPressDown){
      vel.y += 3;
    }
    move.p1 = pos;
    move.p2 = addPoint(pos, vel);
    
    Line minLine = move.clone();
    float minLength = 10000000;
    for(int i = 0; i < ls.size(); i++){
      Line l = ls.get(i);
      if(isIntersected(move, l) && innerProduct(move.delta(), l.vv) > 0){
        Line collision = bendLine(move, l);
        if(minLength > collision.squareDistance()){
          minLength = collision.squareDistance();
          minLine = collision;
        }
      }
    }
    if(minLength < 10000000){
      move = minLine;
    }
    minLine = move.clone();
    minLength = 10000000;
    for(int i = 0; i < ls.size(); i++){
      Line l = ls.get(i);
      if(isIntersected(move, l) && innerProduct(move.delta(), l.vv) > 0){
        Line collision = foldLine(move, l);
        if(minLength > collision.squareDistance()){
          minLength = collision.squareDistance();
          minLine = collision;
        }
      }
    }
    if(minLength < 10000000){
      move = minLine;
    }
    pos = addPoint(pos, move.delta());
  }
  void draw(){
    stroke(0, 0, 0);
    strokeWeight(5);
    drawPoint(pos);
  }
}

//----------------------------------------------------------
//variable define
Player player;
ArrayList<Line> lines;
boolean isPressRight, isPressLeft, isPressUp, isPressDown;
//variable define end

//----------------------------------------------------------
//function define

//key event
void keyPressed() {
  if (key == CODED) {
    if (keyCode == RIGHT) {
      isPressRight = true;
    }
    if(keyCode == LEFT){
      isPressLeft = true;
    }
    if(keyCode == UP){
      isPressUp = true;
    }
    if(keyCode == DOWN){
      isPressDown = true;
    }
  }
}

void keyReleased(){
  if (key == CODED) {
    if (keyCode == RIGHT) {
      isPressRight = false;
    }
    if(keyCode == LEFT){
      isPressLeft = false;
    }
    if(keyCode == UP){
      isPressUp = false;
    }
    if(keyCode == DOWN){
      isPressDown = false;
    }
  }
}

//operator function
Point addPoint(Point lp, Point rp){
  return new Point(lp.x + rp.x, lp.y + rp.y);
}

Point subPoint(Point lp, Point rp){
  return new Point(lp.x - rp.x, lp.y - rp.y);
}

Point product(Point lp, float rf){
  return new Point(lp.x * rf, lp.y * rf);
}

float innerProduct(Point lp, Vec rp){
  return lp.x * rp.x + lp.y * rp.y;
}

float innerProduct(Line ll, Vec ru){
  return innerProduct(ll.delta(), ru);
}

//collision function
boolean isIntersected(Line l1, Line l2) {
    if (l1.p1.x == l1.p2.x && l2.p1.x == l2.p2.x && l1.p1.x == l2.p1.x) {
        return false;
    }
    if (l1.p1.y == l1.p2.y && l2.p1.y == l2.p2.y && l1.p1.y == l2.p1.y) {
        return false;
    }
    float a = (l2.p1.x - l2.p2.x) * (l1.p1.y - l2.p1.y) + (l2.p1.y - l2.p2.y) * (l2.p1.x - l1.p1.x);
    float b = (l2.p1.x - l2.p2.x) * (l1.p2.y - l2.p1.y) + (l2.p1.y - l2.p2.y) * (l2.p1.x - l1.p2.x);
    float c = (l1.p1.x - l1.p2.x) * (l2.p1.y - l1.p1.y) + (l1.p1.y - l1.p2.y) * (l1.p1.x - l2.p1.x);
    float d = (l1.p1.x - l1.p2.x) * (l2.p2.y - l1.p1.y) + (l1.p1.y - l1.p2.y) * (l1.p1.x - l2.p2.x);
    return c * d <= 0 && a * b <= 0;
}

Line foldLine(Line l1, Line l2) {
  Line ans = l1.clone();
  float s1 = ((l2.p2.x - l2.p1.x) * (l1.p1.y - l2.p1.y) - (l2.p2.y - l2.p1.y) * (l1.p1.x - l2.p1.x));
  float s2 = ((l2.p2.x - l2.p1.x) * (l2.p1.y - l1.p2.y) - (l2.p2.y - l2.p1.y) * (l2.p1.x - l1.p2.x));
  ans.p2.x = l1.p1.x + (l1.p2.x - l1.p1.x) * s1 / (s1 + s2);
  ans.p2.y = l1.p1.y + (l1.p2.y - l1.p1.y) * s1 / (s1 + s2);
  ans.p2 = subPoint(ans.p2, l2.vv);
  return ans;
}

Line bendLine(Line l1, Line l2){
  Line remain = l1.clone();
  Line ans = foldLine(l1, l2);
  ans.p2 = addPoint(ans.p2, product(l2.pv, innerProduct(remain, l2.pv)));
  return ans;
}

//draw function
void drawPoint(float x, float y){
  point(x, y);
}

void drawPoint(Point p){
  drawPoint(p.x, p.y);
}

void drawLine(float x1, float y1, float x2, float y2){
  line(x1, y1, x2, y2);
}

void drawLine(Point p1, Point p2){
  drawLine(p1.x, p1.y, p2.x, p2.y);
}

void drawLine(Line l){
  drawLine(l.p1, l.p2);
}

//main function
void setup() {
  size(500, 500);
  colorMode(RGB,256);
  background(255, 255, 255);
  player = new Player();
  lines = new ArrayList<Line>();
  lines.add(new Line(499, 499, 0, 499));
  lines.add(new Line(0, 499, 0, 0));
  lines.add(new Line(0, 0, 499, 0));
  lines.add(new Line(499, 0, 499, 499));
  lines.add(new Line(50, 50, 449, 449));
  lines.add(new Line(449, 449, 50, 50));
}

void draw(){
  player.update(lines);
  
  noStroke();
  rect(0, 0, 500, 500);
  
  for(int i = 0; i < lines.size(); i++){
    Line l = lines.get(i);
    l.draw();
  }
  player.draw();
}