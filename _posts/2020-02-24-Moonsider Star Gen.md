---
layout: post
title: Moonsider- Galaxy Generation (Part I)
excerpt: A couple of days ago, I thought I'd throw together some ideas of creating a galaxy in Unity for one of my current projects, Moonsider...
---

![Generated Spiral Galaxy](https://media.discordapp.net/attachments/85593628650504192/679646988660375562/unknown.png?width=527&height=503)

## To Infinity..

A couple of days ago, I thought I'd throw together some ideas of creating a galaxy in Unity for one of my current projects, Moonsider. 
I did some research, seeing what assets were already available, finding articles on other implementations people have tried, and generally immersing myself in the Science Fiction themes again. 

When I opened up Unity, I already had a vague idea of what I needed to do. I knew I wanted to create a Spiral Galaxy, with stars that were more frequent towards the 'core' of the galaxy. 
This is also something I wanted to do at scale. 

I want a lot of stars, split across a few different arms, and I wanted to put it goether in a way that the star count, the arms, the rotation, and the spread could all be very easily changed to acheive different results.
For me as a developer, it's really important that the code I write can be easilly picked up by a designer or someone who does _not_ understand the code, to effectively use the product of the code in the most efficient and safest way possible.
In secret, part of the reason I do this, is because I know I will forget how this all hangs together and how it works in six months time, unless I take the time to control the editor!

---

## Approach
Knowing roughly what I needed to do, I set out pretty quickly using my 'trial and error' approach. More often than not, it ends in burn-out and frustration. But for this system, it came together fairly well, other than a few key frustrations I had along the way. 


### Arm Ends
First off, I started with creating the arms. 
Three things were known. The center point, the arm reach, and the number of arms.
It was quite easy to loop over the number of desired arms, find the distance needed between each arm, and extend them out for as far as they should reach, so that's exactly what I've done. 
I have made the assumption that the galaxy is flat (no Z variation) - so this was not factored in to the generation of the arm position. It would mean that generated galaxies could be a little more interesting, if there was some slight variation in Z value, but for now I've kept it only to X and Y points. 

{Image of galaxy arms}


### Star distribution
So, how are the number of stars split between the number of arms? Well, for this part in particular, I haven't gone with exact figures. I wanted a little more randomness, a bit of 'fuzz', for how many stars were present in an arm. In the end, it doesn't really matter all that much, as things are twisted and skewed to the point that you couldn't tell which arm a star belonged to anyway. 

To start with, I do find roughly the amount of stars that should be present in each arm. 100 stars across 4 arms, 25 stars. Simple. I wanted this to look a little more random, 29/22/26/23 across each arm - something like that. 
What I've done to acheive this, is find an upperLimit and lowerLimit - which is the even amount of stars (25), plus or minus a small amount (5%), respectively. I would then generate a random value between these limits, and multiply the original star count by this value. 

For those of you who are keen on math, you'll quickly recognise that the _actual_ number of stars may never add up to the total (or even be over) the _desired_ amount of stars. I have chosen to view this as not being a problem. It's something I'm more happy to hide behind an enum of 'Galaxy Size' - where 'Large' might be 150-200 Stars, so anything in that range is perfectly acceptable.

Both this realization, and the math itself behind understanding the "fuzziness" were helped by a long-time friend of mine, and I'm very appreciative of his efforts and patience in explaining concepts to me that I'm sure are simple to most! 
He's working on some of his own games, and has a fascination with the mathemeatical side of gamedev, and manipulating numbers through code. 

### Star Spread
This is where things get a little more interesting. Of course, I already mentioned that I wanted the stars to be more common towards the core of the galaxy. An [Animation Curve](https://docs.unity3d.com/ScriptReference/AnimationCurve.html) was perfect for this, as it becomes something that is entirely customizable inside the editor, and there's no complicated maths that's involved. For placing the stars closer to the cetner, all I had to do was multiply the armReach by a value on this curve.

This was enough to place the stars out at random points along the lines, but they were all still very.. linear. 
There had to be some kind of variance in their distance _out_ from the arm, not just from the core, to make things feel a little more natural

For this, I used a very similar approach, with a different animation curve. 
Though what I've done here, is taken a `Random.onUnitSphere`, where the radius of the arm is provided from the editor, and then multiplied by a sample from this new curve. 
Using `Random.OnUnitSphere`gave nice results - the stars were still _potentially_ close to the center, but were definitely spread out more the further down the arm they got. 

As I mentioned before, it's a 2D planar galaxy, so the Z position was forcefully set back to 0, so that only the X and Y positions were placed.

{Image of star distribution}


### A problem occurs
However, this is where things are falling apart for me a _little bit_. Using this method, there seems to be the odd star that is placed _way out of line_ - even behind, sometimes. 
I'm still trying to understand the solution I have created, to see if I can minimise this, because it actually starts to have a flow on effect to the next steps, spinning the galaxy. 

I've tried a few things, shifting around parts of the forumalae, isolating this part of the code, with nothing but confirmation it's the way I'm shifting the stars away from the arms, that's causing this issue to occur.
Out of about 100 stars, it seems to only happen for a pretty small percentage of stars, but is a little frustrating, because the stars are _more than a little_ out of whack. 


### You spin my heard right round..
This is where the fun begins.. 
With the stars properly spread closer to the core/center of the arms, which I'm _reasonably_ happy with (other than the small issue above), it was time to twist the galaxy so it looked a little more like a spiral, rather than a bicycle wheel with missing spokes. 

When I say fun... I mean a piece of math that I cannot comprehend. Quite ironic, being a developer who doesn't have a firm grasp on mathematics, but here I am!

It was easy enough to calculate the rotation _amount_ - and that was the distance of the star from the center, multiplied by a new `spinFactor` variable. 
This gave me a nice number, which represented stars _closer_ to the core having completed a few more revolutions, with ones further out lagging behind. 

{Image of debug distances}

I was able to conceptually put together exactly what it was that I was looking for. 
In my mind, it was quite simple. Move one point X degrees around another point. There were even plenty of examples online.
I exhausted quite a few solutions I found around Unity Answers and Stack Overflow, but none seemed to offer the right solution, even with an understanding of what the code was doing, and attempted tweaks to remedy it.

In the end, I reached out on a Discord server with my simple understanding of what needed to happen, and was helped with some maths that I struggle to understand. I really should have paid attention in Trigonometry. 

![My simple understanding](https://media.discordapp.net/attachments/85593628650504192/679645243888631824/unknown.png?width=593&height=683)

It worked! I was able to retreive the result I wanted, and other than the misplaced stars, I am now very happy with this solution!


## And Beyond!
Well, I now have a `Dictionary<int,List<Vector3>>` which is providing me with a nice clean pattern of which stars are in which arms. 
I have a well presented and well documented editor that allows for different kinds of generation, and I've also overwridden Random to use a text-based seed, so that identical results can be produced if desired. (And I do desire it!)

There's still a little more work I want to do with the Random seed, mostly in creating a helper function and properly override the method, so that I can call it from anywhere, and feed a single seed to get the _same_ 'random results' each time.

But for the galaxy generation... where to next?

Next up, I need to generate the individual stars names, temperatures, sizes, and all of that jazz.
I've done some [reading](http://jongware.com/galaxy1.html) on an approach that's used in one of the earlier Elite titles, which does some fancy stuff so that none of this information is ever _stored_ - it is all calculated at runtime, based on nothing but the coordinate of the star.

I've already got a working implementation of the rolling bit rotation based name generation, but more on that next time! 
Planning on doing some work between now and then (When?) on the above, and I'll report back when I've got something more exciting to show!

### \>B.
