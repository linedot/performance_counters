# C++ Wrapper for reading performance counters

Implementations:
  - `performance_counters_papi.cpp` - uses PAPI to read performance counters, supports PAPI counter names
  - `performance_counters_perf.cpp` - uses `perf_event` to read performance counters, supports only some hardcoded counters:
    - `CYCLES`
    - `INSTRUCTIONS`
    - `L1D_READ_ACCESS` 
    - `L1D_READ_MISS`
    - `L1D_WRITE_ACCESS`
    - `L1D_WRITE_MISS`
    - `LL_READ_ACCESS`
    - `LL_READ_MISS`
    - `LL_WRITE_ACCESS`
    - `LL_WRITE_MISS`
  - `performance_counters_m1.cpp` - hacky/unstable API to read Apple Silicon M1 performance counters, requires root, is potentially dangerous and supports only some hardcoded counters:
    - `CPMU_CORE_CYCLES`
    - `CPMU_CORE_CYCLE`
    - `CPMU_INST_A64`
    - `CPMU_INST_BRANCH`
    - `CPMU_SYNC_DC_LOAD_MISS`
    - `CPMU_SYNC_DC_STORE_MISS`
    - `CPMU_SYNC_DTLB_MISS`
    - `CPMU_SYNC_ST_HIT_YNGR_LD`
    - `CPMU_SYNC_BR_ANY_MISP`
    - `CPMU_FED_IC_MISS_DEM`
    - `CPMU_FED_ITLB_MISS`
  - `performance_counters_simple.cpp` - Uses some Arch-specific way to read a counter register, only supports `CYCLES` and is potentially inaccurate (depends on CPU vendor)

  ## Usage: 

  ```cpp
  #if defined(__APPLE__)
  constexpr char cycles_event[] = "CPMU_CORE_CYCLE";
  #else
  constexpr char cycles_event[] = "CYCLES";
  #endif

  [...]
  std::vector<std::string> counters = {{cycles_event}};
  performance_counters pc(counters);

  for(std::size_t i = 0; i < measurements; i++)
  {
  pc.tic();
  // Some calculation to benchmark
  pc.toc_stat();
  }

  auto results = pc.get_counter_statistics();
  for (const auto& res : results)
  {
      auto [name,min,avg,max] = res;
  
      double avg_fp = static_cast<double>(avg);
      std::cout << "Event: " << name << " - min: " << min 
                << "; avg: " << avg_fp
                << "; max: " << std::ceil(static_cast<double>(max)) << "\n";
  }

  ```
