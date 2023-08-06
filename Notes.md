# <center>Pixel - Quick Tutorial</center>
---
## 1. What to import?
* Basic: `github.com/faiface/pixel` and `github.com/faiface/pixel/pixelgl`  
* Color: `golang.org/image/x/color`  
* Text : `github.com/faiface/pixel/text`  

## 2. It looks alian.  
```Go
package main

import(
    "github.com/faiface/pixel"
    "github.com/faiface/pixelgl"
)

func run() {
    // The "main thread" here
}

func main() {
    pixelgl.Run(run)
}
```

## 3. Open the window!  
First we need a structure to configue the window, like the following:  
```Go
cfg := pixelgl.WindowConfig{
    // The title
    Title: "Pixel Rocks!",
    // The size of the window, decided by a rectangle
    Bounds: pixel.R(0, 0, 1024, 768),
    // Vertical Sync, would limit the fps
    VSync: true,
}
```  
Then we are able to initalize a window:  
```Go
win, err := pixelgl.NewWindow(cfg)
if err != nil {
    // Should be replaced by more appropriate measures
    panic(err)
}
```  
And finally show the window:  
```Go
for !win.Closed() {
    win.Update()
}
```  
To clear the window with particular color(white for example), we could use this:  
```Go
win.Clear(colornames.White)
```  
Write it in the loop so that it will br executed every time.

So far we have learnt how to initialize a window and clear it.  

## 4. I want a picture!  
In Pixel, anyway, what we finally need is an object with type `pixel.Picture`, which can be converted from given type `image.Image`, while the latter are loaded by decoding an image file.  
So first, we'd open a image and read it. Second, decode it to get an `image.Image` object. Finally, convert it to our expected type, `pixel.Picture`. We integrate all preceedures above into a single function.  
```Go
func loadPNG(path string) (pixel.Picture, error) {
    // Open the file and read it
    file, err := os.Open(path)      // Package "os" needed
    if err != nil {
        return nil, err
    }
    defer file.Close()              // Remerber to close the file

    // Decode the image
    img, _, err := image.Decode(file)  // Package "image" and _"image/png" needed
    if err != nil {
        return nil, err
    }

    // Covert the image and return
    return pixel.PictureDataFromImage(img), nil
}
```  
Notice, that when decoding the image, we need to import specific packages. One is `"image"`, and there's no wonder. However, as for the second one `_"image/png"`, we didn't actually call any function or use any variable and constant; what we do with it is making use of its initailizer, so we need to add underline before its name, that is called "blank import". To load images in other types, just change the package blank imported.  

## 5. Fairy? Sprite!  
In Pixel, a sprite is created from a `pixel.Picture` object. Assuming we have applied the function `func loadPNG(path string) (pixel.Picture, error)`, let's create our first sprite!  
```Go
pic, err := loadPNG(path)       // Assume path is a defined variable
if err != nil {
    panic(err)                  // To be replaced
}
sprite := pixel.NewSprite(pic, pic.Bounds())
```  
The first arguement is obviously a picture. The second one is literally the bounds of the picture. Actually we can interpret the latter argument as a rectangle, only the portion in which will be showed. (Like watch the sky from a window, wide as the sky is, we could only see that in the window)  
Maybe you have come up with the idea that applying different rectangles on one picture to create different sprites. If so, it's brilliant of you! But we'll discuss it later.  

Then how to draw it to the window? Pixel provides every sprite a function named "Draw" to display it. Here is it:  
```Go
sprite.Draw(win, pixel.IM)
```  
Also two arguments are expected. The fist is the target, illustrating where the sprite will be drawn on. The second is a matrix, demonstrating a conversion. Here `pixel.IM` is namely the "Identity Matrix", which means no conversion.
## 6. Inevitable geometry primitives.
In this secssion, we'll simply talk about the Vector, Rectangle and Matrix.  
* #### Vector  
Vectors are usually used to demonstrate positions, movements, velocities, accelerations and so on. Its defination is like this:  
```Go
type Vec struct {
	X, Y float64
}
```  
The "Zero Vector" is pre-defined in Pixel, that is `pixel.ZV`
Some functions are following:  
```Go
// Create the vector (x, y)
func V(x, y float64) Vec

// Return  u+v
func (u Vec) Add(v Vec) Vec

// Return u-v
func (u Vec) Sub(v Vec) Vec

// Return the projection of u on v
func (u Vec) Project(v Vec) Vec

// Return vector u rotated by the given angle in radians.
func (u Vec) Rotated(angle float64) Vec

// Return vector u whose X and Y are both multiplied by c
func (u Vec) Scaled(c float64) Vec

// Return vector u
// whose X and Y are respectively multiplied by v's X and Y 
func (u Vec) ScaledXY(v Vec) Vec
```  
Notice, that all but the first function never change the value of u and v.  

