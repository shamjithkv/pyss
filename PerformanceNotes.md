Don't forget the [rules of optimization](http://c2.com/cgi/wiki?RulesOfOptimization) :)

These optimizations were all driven by profiler runs.

# Current Performance #

To measure the run time yourself, try '`./time_simulator < input`' in the '`prototype01/`' subdirectory.

These measurements were done on a 550Mhz Pentium III running Linux.

  * Input: first 100,000 jobs from sanitized LANL-CM5-1994-3.1-cln.swf.gz
  * Run time: 40.7 seconds
  * Jobs per second: 2457
  * Code revision: [r233](https://code.google.com/p/pyss/source/detail?r=233) (28 April, 2007)

# Optimizations Done #
  * Running with '`python -O`' (which [disables assertions](http://docs.python.org/ref/assert.html))
    * One assert was especially painful: verifying the event hasn't been entered before in `EventQueue.add_event`, it took almost %80 of the run time
    * Also, replacing 'if...: raise ...' with straight assertions in bottlenecks, so '`python -O`' will disable them too
  * Using a heap in `EventQueue` instead of a sorted list
  * Entering (timestamp, event) tuples into the heap to reduce calls to `JobEvent.__cmp__`
  * Input parsing:
    * ~~Parsing input fields lazily in `workload_parser.py`, only a few fields from the SWF format are used~~
    * Hard-coded (and hard to work with) parsing of input lines, saves a lot of time (function calls + logic) ([r233](https://code.google.com/p/pyss/source/detail?r=233))

# Possible Optimizations #
  * Currently all the job submission events are entered into the queue in `Simulator.__init__`, which causes every event queue operation to be slow (many events in the queue). The job submission events are already sorted - we can enter them as needed, when the queue reaches a relevant timestamp, and keep it small this way.
  * In `EventQueue`, use a heap with better performance on partially-sorted data (like our events)
    * Such as Raymond Hettinger's [Two-pass Pairing Heap with Multipass Auxiliary List](http://aspn.activestate.com/ASPN/Cookbook/Python/Recipe/277553)
    * Choose when to insert maintaining heap (`O(log(heap_size)) * no. of pushes`) and when to append several items and then heapify (`O(heap_size) + no. of pushes`)

# Failed Optimizations #
  * Use '`__slots__`' on classes with numerous instances
    * Timing has shown a meager %3 run time reduction with `__slots__` on classes `Job` and `JobEvent*`

# TODO #
  * We may be interested in keeping most of the asserts even in a production run, how can we do this easily?

# Profiling Runs #
## [r233](https://code.google.com/p/pyss/source/detail?r=233) (quick and dirty parsing of input) ##
```
         3600555 function calls (3600553 primitive calls) in 60.963 CPU seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.002    0.002   60.963   60.963 <string>:1(?)
        1    0.066    0.066   60.962   60.962 run_simulator.py:2(?)
        1    0.000    0.000   60.790   60.790 run_simulator.py:13(main)
        1    4.826    4.826   41.947   41.947 prototype.py:130(run)
   300000    6.216    0.000   30.617    0.000 event_queue.py:48(advance)
        1    3.404    3.404   18.843   18.843 prototype.py:110(__init__)
   100001    9.044    0.000   10.607    0.000 prototype.py:83(parse_job_lines_quick_and_dirty)
   300000    2.256    0.000    9.332    0.000 event_queue.py:37(pop)
   100000    0.853    0.000    7.231    0.000 prototype.py:71(_start_job_handler)
   300000    2.635    0.000    7.115    0.000 event_queue.py:9(add_event)
   300000    7.070    0.000    7.075    0.000 simple_heap.py:9(pop)
   300001    2.784    0.000    6.504    0.000 event_queue.py:30(empty)
   100000    3.009    0.000    6.378    0.000 prototype.py:62(add_job)
   100000    3.161    0.000    5.514    0.000 prototype.py:41(job_submitted)
   300000    4.474    0.000    4.480    0.000 simple_heap.py:6(push)
   300001    2.586    0.000    3.721    0.000 event_queue.py:34(__len__)
   300000    3.438    0.000    3.438    0.000 prototype.py:4(__init__)
   300000    1.659    0.000    1.659    0.000 event_queue.py:42(_get_event_handlers)
   100000    1.563    0.000    1.563    0.000 prototype.py:20(__init__)
   300001    1.135    0.000    1.135    0.000 simple_heap.py:17(__len__)
   100000    0.665    0.000    0.665    0.000 prototype.py:67(_remove_job_handler)
        1    0.027    0.027    0.101    0.101 event_queue.py:1(?)
        1    0.038    0.038    0.074    0.074 simple_heap.py:1(?)
        1    0.035    0.035    0.036    0.036 heapq.py:31(?)
      522    0.011    0.000    0.011    0.000 prototype.py:11(__cmp__)
      2/1    0.003    0.002    0.003    0.003 workload_parser.py:10(?)
        1    0.002    0.002    0.002    0.002 bisect.py:1(?)
      2/1    0.001    0.001    0.001    0.001 prototype.py:3(?)
        1    0.000    0.000    0.000    0.000 prototype.py:55(__init__)
        1    0.000    0.000    0.000    0.000 prototype.py:53(Machine)
        1    0.000    0.000    0.000    0.000 event_queue.py:3(EventQueue)
        3    0.000    0.000    0.000    0.000 event_queue.py:55(add_handler)
        1    0.000    0.000    0.000    0.000 event_queue.py:4(__init__)
        1    0.000    0.000    0.000    0.000 simple_heap.py:2(Heap)
        1    0.000    0.000    0.000    0.000 prototype.py:36(__init__)
        1    0.000    0.000    0.000    0.000 simple_heap.py:3(__init__)
        1    0.000    0.000    0.000    0.000 prototype.py:109(Simulator)
        1    0.000    0.000    0.000    0.000 prototype.py:19(Job)
        1    0.000    0.000    0.000    0.000 prototype.py:34(StupidScheduler)
        1    0.000    0.000    0.000    0.000 prototype.py:16(JobStartEvent)
        1    0.000    0.000    0.000    0.000 prototype.py:17(JobEndEvent)
        1    0.000    0.000    0.000    0.000 prototype.py:15(JobSubmitEvent)
        0    0.000             0.000          profile:0(profiler)
```

## [r228](https://code.google.com/p/pyss/source/detail?r=228) (straight 'assert' calls instead of 'if ..: raise ..') ##
```
         4400594 function calls (4400592 primitive calls) in 86.007 CPU seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.002    0.002   86.007   86.007 <string>:1(?)
        1    0.372    0.372   86.005   86.005 run_simulator.py:2(?)
        1    0.000    0.000   85.602   85.602 run_simulator.py:13(main)
        1    5.239    5.239   50.029   50.029 prototype.py:110(run)
   300000    7.048    0.000   37.422    0.000 event_queue.py:48(advance)
        1    5.609    5.609   35.573   35.573 prototype.py:84(__init__)
   100001    9.684    0.000   14.336    0.000 workload_parser.py:79(parse_lines)
   300000    2.563    0.000   11.035    0.000 event_queue.py:37(pop)
   100000    5.235    0.000   10.913    0.000 prototype.py:100(_job_input_to_job)
   100000    1.355    0.000    9.993    0.000 prototype.py:71(_start_job_handler)
   300000    3.082    0.000    8.714    0.000 event_queue.py:9(add_event)
   100000    3.552    0.000    8.638    0.000 prototype.py:62(add_job)
   300000    8.465    0.000    8.472    0.000 simple_heap.py:9(pop)
   300001    3.221    0.000    7.368    0.000 event_queue.py:30(empty)
   100000    3.541    0.000    6.332    0.000 prototype.py:41(job_submitted)
   300000    5.603    0.000    5.632    0.000 simple_heap.py:6(push)
   300001    2.916    0.000    4.147    0.000 event_queue.py:34(__len__)
   100000    2.954    0.000    2.954    0.000 workload_parser.py:11(__init__)
   300000    2.339    0.000    2.339    0.000 prototype.py:4(__init__)
   300000    2.254    0.000    2.254    0.000 event_queue.py:42(_get_event_handlers)
   100000    1.698    0.000    1.698    0.000 workload_parser.py:82(_should_skip)
   100000    1.579    0.000    1.579    0.000 workload_parser.py:25(run_time)
   100000    1.539    0.000    1.539    0.000 workload_parser.py:19(submit_time)
   300001    1.231    0.000    1.231    0.000 simple_heap.py:17(__len__)
   100000    1.194    0.000    1.194    0.000 prototype.py:20(__init__)
   100000    1.058    0.000    1.058    0.000 workload_parser.py:16(number)
   100000    0.994    0.000    0.995    0.000 workload_parser.py:37(num_requested_processors)
   100000    0.853    0.000    0.853    0.000 workload_parser.py:45(requested_time)
   100000    0.760    0.000    0.760    0.000 prototype.py:67(_remove_job_handler)
      522    0.036    0.000    0.036    0.000 prototype.py:11(__cmp__)
        1    0.001    0.001    0.028    0.028 event_queue.py:1(?)
        1    0.011    0.011    0.027    0.027 simple_heap.py:1(?)
        1    0.014    0.014    0.016    0.016 heapq.py:31(?)
      2/1    0.002    0.001    0.002    0.002 workload_parser.py:10(?)
        1    0.002    0.002    0.002    0.002 bisect.py:1(?)
      2/1    0.001    0.000    0.001    0.001 prototype.py:3(?)
       39    0.000    0.000    0.000    0.000 workload_parser.py:28(num_allocated_processors)
        1    0.000    0.000    0.000    0.000 prototype.py:55(__init__)
        1    0.000    0.000    0.000    0.000 event_queue.py:3(EventQueue)
        3    0.000    0.000    0.000    0.000 event_queue.py:55(add_handler)
        1    0.000    0.000    0.000    0.000 event_queue.py:4(__init__)
        1    0.000    0.000    0.000    0.000 prototype.py:53(Machine)
        1    0.000    0.000    0.000    0.000 simple_heap.py:2(Heap)
        1    0.000    0.000    0.000    0.000 prototype.py:36(__init__)
        1    0.000    0.000    0.000    0.000 simple_heap.py:3(__init__)
        1    0.000    0.000    0.000    0.000 prototype.py:83(Simulator)
        1    0.000    0.000    0.000    0.000 prototype.py:19(Job)
        1    0.000    0.000    0.000    0.000 prototype.py:34(StupidScheduler)
        1    0.000    0.000    0.000    0.000 prototype.py:16(JobStartEvent)
        1    0.000    0.000    0.000    0.000 prototype.py:17(JobEndEvent)
        1    0.000    0.000    0.000    0.000 prototype.py:15(JobSubmitEvent)
        0    0.000             0.000          profile:0(profiler)
```

## [r223](https://code.google.com/p/pyss/source/detail?r=223) ( (timestamp, event) pairs ) ##

```
         6800595 function calls (6800593 primitive calls) in 98.291 CPU seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.002    0.002   98.291   98.291 <string>:1(?)
        1    0.362    0.362   98.289   98.289 run_simulator.py:2(?)
        1    0.000    0.000   97.885   97.885 run_simulator.py:13(main)
        1    4.982    4.982   66.129   66.129 prototype.py:110(run)
   300000    8.191    0.000   54.161    0.000 event_queue.py:54(advance)
        1    5.054    5.054   31.756   31.756 prototype.py:84(__init__)
   300000    3.780    0.000   19.763    0.000 event_queue.py:43(pop)
   900001    7.516    0.000   19.019    0.000 event_queue.py:32(empty)
   600000    3.845    0.000   15.877    0.000 event_queue.py:39(_assert_not_empty)
   100001    9.122    0.000   13.018    0.000 workload_parser.py:79(parse_lines)
   900001    8.204    0.000   11.503    0.000 event_queue.py:36(__len__)
   100000    4.742    0.000    9.874    0.000 prototype.py:100(_job_input_to_job)
   100000    0.940    0.000    9.477    0.000 prototype.py:71(_start_job_handler)
   100000    3.652    0.000    8.537    0.000 prototype.py:62(add_job)
   300000    8.298    0.000    8.304    0.000 simple_heap.py:9(pop)
   300000    2.947    0.000    8.026    0.000 event_queue.py:11(add_event)
   100000    3.408    0.000    5.986    0.000 prototype.py:41(job_submitted)
   300000    5.072    0.000    5.078    0.000 simple_heap.py:6(push)
   900001    3.298    0.000    3.298    0.000 simple_heap.py:17(__len__)
   100000    2.390    0.000    2.390    0.000 workload_parser.py:11(__init__)
   300000    2.273    0.000    2.273    0.000 prototype.py:4(__init__)
   300000    1.842    0.000    1.842    0.000 event_queue.py:48(_get_event_handlers)
   100000    1.505    0.000    1.505    0.000 workload_parser.py:82(_should_skip)
   100000    1.447    0.000    1.447    0.000 workload_parser.py:25(run_time)
   100000    1.006    0.000    1.006    0.000 prototype.py:20(__init__)
   100000    1.000    0.000    1.000    0.000 workload_parser.py:16(number)
   100000    0.974    0.000    0.974    0.000 workload_parser.py:19(submit_time)
   100000    0.907    0.000    0.907    0.000 workload_parser.py:37(num_requested_processors)
   100000    0.771    0.000    0.771    0.000 workload_parser.py:45(requested_time)
   100000    0.705    0.000    0.705    0.000 prototype.py:67(_remove_job_handler)
        1    0.001    0.001    0.039    0.039 event_queue.py:1(?)
        1    0.012    0.012    0.038    0.038 simple_heap.py:1(?)
        1    0.025    0.025    0.026    0.026 heapq.py:31(?)
      522    0.012    0.000    0.012    0.000 prototype.py:11(__cmp__)
      2/1    0.002    0.001    0.002    0.002 workload_parser.py:10(?)
        1    0.001    0.001    0.001    0.001 bisect.py:1(?)
      2/1    0.001    0.000    0.001    0.001 prototype.py:3(?)
       39    0.000    0.000    0.000    0.000 workload_parser.py:28(num_allocated_processors)
        1    0.000    0.000    0.000    0.000 event_queue.py:3(EventQueue)
        1    0.000    0.000    0.000    0.000 prototype.py:55(__init__)
        3    0.000    0.000    0.000    0.000 event_queue.py:61(add_handler)
        1    0.000    0.000    0.000    0.000 event_queue.py:6(__init__)
        1    0.000    0.000    0.000    0.000 prototype.py:53(Machine)
        1    0.000    0.000    0.000    0.000 simple_heap.py:2(Heap)
        1    0.000    0.000    0.000    0.000 prototype.py:36(__init__)
        1    0.000    0.000    0.000    0.000 simple_heap.py:3(__init__)
        1    0.000    0.000    0.000    0.000 prototype.py:83(Simulator)
        1    0.000    0.000    0.000    0.000 prototype.py:19(Job)
        1    0.000    0.000    0.000    0.000 prototype.py:34(StupidScheduler)
        1    0.000    0.000    0.000    0.000 event_queue.py:4(EmptyQueue)
        1    0.000    0.000    0.000    0.000 prototype.py:15(JobSubmitEvent)
        1    0.000    0.000    0.000    0.000 prototype.py:16(JobStartEvent)
        1    0.000    0.000    0.000    0.000 prototype.py:17(JobEndEvent)
        0    0.000             0.000          profile:0(profiler)
```

## [r211](https://code.google.com/p/pyss/source/detail?r=211) (heap queue) ##
```
         13003914 function calls (13003912 primitive calls) in 220.864 CPU seconds

   Ordered by: cumulative time, internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.002    0.002  220.864  220.864 <string>:1(?)
        1    0.359    0.359  220.862  220.862 run_simulator.py:2(?)
        1    0.000    0.000  220.479  220.479 run_simulator.py:9(main)
        1    5.260    5.260  188.318  188.318 prototype.py:110(run)
   332307    8.334    0.000  178.618    0.001 event_queue.py:53(advance)
   332307   37.458    0.000  112.489    0.000 event_queue.py:43(pop)
  7133041   97.705    0.000   97.705    0.000 prototype.py:11(__cmp__)
   332307   15.340    0.000   43.311    0.000 event_queue.py:10(add_event)
   110769    0.944    0.000   42.115    0.000 prototype.py:71(_start_job_handler)
   110769    3.838    0.000   41.171    0.000 prototype.py:62(add_job)
        1    5.175    5.175   32.160   32.160 prototype.py:84(__init__)
   110770    7.295    0.000   11.456    0.000 workload_parser.py:79(parse_lines)
   996922    7.828    0.000   11.259    0.000 event_queue.py:32(empty)
   664614    3.978    0.000   10.796    0.000 event_queue.py:39(_assert_not_empty)
   110769    4.928    0.000   10.214    0.000 prototype.py:100(_job_input_to_job)
   110769    3.581    0.000    7.458    0.000 prototype.py:41(job_submitted)
   996922    3.431    0.000    3.431    0.000 event_queue.py:36(__len__)
   110769    2.554    0.000    2.554    0.000 workload_parser.py:11(__init__)
   332307    2.212    0.000    2.212    0.000 prototype.py:4(__init__)
   332307    1.933    0.000    1.933    0.000 event_queue.py:47(_get_event_handlers)
   110816    1.608    0.000    1.608    0.000 workload_parser.py:82(_should_skip)
   110769    1.426    0.000    1.426    0.000 workload_parser.py:25(run_time)
   110769    1.096    0.000    1.096    0.000 prototype.py:20(__init__)
   110769    1.005    0.000    1.005    0.000 workload_parser.py:16(number)
   110769    1.002    0.000    1.002    0.000 workload_parser.py:19(submit_time)
   110769    0.972    0.000    0.972    0.000 workload_parser.py:37(num_requested_processors)
   110769    0.790    0.000    0.790    0.000 prototype.py:67(_remove_job_handler)
   110769    0.787    0.000    0.787    0.000 workload_parser.py:45(requested_time)
        1    0.011    0.011    0.023    0.023 event_queue.py:1(?)
        1    0.010    0.010    0.011    0.011 heapq.py:31(?)
        1    0.002    0.002    0.002    0.002 bisect.py:1(?)
      2/1    0.001    0.000    0.001    0.001 prototype.py:3(?)
      2/1    0.000    0.000    0.000    0.000 workload_parser.py:10(?)
       39    0.000    0.000    0.000    0.000 workload_parser.py:28(num_allocated_processors)
        1    0.000    0.000    0.000    0.000 event_queue.py:2(EventQueue)
        1    0.000    0.000    0.000    0.000 prototype.py:55(__init__)
        3    0.000    0.000    0.000    0.000 event_queue.py:60(add_handler)
        1    0.000    0.000    0.000    0.000 prototype.py:53(Machine)
        1    0.000    0.000    0.000    0.000 prototype.py:36(__init__)
        1    0.000    0.000    0.000    0.000 event_queue.py:5(__init__)
        1    0.000    0.000    0.000    0.000 prototype.py:83(Simulator)
        1    0.000    0.000    0.000    0.000 prototype.py:34(StupidScheduler)
        1    0.000    0.000    0.000    0.000 prototype.py:19(Job)
        1    0.000    0.000    0.000    0.000 prototype.py:16(JobStartEvent)
        1    0.000    0.000    0.000    0.000 prototype.py:17(JobEndEvent)
        1    0.000    0.000    0.000    0.000 event_queue.py:3(EmptyQueue)
        1    0.000    0.000    0.000    0.000 prototype.py:15(JobSubmitEvent)
        0    0.000             0.000          profile:0(profiler)
```

## [r196](https://code.google.com/p/pyss/source/detail?r=196) (sorted list queue) ##
```
         11466754 function calls (11466752 primitive calls) in 1658.827 CPU seconds

   Ordered by: cumulative time, internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.003    0.003 1658.827 1658.827 <string>:1(?)
        1    0.405    0.405 1658.825 1658.825 run_simulator.py:2(?)
        1    0.000    0.000 1658.416 1658.416 run_simulator.py:9(main)
        1   27.256   27.256 1586.661 1586.661 prototype.py:110(run)
   332307   56.251    0.000 1540.637    0.005 event_queue.py:47(advance)
   332307 1026.374    0.003 1034.093    0.003 event_queue.py:37(pop)
   332307  290.834    0.001  388.929    0.001 event_queue.py:10(add_event)
   110769    4.688    0.000  282.340    0.003 prototype.py:71(_start_job_handler)
   110769   21.677    0.000  277.652    0.003 prototype.py:62(add_job)
   110769   26.950    0.000  141.465    0.001 prototype.py:41(job_submitted)
  5171905   98.095    0.000   98.095    0.000 prototype.py:11(__cmp__)
        1    5.979    5.979   71.754   71.754 prototype.py:84(__init__)
   996922   23.231    0.000   30.023    0.000 event_queue.py:26(empty)
   664614    6.025    0.000   17.281    0.000 event_queue.py:33(_assert_not_empty)
   110770    8.623    0.000   13.716    0.000 workload_parser.py:79(parse_lines)
   110769    2.683    0.000   12.688    0.000 prototype.py:75(free_processors)
   110769    5.450    0.000   12.088    0.000 prototype.py:100(_job_input_to_job)
   332307   11.701    0.000   11.701    0.000 event_queue.py:41(_get_event_handlers)
   110769    7.783    0.000   10.005    0.000 prototype.py:79(busy_processors)
   332307    7.697    0.000    7.697    0.000 prototype.py:4(__init__)
   996922    6.791    0.000    6.791    0.000 event_queue.py:30(__len__)
   110769    5.225    0.000    5.225    0.000 prototype.py:67(_remove_job_handler)
   110769    3.229    0.000    3.229    0.000 workload_parser.py:11(__init__)
   202439    2.222    0.000    2.222    0.000 prototype.py:81(<generator expression>)
   110816    1.864    0.000    1.864    0.000 workload_parser.py:82(_should_skip)
   110769    1.819    0.000    1.819    0.000 workload_parser.py:25(run_time)
   110769    1.647    0.000    1.647    0.000 prototype.py:20(__init__)
   110769    1.147    0.000    1.147    0.000 workload_parser.py:19(submit_time)
   110769    1.119    0.000    1.119    0.000 workload_parser.py:16(number)
   110769    1.044    0.000    1.044    0.000 workload_parser.py:37(num_requested_processors)
   110769    1.007    0.000    1.007    0.000 workload_parser.py:45(requested_time)
        1    0.001    0.001    0.002    0.002 event_queue.py:1(?)
      2/1    0.001    0.000    0.001    0.001 prototype.py:3(?)
        1    0.001    0.001    0.001    0.001 bisect.py:1(?)
        1    0.000    0.000    0.000    0.000 prototype.py:83(Simulator)
      2/1    0.000    0.000    0.000    0.000 workload_parser.py:10(?)
       39    0.000    0.000    0.000    0.000 workload_parser.py:28(num_allocated_processors)
        1    0.000    0.000    0.000    0.000 event_queue.py:2(EventQueue)
        1    0.000    0.000    0.000    0.000 prototype.py:55(__init__)
        3    0.000    0.000    0.000    0.000 event_queue.py:54(add_handler)
        1    0.000    0.000    0.000    0.000 prototype.py:53(Machine)
        1    0.000    0.000    0.000    0.000 prototype.py:36(__init__)
        1    0.000    0.000    0.000    0.000 event_queue.py:5(__init__)
        1    0.000    0.000    0.000    0.000 prototype.py:34(StupidScheduler)
        1    0.000    0.000    0.000    0.000 prototype.py:19(Job)
        1    0.000    0.000    0.000    0.000 prototype.py:17(JobEndEvent)
        1    0.000    0.000    0.000    0.000 event_queue.py:3(EmptyQueue)
        1    0.000    0.000    0.000    0.000 prototype.py:15(JobSubmitEvent)
        1    0.000    0.000    0.000    0.000 prototype.py:16(JobStartEvent)
        0    0.000             0.000          profile:0(profiler)
```