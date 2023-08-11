# <center>Pixel - Advanced Using</center>

## 1. Sprites out of a Single Picture 
In the previous chapter *Fairy? Sprite!* We have mentioned using defferent potion of a single picture to create different sprites, and here is it!  
The core of such operation is use a list of rectangles to identify the range of different portions.   
```Go
// Assume we have defined the following variables:
// spritesheet pixel.Picture // 96x96 = (32x3)x(32x3)
// treeFrames  []pixel.Rect
for x := spritesheet.Bounds().Min.X; x < spritesheet.Bounds().Max.X; x += 32 {
    for y := spritesheet.Bounds().Min.Y; y < spritesheet.Bounds().Max.Y; y += 32 {
        treesFrames = append(treesFrames, pixel.R(x, y, x+32, y+32))
    }
}
```  
Since the scale of the picture is known, we just simply define the rectangles and store tem in a slice.  
And now we are able to use the slice to create sprites.  
```Go
sprite := pixel.NewSprite(spritesheet, treeFrames[2])
```
Or apply random number.  
```Go
sprite := pixel.NewSprite(spritesheet, treeFrames[rand.Intn(len(treeFrames))])
```
And other funny ways you can think.  

## 2. Camera's looking at the Game  
Imagine some 2D games, like *Don't Starve*. the background is moving when we move the character. If we assume all the scenery is collected by a camera in the air, then it is moving when the character moves.  
Not difficult to think of that, in the game, the locations of the trees are constants, but they are actually moving on the screen. To illustrate such difference, we use the terms "Game Space" and "Screen Space".  
Let's have an example! When we press "Down", the location of camera(or rather, Wilson) on Y-axis reduced in game space, while that of the trees increased in the screen space. If we use a matrix to implement the movement of camera, then its inverse matrix is expected for the trees. That is, when apply `pixel.IM.Moved(camPos)` to the screen space, we will get the game space. So far, the hardship has been overcome. Please check the following code rapidly.
```Go
// 1. Record the position of the camera and its speed
var(
    camPos = pixel.ZV
    camSpeed = 500.0
)

// 2. Record the time for the calculation of camPos
last := time.Now()      // Package "time" needed

for !win.Closed() {
    dt := time.Since(last).Seconds()    // The length of a period
    last := time.Now()

    // 3. Create the matrix "cam" out of camPos
    cam := pixel.IM.Moved(win.Bounds().Center().Sub(camPos))
    win.SetMatrix(cam)   // All the objects drawn on win will undergo cam

    // 4. Update camPos
    if win.Pressed(pixelgl.KeyLeft) {
        camPos.X -= camSpeed * dt
    }
    if win.Pressed(pixelgl.KeyRight) {
        camPos.X += camSpeed * dt
    }
    if win.Pressed(pixelgl.KeyDown) {
        camPos.Y -= camSpeed * dt
    }
    if win.Pressed(pixelgl.KeyUp) {
        camPos.Y += camSpeed * dt
    }

    win.Clear(colornames.Forestgreen)
    win.Update()
}
```  
Notice, that the  we used the vector `win.Bound().Center().Sub(camPos)` instead of `camPos`. Why? It have something to do with the origins of the screen space and the game space. This matrix ought to demonstrate a conversion from game space to screen space, so we must apply "-camPos", which in code is written like `someVector.Sub(camPos)`. But what should "someVector" be? In the beginning, the origin of game space should be at the center of the screen. So "someVector" should be `win.Bound().Center()`.  

## 3. A Batch of Sprites
When we draw many many sprites at a time, we can feel a clear sense of lag(or low fps). That's because the fps depends on the times `Draw` is called. However, we have to call `Draw` for every sprite till now. Thus, here comes a kind of container to deal with a large amount of call of that function - `pixel.Batch`.  
To create a batch, we can use the following code.  
```Go
batch := pixel.NewBatch(&pixel.TrianglesData{}, picture)
```  
The second argument should be the same picture as the sprites. For simple uses, we needn't understand what `&pixel.TrianglesData{}` actually mean, but just use it in our code. (Honestly I haven't yet figured it out. XD)  
To add a sprite into the batch, we should just "draw the sprite on the batch", and finally draw tne batch on the screen.  
```Go
sprite.Draw(batch, pixel.IM)
// ...
batch.Draw(win)
```
Another method frequently used is `batch.Clear()`, which clean up it to make an empty but initialized batch.  

## 4. Align the Text!
By default, the text are aligned to the left. But how could we change that? We can have it done by changing `Dot` and `Orig`.  
First, since we have to deal with the content line by line, The content should be devided apart, like this:  
```Go
lines := []string{
    "This is a very long line",
    "Short line",
    "!@#$^&*()_+",
}
```  
Then move every line's `Dot` the length of itself to the left, and write it into `txt`.  
```Go
for _, line := range lines {
    txt.Dot.X -= txt.BoundsOf(line).W()
    fmt.Fprintln(txt, line)
}
```
If we print the text, we'll find it at the left of `Orig` and aligning to it. So remember not to make `Orig` at left.  
Aligning to the middle is similar to what we have down just now. Merely by change this line couuld do that.  
```Go
txt.Dot.X -= txt.BoundsOf(line).W()/2
```

## 5. IMDraw 
~~More knowledges needed, I haven't figured it out.~~  
See more at [Authoritized Tutorial](https://github.com/faiface/pixel/wiki/Drawing-shapes-with-IMDraw)

---

####For more information, please check the following websites:  

**Pixel**: <https://github.com/faiface/pixel>
**Pixel Package**: <https://pkg.go.dev/github.com/faiface/pixel>
**Pixel Wiki**: <https://github.com/faiface/pixel/wiki>