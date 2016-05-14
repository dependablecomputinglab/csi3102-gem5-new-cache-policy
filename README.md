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

##How to copy source code in this repository
```shell
$ git clone csi3102-gem5-new-cache-policy 
$ cp -rf csi3102-gem5-new-cache-policy/* gem5/
```

##Example: Implementing FIFO in _gem5_
###1. Modify These Files
- `src/mem/cache/Cache.py`
```Python
...

#tags = Param.BaseTags(LRU(), "Tag store (replacement policy)")
tags = Param.BaseTags(Fifo(), "Tag store (replacement policy)")

...
```

- `src/mem/cache/base.cc`
```C++
...

#include "mem/cache/tags/lru.hh"
#include "mem/cache/tags/fifo.hh"

...
```

- `src/mem/cache/tags/Tags.py`
```Python
...

class LRU(BaseSetAssoc):
    type = 'LRU'
    cxx_class = 'LRU'
    cxx_header = "mem/cache/tags/lru.hh"

class Fifo(BaseSetAssoc):
    type = 'Fifo'
    cxx_class = 'Fifo'
    cxx_header = "mem/cache/tags/fifo.hh"

...
```

- `src/mem/cache/tags/SConscript`
```Python
...

Source('lru.cc')
Source('fifo.cc')

...
```

###2. Create These Files
FIFO is similar to LRU. <br />
___TIP___ __Just focus on difference in the function accessBlock()__

- `src/mem/cache/tags/fifo.hh` will look like:
```C++
//#ifndef __MEM_CACHE_TAGS_LRU_HH__
//#define __MEM_CACHE_TAGS_LRU_HH__
#ifndef __MEM_CACHE_TAGS_FIFO_HH__
#define __MEM_CACHE_TAGS_FIFO_HH__

//#include "params/LRU.hh"
#include "params/Fifo.hh"

//class LRU : public BaseSetAssoc
class Fifo : public BaseSetAssoc
{
    public:
        /** Convenience typedef. */
        //typedef LRUParams Params;
        typedef FifoParams Params;
 
        /**
         * Construct and initialize this tag store.
         */
        //LRU(const Params *p);
        Fifo(const Params *p);

    
        /**
         * Destructor
         */
        //~LRU() {}
        ~Fifo() {}

        ...
}

//#endif // __MEM_CACHE_TAGS_LRU_HH__
#endif // __MEM_CACHE_TAGS_FIFO_HH__
```

- `src/mem/cache/tags/fifo.cc`
```C++
...

//#include "mem/cache/tags/lru.hh
#include "mem/cache/tags/fifo.hh

...

Fifo*
FifoParams::create()
{
    return new Fifo(this);
}

...
```

###3. Rebuild
```shell
$ scons build/ARM/gem5.opt
```

##Insert new statistic in _gem5_
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

If you open the file '_stats.txt_', you can see new stat 'myCustomStat' is inserted.
