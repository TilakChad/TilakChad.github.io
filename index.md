<script type="text/javascript" async
 src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

# Welcome to Tilak Chad Pages

Greetings, this is my first blog on programming (or anything else). I hope you would enjoy it.

### [Graphics Programming] 
## Rendering smooth anti-aliased thick lines 
It sure sounds easy, doesn't it? Its just rendering a line not a world, you say. But drawing perfect smooth (anti-aliased) thick lines is quite a challenging task. 
We will go through some possible options of rendering lines.

So let's get started. 

## The obvious way 
The obvious way to draw thick lines in renderer is to use builtin function to adjust line thickness. 
In case of OpenGL, it is <br>
`glLineWidth(width)`, where width is in screen pixel <br>
<br>
DirectX has no such builtin function but don't worry. You aren't alone who can't draw nice lines xD. There's no current GPU that can draw thick lines natively in the hardware. OpenGL fakes the thick line by drawing polygons. 

The output for a typical line with width 15 as drawn by OpenGL is : 

<p align="left">
	<img src = "./include/gl_line_width.png">
</p>


Sounds fine, right? 
Let's see what happens if we try to draw line_strips : 

<p align="left">
	<img src = "./include/not_believe.png">
</p>

You mightn't believe it, but these two lines have same width, according to OpenGL. And yes, they do have, in a sense.  
The reason why their width looks different, as drawn by OpenGL, is explained in GL docs. If del(X) >= del(Y), 
`width` pixels are filled along each column. But, since line is tilted, the total length across its normal boundaries is less than its vertical width. Actual experimentation is left to the reader as an exercise. :D  

We will solve this problem. In fact, its quite easy to solve this problem. Even if we fixed the line width somehow, we are still left with another problem. 
It will be fine for drawing a single line, but while rendering multiple connected line using LINE_STRIP topology, we can clearly see the disconnected lines ruining our beautiful lines. 


### Drawing Lines, the easy way 

The first approach to try to render fixed width lines is not drawing lines at all. Instead, using TRIANGLES topology to plot a thin quad that will approximate line quite nicely. 
Notice the lines below, we want lines endpoint to be normal to the direction of line, rather than being vertical or horizontal. 

<p align="left">
	<img src = "./include/line_vec.png">
</p>

In the figure above, 

Let <br> $$ A = (x0,y0)$$ <br> $$ B = (x1, y1) $$ <br>
Vector from A to B is given by, <br>
$$ \vec{L} = B - A = (x1 - x0, y1 - y0) $$ <br>
(A slight abuse in notation)<br>

There are two vectors normal to this in 2D Euclidean plane. The counterclock one to the current direction of line is obtained as : <br>
$$ \vec{n} = (L.y, -L.x) $$ <br><br>
L and $$\vec{n}$$ are orthogonal to each other as can be easily verified using dot product. 

Normalizing the normal vector $$\vec{n}$$, <br> <br>
$$ \vec{n} = \frac{\vec{n}}{||\vec{n}||} $$ <br> <br>
This gives unit vector perpendicular to the line's direction. 
Scaling it by half the thickness, <br>
$$\vec{n} = \vec{n} * \frac{t}{2}$$ <br>

Now adding this normal vector to both end and subtracting it to both end, gives four co-ordinates that approximates required thick line. 

The resulting quad need to be drawn using TRIANGLES topology now. 

<p align="left">
	<img src = "./include/line_show.png">
</p>

Two trianges need to be drawn for each line now, 
namely 
`Triangle(P0,P1,P2)` and `Triangle(P1,P2,P3)`

Now the line looks quite good. At least better than it was previously. 

<p align="left">
	<img src = "./include/good_line.png">
</p>

Its good that we can now draw line with constant width with their endpoints perpendicular to their axis. 
The quest of drawing perfect line isn't over yet.

We can draw `good` single lines, but while drawing as line strips, we can see the broken artifacts : 

With OpenGL's default line drawing, the graph of $$ sin(x) $$ roughly looks : 
<p align="left">
	<img src = "./include/broken_line2.png">
</p>

Our above implementation yields, quite good lines, yet not how we want : 
<p align="left">
	<img src = "./include/line_gap.png">
</p>

## Breaking the break 

To remove those gaps at the joint, we need to fill
<p align = "left"> 
	<img src = "./mid_calc.png"> 
</p>

Either magnitude of t or s need to be determined to precisely find the point where the two lines will meet at the tip. 

Let cn be the vector perpendicular to c and dn be the vector perpendicular to d. 
cn and dn can be easily determined just like we determined n earlier. 

Using Vector geometry, 

$$ \vec{c} + s * \vec{cn} = \vec{d} + t * vec{dn} $$ 

Taking dot product with c on both side, 

$$ <\vec{c},\vec{c}> + s * <\vec{c},\vec{cn}> = <\vec{d},\vec{c}> + t * <\vec{dn},\vec{c}> $$ where <a,b> denotes the inner product of a and b 

$$ <\vec{c}, \vec{cn}> = 0, since they are othogonal to each other. $$ 

Rearranging,

$$ t = \frac{<\vec{c}, \vec{c}> - <\vec{c},\vec{d}>} {<\vec{c},\vec{dn}>} 

$$\vec{m} = \vec{d} + t * \vec{dn} $$ 

If this calculation feels too long, you can take 
$$\vec{m} = \frac{\vec{c} + \vec{d}}{2} $$

It might be enough to get close approximation to above point. 

Finally, we again emit two new triangles at each joint to seal off the gap, and have nice, dis-disconnected lines. 
We now have our lines like we always wanted to. 

<p align = "center">
	<img src = "./include/fill_gap.png">
</p>

## Almost perfect lines 
The gaps are filled by calculating the point of intersection of line end. So when lines are close to parallel, the gap appears more pointed and away from the lines like : 

<p align = "center"> 
	<img src = "./include/pointed.png"> 
</p>

The solution is simple. Just limit the magnitude of $$\vec{m}$$. 

$$\vec{m} = \frac{\vec{m}}{||\vec{m}||} * cutoff_length $$ 

An extra triangle might need to be emitted for this case to draw pixel perfect line. The edge might appear a little pointed but it looks fine. 

The resulting output becomes : 

<p align = "center"> 
	<img src = "./include/not_bad.png"> 
</p> 

Doesn't look bad, right? 

## Achieving Perfection

```c
#include <stdio.h> 

int main(int argc, char** argv)
{
  return 0; 
}
```
TODO :: use math script 
TODO :: Complete it :D
Blog in progress ... 


### References
