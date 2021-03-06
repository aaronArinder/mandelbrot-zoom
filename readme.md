### Overview of the fractal zoom generator for the Mandelbrot set
This project generates a zoom of the Mandelbrot set using Node's cluster module, piping individual fractal frames to a spawned ffmpeg. The code for generating an individual fractal frame is based on Rosetta Code's fractal code. An explanation of the math is below in this README.

### How to use this project to generate a fractal zoom

To run the fractal generator, do the following:

- start the server: `npm run server`

The server is a redis-like server meant to make sharing memory between the child processes easier. When I started this project, worker threads weren't stable. They became stable in Node 12.11.0. Eventually, I'll convert it to use worker threads.

- generate a fractal zoom with n number of frames: `npm run generate n`, where n is some number. E.g., `npm run generate 100` for a short zoom, or `npm run generate 3000` for a longer zoom.
  - you _MUST_ pass in some number of frames

A file called `fractal-zoom.mp4` will be generated in the project's home directory.

### Future improvements
There are some problems with this fractal zoom that I want to figure out.

- allow folks to pass in a name for the mp4 rather than hard-coding it to `fractal-zoom.mp4`
- figure out a better `consumeHash`: `setImmediate` is needed to keep from blowing the stack, but I'm not sure why. I tried a tail-call-optimized recursive function that immediately invoked `consumeHash`, but it kept blowing the stack.
- implement Big.js, a library for big numbers. Right now, if you generate a bunch of frames, eventually you'll run into lack of precision in the floats used to generate thee fractals. The dep is already in the package.json, but I'm putting off its implementation until I can figure out how to gracefully shutdown the child processes and pipes.
- the transforms are saved in the server, but only to a certain depth of zoom. More transforms will be generated for longer zooms, and those new transforms should be saved to a file to be used for later zooms.
- see the notes in `shutdown()` in `engine.js`, but there's some funk going on with the master process not shutting down when generating a zoom with 3k+ frames.


### Math for mandelbrot set

The mandelbrot set is the set of complex numbers that, under a particular recursive function, don't grow or shrink infinitely. The recursive function is f_c(z) => z^2 + c. The function is then called with the result of its previous invocation. That is,

```
f(0) => 0^2 + c;
so, f(z) === c;
```

the next invocation uses the result of the first
```
so, f(c) => c^2 + c;
```

And let's unpack part of that. c^2 is a complex number squared. A complex number is made up of two parts, a real and imaginary. Let `a` represent the real part, and `bi` represent the imagine part, `b`, multiplied by `i`, the imaginary number.

```
so, c^2 === (a + bi)(a + bi);
so, the above, c^2 + c can be rewritten to: (a + bi)(a + bi) + c;
```

```
since (a + bi)(a + bi) === a^2 + abi + abi + (b^2)(i^2),
and since (i^2) === -1 by definition of the imaginary number,

we can rewrite c^2 + c to a^2 + 2abi - b^2,
and so, f(c) = a^2 + 2abi - b^2 + c;

a^2 + 2abi - b^2 + c can be rewritten too a^2 - b^2 + 2abi + c, which is on the complex plane
```

so, we can figure out which points are in the  mandelbrot set by figuring out whether a^2 - b^2 + 2ab tends toward positive or negative infinity when z begins at 0


