# Optimal stopping - Does the 37% rule make sense?

So I was reading [Algorithms to live by: Computer Science of Human Decisions](https://www.amazon.com/Algorithms-Live-Computer-Science-Decisions/dp/1627790365) by Brian Christian and Tom Griffiths where in one chapter they talk about Optimal Stopping, specifically the Secretary problem. The history of the problem was detailed and interesting but I was hung up on this 37% rule, does it work. Before I go to far let me briefly describe the Secretary problem (from [Wikipedia](https://en.wikipedia.org/wiki/Secretary_problem))

> Although there are many variations, the basic problem can be stated as follows:
- There is a single position to fill.
- There are n applicants for the position, and the value of n is known.
- The applicants, if seen altogether, can be ranked from best to worst unambiguously.
- The applicants are interviewed sequentially in random order, with each order being equally likely.
- Immediately after an interview, the interviewed applicant is either accepted or rejected, and the decision is irrevocable.
- The decision to accept or reject an applicant can be based only on the relative ranks of the applicants interviewed so far.
- The objective of the general solution is to have the highest probability of selecting the best applicant of the whole group. This is the same as maximizing the expected payoff, with payoff defined to be one for the best applicant and zero otherwise.

So what I want to do is test this. Its going to be simple. 

1. I will create a list of integers from 1 to 100
2. Shuffle those integers
3. Pick the largest number of the first 37 as my benchmark
4. compare the values in the remaining list to that benchmark and stop when I find the value larger than the benchmark.

But wait thats not all, I want to do this multiple times to understand the probability distribution of this method, so I will have "one hundred secretary" jobs to fill 10.000 times then I will plot the distribution of only those values that hit 100, that being the best candidate. So lets do just that.


```python
%timeit
import random
from random import shuffle

primary_counter = 0
primary_counter_sum = []
while primary_counter < 10000:

    counter = 1
    benchmark_list = []
    best_pick = []


    while counter < 100:
        initial = [i for i in range(100)]

        secrateries = [i + 1 for i in initial]

        shuffle(secrateries)

        first37 = secrateries[:37]
        pool = secrateries[37:]
        benchmark = max(first37)
        benchmark_list.append(benchmark)
        for j in range(len(pool)):
            if pool[j] > benchmark:
                best_pick.append(pool[j])
                break
            if benchmark == 100:
                best_pick.append(benchmark)
                break
        counter += 1

    countersum = 0
    for i in best_pick:
        if i == 100:
            countersum += 1
    primary_counter_sum.append(countersum)
    primary_counter += 1
```


```python
import matplotlib.pyplot as plt
from matplotlib import style
import seaborn as sns
style.use("dark_background")

plt.hist(primary_counter_sum, bins = 100)
plt.show()
```


![png](https://github.com/EirikurJonsson/EirikurJonsson.github.io/blob/master/images/page4_images/output_2_0.png?raw=true)


Well thats very telling (I am trying the dark theme for matplotlib, seems to be working ok I guess). I am surprised that this method is hovering around 73% of finding the optimal best. That last statements is almost a lie, I did use 37% of the data as benchmark, so seeing that the distribution of finding the optimal solution is hovering there isn't that surprising, but still really cool. This does however show how a fairly "simple" solution can yield good results. I say simple with quotes since its elegant in its simplicity, showcasing again that the simpler the solution statement - the more complex it really is.

This may lead someone to think that this is not applicable in real life, well I got to tell you that the authors of the book would disagree. Optimal stopping is a subfield onto itself and a very interesting one, methods like gradient decent, simulated annealing and more are a part of ["probability techniques for approximating the global optimum given a function"](https://en.wikipedia.org/wiki/Simulated_annealing).

I for one found this to be very surprising and a fun project to work on with random numbers. Its not very complicated as such but the implications are deep - consider the questions "How many houses should I look at before I buy?" (replace car, TV or whatever in there), "How many parking spaces should I pass before picking one?" and more trivial questions that have given a rise to optimization problems being their own subfield of CS and mathematics. 
