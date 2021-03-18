# Hello Friend

I thought the best use of this cool github pages feature would be a TIL/Technical blog, welcome!

## TIL (Today I Learned)

* 3-18-2021: Not much of a TIL but today I published several solutions to various KringleCon2020 challenges, very overdue since I had these commits sitting since Dec of 2020 but better late than never! Partial writeup can be found [here!](https://github.com/edrapac/KringleCon2020)

* 3-02-2021: TIL of 2 very useful linux CLI tricks involving `awk` and `find` respectively. The first trick is that `awk` can be used to fuzzy match on columns of text. Let's say that we have a 3 column file, `myfile.txt` like the one below:
```
testing		foobar		thisisatest
```

I want to fuzzy match on column 2, and look for anything that has "foo" in it. Well with `awk`, that is as simple as sending `myfile.txt` to awk using the following command

```
awk '$2 ~ /foo/' myfile.txt
``` 
It's not the most intuative notation to read, but it is incredibly effective. 

The second trick involves `find` a tool which, the more I use it the more I realize its powers extend far beyond simple directory searches. Using `find` it is possible to not only execute complex and useful search queries but to also execute commands on the found files/directories. `find` has a very powerful feature called `execdir` which allows you to execute a command or script from the context of the directory in which your `find` query found a match. 

For example lets say we have a simple directory structure as below:

```
a
├── b
│	└── foo
├── c
│	└── foo
├── d
├── e
```
We want to find which directory contains a file called `foo` and then run a command on that file. To do so, we can use something like the following command
```
find -name "foo" -type f -execdir somecommand \;
```
Which will execute `somecommand` within the context of the directories containing the `foo` file. This is akin to manually `cd`'ing into the directory containing the file and running the command. Insanely useful!

* 12-15-2020: TIL that if crafting a CSRF Proof Of Concept attack, the use of other methods besides POST and GET require the attack to invoke an Ajax API (typically XHR) due to the fact that <a href="https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#attr-fs-method">HTTP Forms are limited to only the POST and GET Methods</a>   

* 12-02-2020: Over a month since I've last posted, yikes. Today's post is much less of a TIL and more of a "Today I Found" post. Today I Found (rediscovered rather) the great folks over at Pentesterlab (https://pentesterlab.com/) a website chock full of free and paid resources for learning more about pentesting. I really enjoyed some of their free challenges, especially some of their entry level scenarios which do a great job highlighting some common vulnerabilities and steps to exploitation, they even have the lab ISOs right there free of charge. In fact, I enjoyed their `SQLi to Shell` entry challenge enough that it prompted me to write a short walkthrough, you can find that <a href="articles/Pentester_Lab_SQLi_to_Shell.md">here</a>


* 10-23-2020: Been quite busy with Work and a relocation but rest assured I am back! TIL how to add a new Hard Disk to an ESXi host, all through the convenient vSphere Center interface. First and foremost, you need to install the drive in an unoccupied drive bay on your server. Boot that bad boy up, and head over to your Storage menu. First look in the Devices tab, this will list all of the currently available storage devices. Make notes of their names. Now go to your Datastores menu, click on all your created Datastores and observe that in the VMFS details tab, they show what device the datastore belongs to. All you need to do by process of elimination is find which device doesnt have any datastores associated with it. Thats the one you want to initalize, which can be done from that Devices menu as well.

* 9-8-2020: Short and sweet today. TIL of a really useful linting and autocorrect tool for python files,the tool autopep8 will automatically edit a python file to conform to PEP8 standards. This normally is a best practice sort of thing, but indentation (which is totally capable of breaking your code) also falls under this realm. Use the following command `autopep8 -i your_file` to fix any indentation errors you might have. 

* 9-5-2020: Hard to believe it's already September and I still haven't had a haircut since March lol...
TIL the weird ways in which we measure wifi strength. There are essentially 2 main methods:
	* RSSI (Recieved Signal Strength Indicator) - A non-standardized value that is essentially measured based on the observed gain of the wireless signal. This value is calcualted by the receiver and is different accross vendors (non standardized). Some wireless manufacturers use a scale from 0-255 whereas others use 0-60
	
	* dBm - Decibels in relation to milliwats. Decibels are just a generic "thing" that can be used to represent a relation to something more absolute (in this case milliwatts). It's just a ratio to make thing's easier because measuring wifi in mW (a unit of power) would require very, very small numbers (like 0.0001 for -40 dBm which is a fantastic wifi signal). The decibel itself is a relative measurement, as previously stated it is used to express a ratio. In regards to RSSI we say that dBm is the absolute measurement because it is has been standardized, which is that 0 dBm = 1 mW
	dBm are thus always expressed on a negative scale as it is impossible to have a signal strength of 0 dBm or 1 mW. It is a logarithmic scale, where 3 and 10 have a particular significance. +- 3 dB doubles or halves the strength and +- 10 dB changes the dBm by a factor of 10
		1. -3 dBm = 0.5 mW
		2. -10 dBm = 0.1 mW
		3. -20 dBm  = 0.01 mW


* 8-17-2020: Though the terms bytecode and assembly are sometimes (incorrectly) used interchangeably, bytecode refers to the product of the compilation of code that will then be consumed by a virtual machine or virtual environment, bytecode is intended to be consumed by another piece of software. Often, when something is compiled to bytecode it is then fed to it's VM in the example of languages like Java and Lua. Assembly language is infact neither compiled nor interpreted but rather _assembled_ as the name suggests, since machines can read the code as is, it just simply has human readable syntax such as `MOV`.

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
