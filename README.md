A long time ago, in a galaxy far, far away ...

A wise and powerful wizard lived in a small village in the middle of the desert. And his name was Dumbledalph. He was not only wise and powerful, but also helped people who came from distant lands to ask for help from the wizard. Our story began when a traveler brought a magic scroll to a wizard. The traveler did not know what was in the scroll, he only knew that if anyone could reveal all the secrets of the scroll, it would be Dumbledalf.

# Chapter 1: Single-threaded, single-process

In case you haven't already guessed, I was drawing an analogy with a processor and its functions. Our wizard is the processor, and the scroll is a list of links that lead to Python's power and knowledge to master it.

The first thought of the wizard, who easily deciphered the list, was to send his faithful friend (Garrigorn? I know, I know, that sounds terrible) to each of the places that were in the scroll to find and bring what he found there.

```python
In [1]:
import urllib.request
from concurrent.futures import ThreadPoolExecutor
In [2]:
urls = [
  'http://www.python.org',
  'https://docs.python.org/3/',
  'https://docs.python.org/3/whatsnew/3.7.html',
  'https://docs.python.org/3/tutorial/index.html',
  'https://docs.python.org/3/library/index.html',
  'https://docs.python.org/3/reference/index.html',
  'https://docs.python.org/3/using/index.html',
  'https://docs.python.org/3/howto/index.html',
  'https://docs.python.org/3/installing/index.html',
  'https://docs.python.org/3/distributing/index.html',
  'https://docs.python.org/3/extending/index.html',
  'https://docs.python.org/3/c-api/index.html',
  'https://docs.python.org/3/faq/index.html'
  ]
In [3]:
%%time

results = []
for url in urls:
    with urllib.request.urlopen(url) as src:
        results.append(src)

CPU times: user 135 ms, sys: 283 µs, total: 135 ms
Wall time: 12.3 s
In [ ]:
```
As you can see, we just iterate over the URLs one by one with a for loop and read the response. Thanks to %% time and the magic of IPython, we can see that with my sad internet it took about 12 seconds.

# Chapter 2: Multithreading

It was not for nothing that the wizard was famous for his wisdom, he was quickly able to come up with a much more effective way. Instead of sending one person to each place in order, why not gather a detachment of reliable companions and send them to different parts of the world at the same time! The wizard will be able to combine all the knowledge that they will bring at once!

That's right, instead of looping through the list sequentially, we can use multithreading to access multiple URLs at the same time.

```python
In [1]:
import urllib.request
from concurrent.futures import ThreadPoolExecutor
In [2]:
urls = [
  'http://www.python.org',
  'https://docs.python.org/3/',
  'https://docs.python.org/3/whatsnew/3.7.html',
  'https://docs.python.org/3/tutorial/index.html',
  'https://docs.python.org/3/library/index.html',
  'https://docs.python.org/3/reference/index.html',
  'https://docs.python.org/3/using/index.html',
  'https://docs.python.org/3/howto/index.html',
  'https://docs.python.org/3/installing/index.html',
  'https://docs.python.org/3/distributing/index.html',
  'https://docs.python.org/3/extending/index.html',
  'https://docs.python.org/3/c-api/index.html',
  'https://docs.python.org/3/faq/index.html'
  ]
In [4]:
%%time

with ThreadPoolExecutor(4) as executor:
    results = executor.map(urllib.request.urlopen, urls)

CPU times: user 122 ms, sys: 8.27 ms, total: 130 ms
Wall time: 3.83 s
In [5]:
%%time

with ThreadPoolExecutor(8) as executor:
    results = executor.map(urllib.request.urlopen, urls)

CPU times: user 122 ms, sys: 14.7 ms, total: 137 ms
Wall time: 1.79 s
In [6]:
%%time

with ThreadPoolExecutor(16) as executor:
    results = executor.map(urllib.request.urlopen, urls)

CPU times: user 143 ms, sys: 3.88 ms, total: 147 ms
Wall time: 1.32 s
In [ ]:
```
Much better! Almost like ... magic. Using multiple threads can significantly speed up many I / O tasks. In my case, most of the time spent reading URLs is due to network latency. Programs tied to I / O spend most of their life waiting, you guessed it, for input or output (similar to how a wizard waits for his friends to travel to and from a scroll). It can be I / O from a network, a database, a file, or from a user.This I / O is typically time consuming because the source may need to do some preprocessing before sending data to the I / O. For example, the processor will count much faster than the network connection will transfer data (about the speed of Flash versus your grandmother).

Note: multithreading can be very useful for tasks such as cleaning up web pages.

# Chapter 3: Multiprocessing

As the years passed, the fame of the good wizard grew, and with it the envy of one impartial dark wizard grew (Sarumort? Or maybe Volandeman?). Armed with immeasurable cunning and driven by envy, the dark wizard cast a terrible curse on Dumbledalf. When the curse overtook him, Dumbledalph realized that he had only a few moments to repel it. Desperate, he rummaged through his spellbooks and quickly found one counter-spell that should have worked. The only problem was that the wizard had to calculate the sum of all prime numbers less than 1,000,000. A strange spell, of course, but what do we have.

The wizard knew that calculating a value would be trivial if he had enough time, but he didn't have that luxury. Despite the fact that he is a great wizard, he is still limited by his humanity and can check for simplicity only one number at a time. If he decided to just sum the primes one after another, it would take too much time. When only a few seconds were left before applying the counter-spell, he suddenly remembered the multiprocessing spell he had learned from a magic scroll many years ago. This spell will allow him to copy himself in order to distribute the numbers among his copies and check several at the same time. And in the end, all he has to do is just add up the numbers that he and his copies find.

