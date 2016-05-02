#Modifying _gem5_ Cache Source Code
##Objective
Add new cache replacement policy to _gem5_ by modifying its source code.

##Reference
- [*gem5* Memory System](http://www.gem5.org/docs/html/gem5MemorySystem.html)

##Recommended Utilities
- vim
- ctags
- cscope
```shell
$ sudo apt-get install -y vim ctags cscope
$ cp .vimrc ~/
$ cp -R .vim ~/
```

##Modify These Files
- `src/mem/cache/Cache.py`
```Python
...

#tags = Param.BaseTags(LRU(), "Tag store (replacement policy)")
tags = Param.BaseTags(MyRepl(), "Tag store (replacement policy)")

...
```

- `src/mem/cache/base.cc`
```C++
...

//#include "mem/cache/tags/lru.hh"
#include "mem/cache/tags/my_repl.hh"

...
```

- `src/mem/cache/tags/Tags.py`
```Python
...

#class LRU(BaseSetAssoc):
#    type = 'LRU'
#    cxx_class = 'LRU'
#    cxx_header = "mem/cache/tags/lru.hh"

class MyRepl(BaseSetAssoc):
    type = 'MyRepl'
    cxx_class = 'MyRepl'
    cxx_header = "mem/cache/tags/my_repl.hh"

...
```

- `src/mem/cache/tags/base.hh`
```C++
...

class BaseTags : public ClockedObject
{
  protected:
    /** The block size of the cache. */
    const unsigned blkSize;

    ...

    // Statistics
    /**
     * @addtogroup CacheStatistics
     * @{
     */

    Stats::Scalar myStat;

    ...
```

- `src/mem/cache/tags/base.cc`
```C++
...

#include<typeinfo>

...

void
BaseTags::regStats()
{
    using namespace Stats;

    myStat
        .name(name() + ".myCustomStat")
        .desc(typeid(*this).name())
        ;

...
```

- `src/mem/cache/tags/SConscript`
```Python
...

#Source('lru.cc')
Source('my_repl.cc')

...
```

##Create These Files
- `src/mem/cache/tags/my_repl.hh`
```C++
//#ifndef __MEM_CACHE_TAGS_LRU_HH__
//#define __MEM_CACHE_TAGS_LRU_HH__
#ifndef __MEM_CACHE_TAGS_MYREPL_HH__
#define __MEM_CACHE_TAGS_MYREPL_HH__

//#include "params/LRU.hh"
#include "params/MyRepl.hh"

//class LRU : public BaseSetAssoc
class MyRepl : public BaseSetAssoc
{
    public:
        /** Convenience typedef. */
        //typedef LRUParams Params;
        typedef MyReplParams Params;
 
        /**
         * Construct and initialize this tag store.
         */
        //LRU(const Params *p);
        MyRepl(const Params *p);

    
        /**
         * Destructor
         */
        //~LRU() {}
        ~MyRepl() {}

        ...
}

//#endif // __MEM_CACHE_TAGS_LRU_HH__
#endif // __MEM_CACHE_TAGS_MYREPL_HH__
```

- `src/mem/cache/tags/my_repl.cc`
```C++
...

//#include "mem/cache/tags/lru.hh
#include "mem/cache/tags/my_repl.hh

...

MyRepl*
MyReplParams::create()
{
    return new MyRepl(this);
}

...
```

##Rebuild
```shell
$ scons build/ARM/gem5.opt
```
