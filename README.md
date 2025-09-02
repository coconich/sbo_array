# Small Buffer Optimized Array

The public API acts as a drop in std::vector replacement where you will see performance compareable to std::vector, but with a boost in speed (in exchange for memory usage) when container size stays below size_threshold.

The size_threshold template parameter can be used to tune how often you stay on the stack or need to go to heap.
```c
template <typename T, size_t size_threshold = 64>
class SboArray
```

The underlying data relies on a union of a stack buffer and a heap pointer. It transitions between usage as necessary, and currently uses malloc and free for allocating memory.
```c
union Storage 
{
    char stack_buffer[size_threshold * sizeof(T)] = {0};
    T* heap_ptr;
};
Storage storage_;
size_t count_ = 0;
size_t capacity_ = size_threshold;
bool using_heap_ = false;
T* cached_data_ptr_ = nullptr;
```


Because the container uses internal stack storage until it needs more, it will not allocate upon construction

  - *use reserve to force the allocation or push more than size_threshold items*

There are some optimizations specifically when used with plain old data.
```c
static constexpr bool plain_old_data_ = std::is_trivially_copyable_v<T> &&
                                        std::is_trivially_default_constructible_v<T> &&
                                        std::is_trivially_destructible_v<T> &&
                                        std::is_trivially_move_constructible_v<T>;
```
   

I mainly use it for gameplay code where i need most of the time small containers of id's and other pod's 
I also use it as the base container for a PriorityQueue 
     this can make pathfinding for shorter paths/smaller graphs almost always on the stack


Example: this code is "slideware" (not real code)
```c
SboArray entities_to_process;
for (const u32 e : game.entities)
{
    // we need to know now that this trait is active here
    // the code in the loop may invalidate the flags, but we still need to process that state this frame
    if(EntitySystem::HasFlag(e, SOME_ENTITY_FLAG & SOME_OTHER_ENTITY_FLAG)) { entities_to_process.push_back(e); }

    // ... a bunch of gameplay code that needs to get done before processing the entities
    // ...
}

for(auto e : entities_to_process)
{
    // now do the thing to those entities    
    EntitySystem::DoTheThing(e);
}
```

