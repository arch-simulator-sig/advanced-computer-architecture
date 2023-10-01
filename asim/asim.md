# asim

overall：system consists of module with port interface

language: mainly C++

elements: modules, ports, method-call, models, benchmarks

![image](https://github.com/arch-simulator-sig/advanced-computer-architecture/assets/100679322/2042ac93-883e-4dec-9798-5f107d355de3)

Features:

1. Support hardware & non-hardware(feeder) modeling
2. independent model development
3. Support timing & stand-alone perf eval

module derivation: base Class & inheritance(asim_module_class)

e.g. generic BPU:

GetPrediction, UpdateBranchPredictor, HandleMispredictedBranch

![image](https://github.com/arch-simulator-sig/advanced-computer-architecture/assets/100679322/314cc975-d6f0-4068-9401-9ac78b058a06)

ports vs method-call: bounded vs internal interface

dependency: 

module type define interface and required internal/external module/method implicitly

info management: .awb file

keywords: name, desc, attributes, provides, requires, public, private, param

![image](https://github.com/arch-simulator-sig/advanced-computer-architecture/assets/100679322/bd3f032c-ecad-4d58-b7cb-c4a17cd84d93)

ports: message info, delay, max bandwidth

define msg between modules: id string with mseg queue

```jsx
template<class T, UINT32 N> class fifo
{
  private: 

	UINT32 head;
	UINT32 tail;
	UINT32 count;
	  
	T data[N];
```

```jsx
class BasePort : public TRACEABLE_CLASS
{
public:
  enum PortType
  { BaseType,
    WriteType,
    WritePhaseType,
    ReadType,
    ReadPhaseType,
    ConfigType,
    PeekType,        
    LastType }; // This should always be the last one!!!

  static const char *PortName[];
  static int id_count;
  int my_id;
protected:
  static asim::Vector<BasePort*> AllPorts;
protected:
  // this is used to ensure the endpoints of a connected port have the same type,
  // and is implemented in derived classes:
  virtual const type_info &GetDataType() = 0;

  // These three variable define how to identify an endpoint and how
  // its connection to the other endpoint should be handled.
  char *Scope;
  char *Name;
  int Instance;

  // Indicates whether this buffer has been attached to another yet.
  bool Connected;

  // The bandwidth and latency of the buffer.  This information is
  // stored at each endpoint.
  int Bandwidth;
  int Latency;

  // Fanout for WritePorts only!  Needed here because when sorting and
  // connecting up BasePorts, we need to access Fanout to determine
  // what to connect to what
  int Fanout;

  // For automatic event generation purposes, the node that uses this port must be known 
  int node;
```

Clocked method: scheduler calls active logical activity

feeders: system workload trace

OOO issues: static/dynamic inst traces

separation of feeder & perf model

feeder provide mesg at ticks perf model call it to do so

feeder→logical activicity，perf model→scheduler

![image](https://github.com/arch-simulator-sig/advanced-computer-architecture/assets/100679322/54128f27-0238-4f9d-a9ec-653c398e9747)

architect workbench:

![image](https://github.com/arch-simulator-sig/advanced-computer-architecture/assets/100679322/48654b7c-1e67-4982-a694-8684ce0fb5e0)

specify modules, hierarchy, src file, etc. Build things automatically

asim cache overall abstraction:

1. functionality module(internal feeder)
2. impl <tag, index>pair directly(addr→<tag, index> under XIXT impl is outside of cache module)
3. return cacheline* for processor impl if tag hit(feeder only: no matter hit or miss), else nullptr
4. abstract cacheline data unit(no assumptions about address or Qword alignment)
    
   ![image](https://github.com/arch-simulator-sig/advanced-computer-architecture/assets/100679322/04f27387-f41e-42ae-9e6a-d51fdc1a6c6e)

    
5. base units: line_state_dynamic, lru_info_dynamic, VictimPolicy→cache_class

```jsx
typedef enum
{ 
    S_PERFECT,
    S_WARM,
    S_MODIFIED, 
    S_EXCLUSIVE, 
    S_SHARED, 
    S_INVALID, 
    S_FORWARD, 
    S_LOCKED,
    S_RESERVED,
    S_NC_CLEAN,
    S_NC_DIRTY,
    S_MAX_LINE_STATUS 
} LINE_STATUS;
```

```jsx
class line_state_dynamic
{
    
  private:
    UINT64		tag;
    UINT8		way;
    LINE_STATUS	        status;
    bool		*valid;
    bool		*dirty;
    UINT32              ownerId;
    UINT32              NumObjectsPerLine;
    
    // Stats
    PUBLIC_OBJ_INIT(UINT32, Accesses, 0);
    PUBLIC_OBJ_INIT(UINT64, AccumDistance, 0);
    PUBLIC_OBJ_INIT(UINT64, PreviousCycle, 0);
```

method：ctor, get_xxx, set_xxx, clear, dump(cacheline* wise, dataunit wise), addr misc

```jsx
class lru_info_dynamic
{
  private: 
    struct linklist_s {
        INT8		next;
        INT8		prev;
    } *linklist;

    UINT8		mru;
    UINT8		lru;
```

```jsx
typedef enum 
{ 
    VP_LRUReplacement,
    VP_PseudoLRUReplacement, 
    VP_RandomReplacement, 
    VP_RandomNotMRUReplacement, 
    VP_MaxReplacement
} VICTIM_POLICY;
```

method:  ctor, make_XRU, get_XRU, dump

```jsx
class VictimPolicy
{
    public:
        //
        // As its name indicates, contains all the LRU data structures for each set in the cache
        //
        typedef lru_info_dynamic		lruInfo;
        UINT8           NumWays; 
        UINT32          NumLinesPerWay;
        VICTIM_POLICY   Policy;
		protected:
        lruInfo **LruArray;
```

method: ctor, **GetVictim(replacement wrapper)**

```jsx
class dyn_cache_class : public VictimPolicy
{
  public:
    //typedef line_state_dynamic<NumObjectsPerLine, INFO> lineState;
    typedef line_state_dynamic lineState;

    UINT32 NumObjectsPerLine; 
    bool   WithData;

  private:

    // WARNING! Each cache instance has its own random state to guarantee
    // that the warm-up phase produces the same result (i.e. lockstepped processors).
    #define CACHE_RANDOM_STATE_LENGTH 128
    
    static UINT32 DEFAULT_CACHE_RANDOM_SEED;
    UINT32 random_state[CACHE_RANDOM_STATE_LENGTH / 4];  
  
    // What percent of this cache is warm (cold misses are turned into hits)? 0-100% are legal values.
    const UINT32 warmPercent;
    const UINT64 warmFactor;  ///< warmFactor = warmPercent*RAND_MAX
    const LINE_STATUS initialWarmedState;
    //
    // Tag array holding the contents of the cache and its state.
    //
    //lineState	TagArray[NumLinesPerWay][NumWays];
    lineState	***TagArray;

    //
    // This array reduces to almost nothing if the user sets 'WithData' to
    // FALSE. Otherwise, this array is used to hold the REAL data that would be in
    // the real cache. At this point, our feeders do not still support this
    // ability, but this class is ready to accept data as soon as the feeders
    // become ready
    //
    //T   DataArray[WithData ? NumLinesPerWay : 1][WithData ? NumWays : 1][NumObjectsPerLine];
    UINT32   ***DataArray;
```

method: ctor(default), **Findway(return way index, -1 for notfound/invalid, fill tag if warmup cacheline && notfound)**

```jsx
if ( warmFactor > rand_factor )
                {
                    //              cout << " -- WARM!" << endl;

                    TagArray[index][warm_way]->SetTag(tag);
                    TagArray[index][warm_way]->SetStatus(initialWarmedState);
                    TagArray[index][warm_way]->SetOwnerId(warm_owner);
                    for (UINT32 j=0; j<NumObjectsPerLine; j++)
                    {
                        TagArray[index][warm_way]->SetValidBit(j);
                    }
                    return warm_way;
                }
                else
                {
                    //              cout << " -- COLD!" << endl;

                    TagArray[index][warm_way]->SetTag(tag);
                    TagArray[index][warm_way]->SetStatus(S_INVALID);
                    return -1;
                }
```

method: getter&setter(data, state, lruinfo…), Dump, addr misc

**asim/cache_dyn.h: abstract cache class impl**

**asim/cache_manager.h: system level cache module impl(higher abstraction)**

```jsx
class CACHE_MANAGER
{
  protected:
    class LINE_MANAGER
    {
        struct LINE_STATE
        {
            LINE_STATUS GetStatus() const;
            LINE_STATUS GetStatus(UINT32 owner) const;
            void SetStatus(UINT32 owner, LINE_STATUS status);
            bool IsValid() const
            {
                return !state_.empty();
            };

            std::map<UINT32 /* Owner */, LINE_STATUS /* Status */> state_;
        };

      public:
        typedef std::pair<UINT32 /* Index */, UINT64 /* Tag */> ADDRESS;

        LINE_MANAGER();
        ~LINE_MANAGER();

        LINE_STATUS GetStatus(ADDRESS address) const;
        LINE_STATUS GetStatus(UINT32 owner, ADDRESS address) const;
        void SetStatus(UINT32 owner, ADDRESS address, LINE_STATUS status);
        
      private:
        std::map<ADDRESS, LINE_STATE> addr2status_;
        std::string level_;
    };
		public:
	    static CACHE_MANAGER& GetInstance();
	    virtual void Register(std::string level);
	
	    LINE_STATUS GetStatus(std::string level, UINT32 index, UINT64 tag);
	    LINE_STATUS GetStatus(std::string level, UINT32 owner, UINT32 index, UINT64 tag);
	    void SetStatus(std::string level, UINT32 owner, UINT32 index, UINT64 tag, LINE_STATUS status);
	    
	  private:
	    std::map<std::string, LINE_MANAGER> str2manager_;
	    
	    bool clear_lines;
	    bool activated;
	  
	  protected:
	    virtual LINE_MANAGER *find_line_manager(std::string level);
	
	  public:
	    void setClearLines() { clear_lines = true; }
	    bool getClearLines() { return clear_lines; }
	    
	    void deactivate() { activated = false; }
	
};
```

**asim/line_status.h: specify cache status**

others: cache.h, cache_manager_smp.h, cache_mesi.h（standalone simple impl）
