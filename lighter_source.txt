//Created by: Dylan Randall
//Last update: 14/08/2017

color solidCol = color(0,0,0);
color emptyCol = color(0,99,99);
color lightCol = color(0,255,255);
color shadowCol = color(0,100,100);
color mouseCol = color(255,0,0);

int cols = 10;
int rows = 10;
int cellWidth = 0;
int cellHeight = 0;

int s0 = 0;
int s1 = s0;
int frames = 0;
int fps = 0;

boolean[][] map = new boolean[cols][rows];

float percentSolid = 0.05;
float step = 0.01;
float angleStep = 0.5f;

void setup()
{
  size(800,800);
  initMap();
  noStroke();
  noCursor();
  textSize(26);
  s0 = second();
  s1 = s0;
}

void draw()
{ 
  s1 = second();
  frames++;
  drawMap();
  drawRays();
  blendLight();
  fill(mouseCol);
  ellipse(mouseX,mouseY,20,20);
  
  fill(255,0,0);
  if((s1-s0) >= 1)
  {
    fps = frames;
    s0=second();
    s1=s0;
    frames=0;
  }
  
  text("FPS: "+str(fps), 25, 25);
  text("ROOM DENSITY: "+Float.toString(percentSolid), 25, 55);
}

void mouseWheel(MouseEvent event) 
{
  float e = event.getCount();
  boolean update = false;
  if(e < 0) //mouse wheel up
  {
    update = true;
    for(int i = 0; i < (int) (e*-1); i++)
    {
      percentSolid = incrementVal(percentSolid, step, 1);
    }
  }
  else if(e > 0)
  {
    update = true;
    for(int i = 0; i < (int)e; i++)
    {
      percentSolid = decrementVal(percentSolid, step, 0);
    }
  }
  
  if(update == true)
  {
    update = false;
    initMap();
  }
}

void mousePressed()
{
  if(mouseButton == LEFT)
  {
    initMap();
  }
}

private float mapValue(float value, float lMin, float lMax, float rMin, float rMax) //maps a value of one range into another range
{
  float lRange = lMax-lMin;
  float rRange = rMax-rMin;
  
  float sclVal = (value-lMin)/lRange;
  sclVal = rMin+(sclVal*rRange);
  return sclVal;
}

private int[] getRayPoint(int[] origin, float angle, int radius) //calculates point of ray
{
  int[] hit = new int[2];
  hit[0] = round(origin[0]+radius*cos(radians(angle)));
  hit[1] = round(origin[1]+radius*sin(radians(angle)));
  return hit;
}

private float incrementVal(float val, float step, float max)
{
  if((val+step) <= max)
  {
    return val+step;
  }
  else
  {
    return max;
  }
}

private float decrementVal(float val, float step, float min)
{
  if((val-step) >= min)
  {
    return val-step;
  }
  else
  {
    return min;
  }
}

private void initMap()
{
  cellWidth = width/cols;
  cellHeight = height/rows;
  float xOff = floor(random(0,1000));
  float yOff = floor(random(0,1000));
  noiseDetail((int)random(0,4),random(0,percentSolid));
  noiseSeed((int)random(0,10000));
  for(int x = 0; x < cols; x++) //sets up map
  {
    for(int y = 0; y < rows; y++)
    {
      boolean isSolid = false;
      float r = noise((x+xOff)*cellWidth,(y+yOff)*cellHeight);
      
      if(r <= percentSolid) //cell is solid
      {
        isSolid = true;
      }
      
      map[x][y] = isSolid;
    }
  }
}

private void drawMap()
{
  for(int x = 0; x < width; x += cellWidth)
  {
    for(int y = 0; y < height; y += cellHeight)
    {
      int adjX = (int)(mapValue(x,0,width,0,cols));
      int adjY = (int)(mapValue(y,0,height,0,rows));
      
      if(map[adjX][adjY] == true)
      {
        fill(solidCol);
      }
      else
      {
        fill(emptyCol);
      }
      
      rect(x,y,cellWidth,cellHeight);
    }
  }
}

int pad = 2;
private void drawRays()
{
  int x = mouseX; //point of mouse in screen
  int y = mouseY;
  
  if(x < pad)
  {
    x = pad;
  }
  else if(x > width-pad)
  {
    x = width-pad;
  }
  
  if(y < pad)
  {
    y = pad;
  }
  else if(y > height-pad)
  {
    y = height-pad;
  }
  
  loadPixels();
  for(float angle = 0; angle < 360; angle += angleStep)
  {
    boolean run=true,drawShadow=false;
    int l = 0;
    while(run==true)
    {
      int[] xy = getRayPoint(new int[]{x,y}, angle, l); //casts ray
      
      if((xy[0] > 0 && xy[0] < width-1)&&(xy[1] > 0 && xy[1] < height-1)) //validates the point
      {
        color pixCol = pixels[xy[1]*width+xy[0]];
        
        if(drawShadow==false)
        {
          if(pixCol == solidCol)
          {
            drawShadow = true;
          }
          else
          {
            pixels[xy[1]*width+xy[0]] = lightCol;
          }
        }
        else
        {
          if(pixCol != solidCol)
          {
            pixels[xy[1]*width+xy[0]] = shadowCol;
          }
        }
      }
      else
      {
        run = false;
      }
      l++;
    }
  }
  
  updatePixels();
}

private void blendLight()
{
  blendHor();
  blendVer();
}

private void blendHor()
{
  loadPixels();
  for(int y = 0; y < height; y++)
  {
    color col = color(0,0,0);
    boolean pass = true;
    for(int x = 0; x < width; x++)
    {
      color pixCol = pixels[y*width+x];
      if(pixCol == lightCol)
      {
        pass = false;
        col = lightCol;
      }
      else if(pixCol == shadowCol)
      {
        pass = false;
        col = shadowCol;
      }
      else if(pixCol == solidCol)
      {
        pass = true;
      }
      
      if(pass != true)
      {
        pixels[y*width+x] = col;
      }
    }
  }
  updatePixels();
}

private void blendVer()
{
  for(int x = 0; x < width; x++)
  {
    color col = color(0,0,0);
    boolean pass = true;
    for(int y = 0; y < height; y++)
    {
      color pixCol = pixels[y*width+x];
      if(pixCol == lightCol)
      {
        pass = false;
        col = lightCol;
      }
      else if(pixCol == shadowCol)
      {
        pass = false;
        col = shadowCol;
      }
      else if(pixCol == solidCol)
      {
        pass = true;
      }
      
      if(pass != true)
      {
        pixels[y*width+x] = col;
      }
    }
  }
  updatePixels();
}