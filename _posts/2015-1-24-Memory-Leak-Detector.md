---
layout: page
title: Memory Leak Detector
category: post
description: How I developed a memory leak detector for my game engine
---

## Initial Findings and Issues

Whilst developing my Atom game engine, I realised I needed to create a memory leak detector. Creating debugging tools is always helpful especially for complex systems, like a game engine.

I initially started by looking at the CRT Debugging tools for Windows. However, the memory dumps did not provided sufficient information for me. Next I tried using the CRT hooking function so whenever dynamic memory was allocated my callback function would be invoked. This ended in failure as getting the filename and line failed to work.

My next step was to create my own memory tracker, by implementing a static MemoryManager class which would be used instead of the __new__ and __delete__ operators. This worked really well, as I used macros to shorten the function calls, and I could use the __\_\_FILE\_\___ and __\_\_LINE\_\___ macros to get the file and line which the memory allocation occured on. Unfortunately, I did not like the way I had to replace the __new__ and __delete__ keywords with macros.

## The solution

I decided to then learn how to overload the __new__ and __delete__ global operators so they were invoked instead, and I could track all dynamic memory allocations used by my engine.

{% highlight c++ %}
extern void* operator new(size_t size, size_t line, const char* filename) throw(std::bad_alloc);
{% endhighlight %}

Here is the function declaration for the global new operator. I simply add a line number and filename parameter. However, now I would need to pass in the __\_\_LINE\_\___ and __\_\_FILE\_\___ macros everytime I allocated memory. This is solved by using a macro to replace new with a placement new.

{% highlight c++ %}
#   define new  new(__LINE__, __FILE__)
{% endhighlight %}

This macro replaces any __new__ found in my engine and inserts the line and filename parameters needed to invoke my overloaded new operator.

{% highlight c++ %}
extern void operator delete(void* ptr) noexcept;
{% endhighlight %}

Next up is to overload the global delete operator. It only takes a single pointer to the memory address you are freeing. There is no need to pass in the filename or line number since that is stored in a struct when we allocate it.

In the source file (.cpp) is where you implement these overloaded functions. First off I create two variables; the first one is an __unsigned long long__ to keep track of the total amount of dynamically allocated memory I have, and the second is a linked list of structs which contain the allocation data.

{% highlight c++ %}
unsigned long long gTotalAllocatedMemory = 0ULL;
{% endhighlight %}

Make sure to initialise it to zero, otherwise it will produce undefined behaviour. I chose an __unsigned long long__ since a game using my engine which requires over 4GB of memory would overflow a 32bit integer.

{% highlight c++ %}
struct AllocHeader
{
    void* ptr;
    size_t size;
    size_t line;
    const char* filename;
};

std::list<AllocHeader> gMemoryAllocations;
{% endhighlight %}

Next is the linked list of allocation data. It stores a pointer to the allocated data; this is used for when it comes to deleting the pointer, I can find the allocation struct in the list.
The other data I store includes the size; which is how many bytes the allocation used, the line number which the allocation occured on, and the filename to track where you allocated the memory and so you can identify where you failed to delete it.

I do have getters for both of these variables, so at anytime whilst the program is running I can check how much memory is allocated. Additionally, at the end of the program, I get the list and iterate through it and print out all the memory leaks so I can fix them.

{% highlight c++ %}
void* AtomAlloc(size_t size, size_t line, const char* filename) throw(std::bad_alloc)
{
    // Allocate the memory
    void* ptr = malloc(size);

    if (ptr == nullptr)
    {
        throw std::bad_alloc();
    }

    // Update the total memory allocation
    gTotalAllocatedMemory += size;

    // Create an allocation header
    AllocHeader header =
    {
        ptr,
        size,
        line,
        filename
    };
    gMemoryAllocations.push_back(header);

    // Return the successful memory
    return ptr;
}
{% endhighlight %}

I created an __AtomAlloc()__ function since both the __new__ and 'new[]' operators do exactly the same thing, so it reduces the code size.
Since I am overloading the new operator, I have to use malloc() to allocate the memory.
I then make sure the memory was successfully allocated and throw a bad allocation exception if not.
Next I update the total amount of memory allocated and created an allocation struct to store all the allocation data.

{% highlight c++ %}
void AtomDelete(void* ptr) noexcept
{
    // Find the pointer in our total memory allocations
    for (auto iter = gMemoryAllocations.cbegin(); iter != gMemoryAllocations.cend(); ++iter)
    {
        if (iter->ptr == ptr)
        {
            // Update the total memory allocation
            gTotalAllocatedMemory -= iter->size;

            // Erase the memory allocation from the list
            gMemoryAllocations.erase(iter);

            // Free the memory
            free(ptr);

            // Return early
            return;
        }
    }
}
{% endhighlight %}

Again, I created an __AtomDelete()__ function since both __delete__ and 'delete[]' do the same thing. I could have used an unordered_map to store the allocation data to improve the deletion speed but that would require a lot more memory. This means I have to iterate through the list everytime I want to delete memory. I could implement a heuristic and iterate from the back of the list, since any initial allocated memory would in general last the longest in memory and anything at the end would likely be deleted sooner.

## Summary
Overall, I really do like this memory tracking system; it allows you to keep tabs on all memory allocations and still be able to use the C++ operators. Of course, when it comes to release mode you would put a guard around the include of the header file, since this is obviously slower than normal memory allocation. Unfortunately, the downside to this system is that it can not track stack-based memory, however, you could probably get it by subtracting the total allocated memory which we store from the total memory usage of the application.