```python
In [1]:
from multiprocessing import Pool
In [2]:
def if_prime(x):
    if x <= 1:
        return 0
    elif x <= 3:
        return x
    elif x % 2 == 0 or x % 3 == 0:
        return 0
    i = 5
    while i**2 <= x:
        if x % i == 0 or x % (i + 2) == 0:
            return 0
        i += 6
    return x
In [17]:
%%time

answer = 0

for i in range(1000000):
    answer += if_prime(i)

CPU times: user 3.48 s, sys: 0 ns, total: 3.48 s
Wall time: 3.48 s
In [18]:
%%time

if __name__ == '__main__':
    with Pool(2) as p:
        answer = sum(p.map(if_prime, list(range(1000000))))

CPU times: user 114 ms, sys: 4.07 ms, total: 118 ms
Wall time: 1.91 s
In [19]:
%%time

if __name__ == '__main__':
    with Pool(4) as p:
        answer = sum(p.map(if_prime, list(range(1000000))))

CPU times: user 99.5 ms, sys: 30.5 ms, total: 130 ms
Wall time: 1.12 s
In [20]:
%%timeit

if __name__ == '__main__':
    with Pool(8) as p:
        answer = sum(p.map(if_prime, list(range(1000000))))

729 ms ± 3.02 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
In [21]:
%%timeit

if __name__ == '__main__':
    with Pool(16) as p:
        answer = sum(p.map(if_prime, list(range(1000000))))

512 ms ± 39.8 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
In [22]:
%%timeit

if __name__ == '__main__':
    with Pool(32) as p:
        answer = sum(p.map(if_prime, list(range(1000000))))

518 ms ± 13.3 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
In [23]:
%%timeit

if __name__ == '__main__':
    with Pool(64) as p:
        answer = sum(p.map(if_prime, list(range(1000000))))

621 ms ± 10.5 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
In [ ]:
```
Modern processors have more than one core, so we can speed up tasks by using the multiprocessing module. Processor-bound tasks are programs that perform most of their time on the processor (trivial math, image processing, etc.). If calculations can be performed independently of each other, we have the opportunity to divide them between the available processor cores, thereby obtaining a significant increase in processing speed.

All you have to do is:
  1 . Determine the function to apply
  2 . Prepare a list of elements to which the function will be applied;
  3 . Spawn processes with multiprocessing.Pool. The number that will be passed to Pool () will be equal to the number of spawned processes. Embedding a with          statement ensures that all processes are killed when they exit.
  4 . Combine the output from the Pool process using the map function. The input data for map will be a function applied to each element and the list of elements        itself.
  
 Note: A function can be defined to perform any task that can be performed in parallel. For example, a function might contain code to write the result of a calculation to a file.
 
 So why do we need to separate multiprocessing and multithreading? If you've ever tried to improve the performance of a task on a processor using multithreading, the effect is exactly the opposite. That's just terrible! Let's see how it happened.
 
 Just as a wizard is limited by his human nature and can only compute one number at a time, Python comes with a thing called the Global Interpreter Lock (GIL). Python will happily let you spawn as many threads as you want, but the GIL ensures that only one of those threads will be executing at any given time.
 
 For an I / O task, this is perfectly normal. One thread sends a request to one URL and waits for a response, only then this thread can be replaced with another, which will send another request to a different URL. Since a thread doesn't have to do anything until it receives a response, it makes no difference that there is only one thread running at a time.
 
 For tasks running on a processor, having multiple threads is almost as useless as nipples on armor. Since only one thread can execute at a time, even if you spawn several, each of which will be allocated a number to check for simplicity, the processor will still only work with one thread. Essentially, these numbers will still be checked one by one. And the overhead of working with multiple threads will contribute to the performance degradation that you can see when using multithreading in tasks running on the processor.
 
 To get around this "limitation", we use the multiprocessing module. Instead of using threads, multiprocessing uses, how would you say ... multiple processes. Each process gets its own interpreter and memory space, so the GIL will not limit you. Basically, each process will use its own processor core and work with its own unique number, and this will be executed simultaneously with the work of other processes. How nice of them!
 
 You may notice that the CPU load will be higher when you use multiprocessing compared to a regular for loop or even multithreading. This is because your program uses more than one core. And this is good!
 
 Remember that multiprocessing has its own multi-process overhead, which is usually greater than the overhead of multithreading. (Multiprocessing spawns separate interpreters and assigns its own memory region to each process, so yes!) That is, as a rule, it is better to use the light version of multithreading when you want to get out this way (think about tasks related to I / O). But when computing on the processor becomes a bottleneck, the time for the multiprocessing module comes. But remember that with great power comes great responsibility.
 
 If you spawn more processes than your processor can handle per unit of time, you will notice that performance starts to drop. This is because the operating system has to do more work shuffling processes between processor cores, because there are more processes. In reality, everything can be even more complicated than I told you today, but I conveyed the main idea. For example, on my system, performance will drop when the number of processes is 16. This is because my processor has only 16 logical cores.
 
# Chapter 4: Conclusion

  For I / O tasks, multithreading can improve performance.
  For I / O tasks, multiprocessing can also improve performance, but the overhead tends to be higher than multithreading.
  The existence of the Python GIL makes it clear that only one thread can be running in a program at any given time.
  For CPU-bound tasks, using multithreading can degrade performance.
  For CPU-bound tasks, using multiprocessing can improve performance.
  The wizards are amazing!
  
  
 This concludes our introduction to multithreading and multiprocessing in Python today. Now go and win!
