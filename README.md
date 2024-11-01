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

class MyRecursiveAction extends RecursiveAction {
    private long workLoad = 0;
    public MyRecursiveAction(long workLoad) {
        this.workLoad = workLoad;
    }

    @Override
    protected void compute() {
        // if work is above threshold, break tasks up into smaller tasks
        if (this.workLoad > 16) {
            System.out.println("Splitting workLoad : " + this.workLoad);
            long workload1 = this.workLoad / 2;
            long workload2 = this.workLoad - workload1;
            MyRecursiveAction subtask1 = new MyRecursiveAction(workload1);
            MyRecursiveAction subtask2 = new MyRecursiveAction(workload2);
            subtask1.fork();
            subtask2.fork();
        } else {
            System.out.println("Doing workLoad myself: " + this.workLoad);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        ForkJoinPool forkJoinPool = new ForkJoinPool(8);
        MyRecursiveAction myRecursiveAction = new MyRecursiveAction(123);
        forkJoinPool.invoke(myRecursiveAction);
        try {
            Thread.sleep(5000);
        } catch (InterruptedException exception) {
            System.out.println("Main thread interrupted");
        }
        System.out.println("Main method execution finished");
    }
}
```

## Output :
```
Splitting workLoad : 123
Splitting workLoad : 62
Splitting workLoad : 61
Splitting workLoad : 31
Splitting workLoad : 31
Doing workLoad myself: 16
Doing workLoad myself: 15
Splitting workLoad : 30
Doing workLoad myself: 15
Doing workLoad myself: 15
Doing workLoad myself: 16
Splitting workLoad : 31
Doing workLoad myself: 15
Doing workLoad myself: 15
Doing workLoad myself: 16
Main method execution finished
```

# References :
1. https://www.pluralsight.com/guides/introduction-to-the-fork-join-framework

2. https://www.youtube.com/watch?v=aiwuJQt7YJU

3. https://github.com/jjenkov/java-examples/blob/main/src/main/java/com/jenkov/java/concurrency/forkjoinpool/JavaForkJoinPoolExample.java