* #### Rectangle
Rectangles are mainly used to illustrate the frame of pictures and the bounds. It was defined like this:  
```Go
type Rect struct {
	Min, Max Vec
}
```  
The two vectors are respectively the lower-left point and the upper-right point.  
Some functions fo example:  
```Go
// Create a rectangle with points (minX, minY) and (maxX, maxY)
func R(minX, minY, maxX, maxY float64) Rect

// Return the Height of the rectangle
func (r Rect) H() float64

// Return the Width of the rectangle
func (r Rect) W() float64

// Return the center of a rectangle
func (r Rect) Center() Vec

// Check whether a point is within the range of a rectangle
func (r Rect) Contains(u Vec) bool

// Return whether the two rectangles intersect each other
func (r Rect) Intersects(s Rect) bool

// Return the rectangle after a given motion
func (r Rect) Moved(delta Vec) Rect
```  
The same as Vector, none of their value was changed inthe functions. And it's worth cautions that the Rectangle can NOT be rotated, which means its direction is fixed.  

* #### Matrix
Matrices represents all kinds of linear transformations, like movement, scaling and rotation. It was defined as a 2x3 affine matrix:  
```Go
type Matrix [6]float64
```  
> It looks wierd two have a 2x3 affine matrix. Obviously it can not be directly applied on a 2D vector.  
> But what if we by default expand the vector to 3D by adding a "1" after x and y? The vector (x, y, 1) would be our friend.  

Usually we don't necessarily need to create a matrix by filling the 6 value. Instead, we start from an identity matrix, that is `pixel.IM` in Pixel.  
Some functions are as followed. These are important and will frequently occur in the future.  
```Go
// Multiplication of matrices. Namely (next * m)
func (m Matrix) Chained(next Matrix) Matrix

// Similar to the functions mentioned above
// But the transformation is saved in a matrix
func (m Matrix) Moved(delta Vec) Matrix
func (m Matrix) Rotated(around Vec, angle float64) Matrix
func (m Matrix) Scaled(around Vec, scale float64) Matrix
func (m Matrix) ScaledXY(around Vec, scale Vec) Matrix

// Apply all the transformations on m to u
func (m Matrix) Project(u Vec) Vec

// Does the inverse operation to Project
func (m Matrix) Unproject(u Vec) Vec
```

## 7. Where's my mouse and keyboard?
Pixel provides a series of functions to deal with keyboard and mouse events. Here are some examples!  
Assuming that we have got a window `win`, if you wonder whether the mouse is clicked down,You can use method `win.JustPressed(key)`, which returns true when the given argument `key` has just been pressed down, or rather, when the motion "press" occurs. Below shows a simple conclusion of its value available.  
```Go
pixelgl.KeyA
// from A to Z

pixelgl.MouseButtonLeft
// Also Right, Middle and so on

pixelgl.KeyLeftControl
// Also Alt, Shift, etc.

//namely "pixelgl.Key" + keyname
```  
If you want a lasting signal after a key being pressed, you may prefer `win.Pressed(key)`, which checks whether the key's status of being pressed.  
To get the current position of the mouse, you could simply use `win.MousePosition`, a vector recording that infomation.  
What about the wheel on my mouse? we can use `win.MouseScroll()`, which returns a vector recording the amount of two directions by the user. Usually we use its field `Y`.  
## 8. "Hello World!"  
So far, we haven't mentioned how to write texts on the window. And now here is it!  
In short, we should read a font into the memory, then use the imported font to make a string a sprite, and finally draw it on the window.  


