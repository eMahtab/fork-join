# Fork Join

The fork/join framework provides support for parallel programming by splitting up a task into smaller tasks to process them using the available CPU cores.

In fact, Java 8's parallel streams and the method Arrays#parallelSort use under the hood the fork/join framework to execute parallel tasks.

**Applying a divide and conquer principle**, the framework recursively divides the task into smaller subtasks until a given threshold is reached. This is the fork part.

Then, the subtasks are processed independently and if they return a result, all the results are recursively combined into a single result. This is the join part.


![Fork Join](fork-join.gif?raw=true)


To execute the tasks in parallel, the framework uses a pool of threads, with a number of threads equal to the number of processors available to the Java Virtual Machine (JVM) by default.

**💥💥💥 Note :**

Each thread has its own double-ended queue (deque) to store the tasks that will execute.

A deque is a type of queue that supports adding or removing elements from either the front (head) or the back (tail). This allows two things:

A thread can execute only one task at a time (the task at the head of its deque).
A work-stealing algorithm is implemented to balance the thread’s workload.
With the work-stealing algorithm, threads that run out of tasks to process can steal tasks from other threads that are still busy (by removing tasks from the tail of their deque).


## RecursiveTask
```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;
    
class SumTask extends RecursiveTask<Long> {
    final int seqThreshold = 500;
    int[] data;
    int start, end;
  
    SumTask(int[] data, int start, int end)
    {
        this.data = data;
        this.start = start;
        this.end = end;
    }
  
    @Override
    protected Long compute()
    {
        long sum = 0;
        if ((end - start) < seqThreshold) {
            for (int i = start; i < end; i++)
                sum += data[i];
        }
        else {
            int middle = (start + end) / 2;
  
            SumTask subtaskA = new SumTask(data, start, middle);
            SumTask subtaskB = new SumTask(data, middle, end);
  
            subtaskA.fork();
            subtaskB.fork();
  
            sum += subtaskA.join() + subtaskB.join();
        }
        return sum;
    }
}

public class Main {
    public static void main(String[] args)
    {
        ForkJoinPool forkJoinPool = new ForkJoinPool();
  
        int[] nums = new int[4000];
        for (int i = 0; i < nums.length; i++) {
            nums[i] = i;
        }
        SumTask task = new SumTask(nums, 0, nums.length);
        long summation = forkJoinPool.invoke(task);
        System.out.println("Summation " + summation);
    }
}
```

### Output :
```
Summation 7998000
```

## RecursiveAction :
```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveAction;
  
// A task that transforms the elements into their square roots
class SqrtTransform extends RecursiveAction {
    final int seqThreshold = 1000;
  
    double[] data;
  
    // Determines what part of data to process
    int start, end;
  
    SqrtTransform(double[] data, int start, int end)
    {
        this.data = data;
        this.start = start;
        this.end = end;
    }
  
    // The method where parallel computation will occur
    @Override
    protected void compute()
    {
        // If the number of elements are less
        // than the sequential threshold
        if ((end - start) < seqThreshold) {
            for (int i = start; i < end; i++) {
                data[i] = Math.sqrt(data[i]);
            }
        }
        else {
            // Otherwise, continue to break the data into smaller pieces
            // Find the midpoint
            int middle = (start + end) / 2;
  
            // Invoke new tasks, using the subdivided tasks.
            invokeAll(new SqrtTransform(data, start, middle),
                      new SqrtTransform(data, middle, end));
        }
    }
}

public class Main {
    public static void main(String[] args)
    {
        // Create a pool of threads.
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        double[] nums = new double[100000];
  
        // Give nums some values
        for (int i = 0; i < nums.length; i++) {
            nums[i] = (double)i;
        }
        System.out.println("A portion of the original sequence");
        for (int i = 0; i < 9; i++) {
            System.out.print(nums[i] + " ");
        }
        System.out.println();
        SqrtTransform task
            = new SqrtTransform(nums, 0, nums.length);
  
        // Start the task
        forkJoinPool.invoke(task);
        System.out.println("A portion of the transformed sequence"
                           + " (to four decimal places): ");
        for (int i = 0; i < 9; i++) {
            System.out.printf("%.4f ", nums[i]);
        }
        System.out.println();
    }
}
  
```

## Output :
```
A portion of the original sequence
0.0 1.0 2.0 3.0 4.0 5.0 6.0 7.0 8.0 
A portion of the transformed sequence (to four decimal places): 
0.0000 1.0000 1.4142 1.7321 2.0000 2.2361 2.4495 2.6458 2.8284 
```

# References :
1. https://www.pluralsight.com/guides/introduction-to-the-fork-join-framework

2. https://www.geeksforgeeks.org/java-util-concurrent-recursivetask-class-in-java-with-examples

3. https://www.geeksforgeeks.org/java-util-concurrent-recursiveaction-class-in-java-with-examples
