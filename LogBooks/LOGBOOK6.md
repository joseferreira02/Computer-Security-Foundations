# Logbook 6 - Format String

## Task 1 - Crashing the program

In this introductory task, we were just supposed crash a program on Docker. In the beggining, we started the teo containers using the fiven docker-compose.yml file an set a message to the server using the command "echo hello | nc 10.9.0.5 9090", receiving the following printed messages.

![Logbook 6 — Testing](/images/logbook6/task1/task1_print1.png)

After that, we created a badfile using "build_string.py" and provided it as an input to the server, crashing the server.

![Logbook 6 — Creating the badfile and providing as an input](/images/logbook6/task1/task1_badfile.png)

![Logbook 6 — Crash](/images/logbook6/task1/task1_print2.png)

## Task 2 - Printing Out the Server Program’s Memory

### Task 2.A: Stack Data
In this task we exploit a format-string vulnerability to locate the buffer’s local variable on the stack. The approach is simple: place a recognizable marker in the buffer, then print successive stack values until the marker appears.

***Why this works:***

A `printf` call expects a format string and matching arguments. Here, the program uses a user-controlled format string, so we control how `printf` interprets the stack. That lets `printf` read arbitrary stack words as if they were arguments.

If the format string contains `%x`, `printf` will read the next value from the stack and print it in hexadecimal. By printing many values, we eventually see our marker — in this lab, the character `A` (`0x41`). When `0x41` appears in the output, we know we’ve reached the buffer’s local variable on the stack, which we did:

![Logbook 6 — Locating buffer on stack](/images/logbook6/task2/task2A/2.1Stack.png)
Figure 2A1 — Location of the local variable

As we can see, we were able to find the `AAAA` (`0x41414141`) right at the end of our stack dump.

The payload we built was fairly simple. We used Python to write to the console the start of the buffer: `AAAA` + `%x` * 64, where 64 was found through trial and error.

![Logbook 6 — Locating buffer on stack](/images/logbook6/task2/task2A/2.1stackpayload.png)
Figure 2A2 — Payload used for Task 2A

### Task 2.B: Heap Data

We follow the same idea as in task 2A, with a few adjustments.

The goal here is to use the format string vulnerability to read an undisclosed message (the “secret message”).

**How do we achieve this?**  
Instead of placing a marker value in the buffer, we place an address. Then we take advantage of the `%s` format specifier, which treats the value it reads as a pointer to a string. If we write the address of the secret message (`0x080b4008`) at the beginning of our buffer, we know that this address will appear as the 64th value on the stack relative to the `printf` call.

So, we modify the payload from task 2A by writing the secret message’s address in little-endian format. At first, we only see the address itself being printed as hex values (`%x`), not the contents stored at that address. To fix this, we replace the final `%x` (the 64th read) with `%s`. This tells `printf` to interpret that stack value as a pointer to a string, allowing us to print the actual secret message.

With this change, we successfully retrieve the hidden string and complete the task.

![Logbook 6 — Result](/images/logbook6/task2/task2B/2.1heap.png)
Figure 2B1 — Attack results

![Logbook 6 — PAYLOAD](/images/logbook6/task2/task2B/2.1heappayload.png)
Figure 2B2 — payload used


## Task 3 - Modifying the Server Program’s Memory

### Task 3.A: Change the value to a different value

This lab follows the same idea as task 2B, but with a different address (`0x080E5068`) and a twist: instead of reading from the address, we want to write to it.

This can be achieved using `%n`, because its purpose is to store the number of characters written so far by the `printf` function into an integer variable pointed to by its argument.

Following this idea, we change the target address to the one we want to write to, and then use `%n` instead of `%s` in the format string.Doing so allows us to successfuly change the desired value in memory.

![Logbook 6 — Result](/images/logbook6/task3/task3A/3Aresult.png)
Figure 3A1 — Attack results

![Logbook 6 — Payload](/images/logbook6/task3/task3A/3Apayload.png)
Figure 3A2 — payload used

### Task 3.B: Change the value to 0x5000

in this final task we are asked to change the target to our specific value: `0x5000`.

In theory its easy because we just need to calculate what we have right now in our payload in hex - wanted result which is 20244 in decimal so we need that same number of bytes to change it.
We were just faced with a small problem in the pratical side of this exercise: we couldnt add chars without messing with its size or local variable location.

-1º we tried the approach of just adding 'A'* 20244 which gave us the desired value but when reading the next value it would read the buffer's A so we would need to loop through all the 'A's to reach the local variable which seemed possible but tedious and heavy.

-2º we tough maybe we could just read 20244 values in the stack which wasnt exacly a great idea since we would pass the stack desired address by far.

Final solution : %Nx where N is an integer number.
The was this works is just that when we in printf write %Nx we are alocating N bytes to , in this case beacuse of %x, only read the next value in the stack. this way we could inject the bytes we needed without messing with our buffer or the size to hit the local variable making us able to defeat this task 

In this final task, we are asked to change the target value to a specific one: `0x5000`.

In theory, this is simple. We just need to calculate the difference between the number of characters currently written by our payload and the desired value (`0x5000`, which is 20480 in decimal). We then need to ensure that `printf` writes exactly that number of characters before reaching the `%n`.

However, we encountered a practical problem: we couldn’t simply add characters without affecting the size of the buffer or shifting the position of the local variable on the stack.

**1º Attempt:**  
We first tried adding `'A' * 20244`. This technically worked in terms of producing the correct number of bytes, but it created a new problem: when reading the next value, `printf` would read all the `'A'`s from the buffer. That meant we would have to loop through tens of thousands of characters before reaching the local variable—possible, tedious and inefficient.

**2º Attempt:**  
We then considered printing 20244, values from the stack directly, but this was not viable because we would move far past the target address.

**Final Solution: using `%Nx`**  
The correct approach was to use `%Nx`, where **N is an integer**. When used in `printf`, `%Nx` prints the next value on the stack but pads the output to **N total characters**. This allows us to inject the required number of bytes *without* adding actual characters to the buffer or shifting its layout.

Using this method, we were able to produce the exact number of printed bytes needed to reach `0x5000` and successfully complete the task.

![Logbook 6 — Result](/images/logbook6/task3/task3B/3bresult.png)
Figure 3B1 — Attack results

![Logbook 6 — Payload](/images/logbook6/task3/task3B/3bpayload.png)
Figure 3B2 — payload used

## Exercise 2 From Moodle:

***Does the format string need to be in the stack to cause damages?***

No, the format string does not need to be on the stack. It can come from anywhere, including the heap, and the vulnerability still works. What matters is that `printf` interprets the attacker-controlled string, and then uses the **stack** to fetch its arguments. Because of that, `%x` will always leak values from the stack, regardless of where the format string or the input buffer is stored.

The important difference is *what memory is reachable*.  
In the original tasks, the buffer was on the stack, so scanning with `%x` eventually printed the marker we had placed there. If the vulnerable buffer were instead allocated on the heap, `%x` would not print its contents, because it only reads stack arguments and does not automatically access heap memory. Unless a stack variable contains a pointer into the heap buffer, our marker would never appear.

With this in mind, task 2A would be impossible if the buffer lived on the heap, because we would have no way to print its local variable by simply walking the stack.

