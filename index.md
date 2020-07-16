# Hello Friend

I thought the best use of this cool github pages feature would be a TIL, as seen across various blogs

## TIL (Today I Learned)

* 7-16-2020: The idea of Concurrency vs Parallelism can be summed up with: 
> Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once. 

Which is best visualized by the following picture 
![concurrency_vs_parallelism](/images/concurrency.png)

We see that in the picture, task execution in the Concurrency example is sporadic. The idea is that modern (and even legacy systems) that are relegated to running multiple tasks on 1 core are able to work on a wide variety of tasks, jumping between them so quickly that it appears they are all ocurring at once. This is essentially also how Python handles multi threading.



* 7-13-2020: When calling a python script from within a bash script, print statements from python are not necessarily going to automatically be sent to stdout. To ensure this happens, when calling a python script within a .sh use the following syntax `echo -ne "$(python myscript.py)\n"` - Provided you want newlines 

* 7-12-2020: Though the primary use case for Docker Swarm is task mirroring/replication accross a scalable cluster of nodes, it can also be utilized for parallel execution of tasks through the use of the `.Task.Slot` environmental variable when configuring a service template. More info found here: https://stackoverflow.com/questions/56040426/parallel-code-execution-in-docker-containers
