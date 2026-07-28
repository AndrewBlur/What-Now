this library allows us to inspect and manage operating system processes

## Deleting all children and parent

*parents and children are psutil processes 

> Getting all descendants of a process
```python
children = parent.children(recursive=True)
```

we can simply iterate through children to access each process

`process.terminate()`
 this politely asks the process to clean up 

after terminate we can use 
`_,alive = psutil.wait_procs(children,timeout=n) `
we an wait for certain timeout and see rest of the alive processes in alive

we can now force kill them 
using 
`process.kill()`

we need to wait again to ensure the force-killed children have actually exited 
`psutil.wait_procs(alive,timeout=n)`

then we terminate the parent 
`parent.terminate()`

then we can check if the process terminated by waiting
`parent.wait(timeout=5)`
if `psutil.TimeoutExpired` is raised 
force kill the parent
`parent.kill()`
then wait again 
`parent.wait(timeout=5)`

then `.join()` in multiprocessing library 
waits for the multiprocessing.process's whole process tree to complete before exiting the function


