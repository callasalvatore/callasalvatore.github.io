The impact of the code on memory performances
---

As Software engineers, we must always consider the impact of the code we write on the memory performance.
It does not matter how complex the code is; if we don't pay due attention, the results could be unpredictable.
Let's consider the following algorithm in which we have two different ways of traversing an array: by row and by column.

```
using System;
using System.Diagnostics;

class Program
{
    static void Main(string[] args)
    {
        TraverseArrayByRow();
        TraverseArrayByColumn();
    }


    private static void TraverseArrayByRow()
    {
        const int N = 10000;
        int[,] arr = new int[N, N];
        Random rand = new Random();

        // Fill array with random integers between 0 and 999
        for (int i = 0; i < N; i++)
        {
            for (int j = 0; j < N; j++)
            {
                arr[i, j] = rand.Next(1000);
            }
        }

        Stopwatch stopwatch = new Stopwatch();

        stopwatch.Start();
        for (int i = 0; i < N; i++)
        {
            for (int j = 0; j < N; j++)
            {
                int value = arr[i, j];
            }
        }

        stopwatch.Stop();
        Console.WriteLine("Traversing by row took: " + stopwatch.ElapsedMilliseconds + " ms");
    }
    
    private static void TraverseArrayByColumn()
    {
        const int N = 10000;
        int[,] arr = new int[N, N];
        Random rand = new Random();

        // Fill array with random integers between 0 and 999
        for (int i = 0; i < N; i++)
        {
            for (int j = 0; j < N; j++)
            {
                arr[i, j] = rand.Next(1000);
            }
        }

        Stopwatch stopwatch = new Stopwatch();

        stopwatch.Start();
        for (int j = 0; j < N; j++)
        {
            for (int i = 0; i < N; i++)
            {
                int value = arr[i, j];
            }
        }

        stopwatch.Stop();
        Console.WriteLine("Traversing by column took: " + stopwatch.ElapsedMilliseconds + " ms");
    }
}
```

Executing this in my notebook generates the following output:

```
Traversing by row took: 417 ms
Traversing by column took: 1202 ms
```

To gain insight into the reasons behind the varying results, it is worth delving into the inner workings of DRAM.

<a title="Glogger at English Wikipedia, CC BY-SA 3.0 &lt;http://creativecommons.org/licenses/by-sa/3.0/&gt;, via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:Square_array_of_mosfet_cells_read.png"><img width="512" alt="Square array of mosfet cells read" src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3d/Square_array_of_mosfet_cells_read.png/512px-Square_array_of_mosfet_cells_read.png"></a>

`By Glogger at English Wikipedia - Transferred from en.wikipedia to Commons., CC BY-SA 3.0, https://commons.wikimedia.org/w/index.php?curid=5549293`


DRAM is arranged in a cell array, with each cell comprising a transistor and a capacitor that stores one bit. 
To access a specific cell in a DRAM array, an address is applied to the row and column lines that correspond to the desired cell. This activates the row and column transistors, allowing access to the cell. However, since the row and column lines are shared among multiple cells, accessing a single cell requires a two-phase process. In the first phase, the desired row is selected by applying a voltage to the row line. In the second phase, the desired column is selected by applying a voltage to the column line. In general, reading a single bit from DRAM involves many steps and for this reason, is subject to various delays and timing constraints that can affect the overall performance of the memory:

<ul>
    <li> <strong>tCL</strong> (CAS latency): the number of clock cycles that elapse between the memory controller requesting data from the DRAM and the time when that data is   available.</li>
    <li> <strong>tRCD</strong> (RAS to CAS delay): is a timing parameter that represents the time interval between the moment when a memory controller activates a specific row in DRAM and the moment when the same controller sends a column address to the DRAM, specifying a location within that row to access data.</li>
    <li> <strong>tRP</strong> (Row Precharge): is the delay that occurs after a row has been accessed and read or written to, and before the memory controller can access a different row.</li>
    <li> <strong>tRAS</strong> (Row Active Time): defines the minimum time required for the memory controller to access data from the DRAM array. Longer tRAS values mean that the memory controller has to wait longer before it can activate a new row, resulting in slower access times and reduced performance.</li>
</ul>

Now is possible to understand that:

<ul>
   <li> To access the data in a different column it will take <strong>tCL</strong> clock cycles. </li>
   <li> To access the data in a different row, we need to wait: <strong>tRP</strong> cycles for the row to be recharged and then <strong>tCL</strong> plus <strong>tRCD</strong> cycles. </li>
</ul>

After discussing the key factors that affect memory read performance, we can now understand why the previous two algorithms produce different results. 
 If the loop accesses memory in a sequential pattern (i.e., accessing memory locations that are adjacent to each other), it can take advantage of the DRAM's ability to prefetch data and improve the overall performance of the loop. On the other hand, if the loop accesses memory in a random pattern (i.e., accessing memory locations that are not adjacent to each other), it can cause the DRAM to perform more read operations, which can increase the overall latency of the loop.


#### <strong>References</strong>

Konrad Kokosa (2018). Pro .Net Memory Management. Apress
