# Hello Friend

I thought the best use of this cool github pages feature would be a TIL, as seen across various blogs

## TIL (Today I Learned)

* 8-17-2020: Though the terms bytecode and assembly are sometimes (incorrectly) used interchangeably, bytecode refers to the product of the compilation of code that will then be consumed by a virtual machine or virtual environment, bytecode is intended to be consumed by another piece of software. Assembly language is infact neither compiled nor interpreted but rather _assembled_ as the name suggests, since machines can read the code as is, it just simply has human readable syntax such as `MOV`.

* 8-5-2020: The HTTP Strict Transport Security policy (set by the Strict-Transport-Security HTTP header) defines a timeframe where a browser must connect to the web server via HTTPS before the connection is terminated

* 7-19-2020: TIL basics of the Go programming language. Go is a statically typed language with strong emphasis on readibility. It feels, and operates like a hardcore C language but looks much closer to Python 🐍. It supports both running .go files in a Go runtime env or compilation of Go files into the executable for your native arch. It relies heavily on implication, using implicit variable declarations to circumvent the cumbersome var type declarations of languages like C or Java. Implementations of interfaces is also implicit. If for example you have an Animal Interface with a `Species` method that returns the Animal's species, and you have a Dog Class, the Dog type can implement the Animal interface by simply instantiating an instance of the Animal Interface passing a Dog instance to it such as 
```
my_dog := Animal(Dog{})
```
Which means you you can call `my_dog.Species()` 
Additionally, I've noticed speed gains on even some of the most trivial code in comparison to its Python counter parts. While this is to be expected, it's still a very pleasant surprise. I hope soon to be working on a project that can utilize Go routes, as parallelism is currently one of my obsessions

* 7-16-2020: The idea of Concurrency vs Parallelism can be summed up with: 
> Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once. 

Which is best visualized by the following picture 
![concurrency_vs_parallelism](/images/concurrency.png)

We see that in the picture, task execution in the Concurrency example is sporadic. The idea is that modern (and even legacy systems) that are relegated to running multiple tasks on 1 core are able to work on a wide variety of tasks, jumping between them so quickly that it appears they are all ocurring at once. This is essentially also how Python handles multi threading.



* 7-13-2020: When calling a python script from within a bash script, print statements from python are not necessarily going to automatically be sent to stdout. To ensure this happens, when calling a python script within a .sh use the following syntax `echo -ne "$(python myscript.py)\n"` - Provided you want newlines 

* 7-12-2020: Though the primary use case for Docker Swarm is task mirroring/replication accross a scalable cluster of nodes, it can also be utilized for parallel execution of tasks through the use of the `.Task.Slot` environmental variable when configuring a service template. More info found here: https://stackoverflow.com/questions/56040426/parallel-code-execution-in-docker-containers
