# Lands of P(ixel) - Improvement Roadmap

## Critical Performance Fixes (Implement First)

### 1. Spatial Partitioning System ? HIGH PRIORITY
**Problem**: O(n) iteration through all objects every frame
**Solution**: Implement spatial hash grid
**Impact**: 5-10x performance improvement with large object counts
**Files**: Create new `SpatialGrid.h/.cpp`

### 2. Reduce Frame Allocations ?? CRITICAL
**Problem**: Creating vectors every frame in hot paths
**Solution**: Use member variables and reserve capacity
**Impact**: Reduces GC pressure, smoother frame times
**Files**: `Game.cpp` (handleObjectHarvesting, Map.cpp (drawObjects)

### 3. Object Pool for Enemies ? HIGH PRIORITY
**Problem**: Frequent new/delete causing fragmentation
**Solution**: Pre-allocate enemy pool, recycle objects
**Impact**: Eliminates allocation stutter during spawns
**Files**: `EnemyManager.h/.cpp`

### 4. Camera Smoothing ?? IMPROVES FEEL
**Problem**: Jarring camera snapping
**Solution**: Lerp camera position
**Impact**: Much smoother gameplay experience
**Files**: `Game.cpp` (setPlayerView/update)

---

## Gameplay Enhancements

### 1. Stamina System (Medium Priority)
- Sprint: 2x speed, drains stamina
- Attack: Costs stamina
- Mining: Costs stamina
- Regenerates over time
- UI bar next to health

### 2. Progressive Difficulty
- First night: 5 enemies max
- Each subsequent night: +2 enemies
- Enemies get slightly faster/stronger
- Reset on death

### 3. Sound Effects (High Impact)
- Footsteps (muffled on grass, loud on stone)
- Hit sounds (wood thunk, stone crack)
- Attack swoosh
- Enemy groans
- Day/night transition ambient

### 4. Resource Tooltips
- Hover over object shows yield
- "Pine Tree: 3-5 Wood"
- Small floating text

### 5. Minimap
- Top-right corner, 150x150px
- Shows terrain, player dot, enemies (red)
- Fog of war for unexplored areas

### 6. Save/Load System
- Auto-save on day transition
- Manual save via menu
- Save format: JSON
- Store: player pos, inventory, time, seed, killed objects

---

## Code Quality

### 1. Constants File Expansion
Move all magic numbers to `Constants.h`:
```cpp
// Combat
constexpr int PLAYER_MAX_HEALTH = 100;
constexpr float ENEMY_DAMAGE_COOLDOWN = 0.5f;
constexpr float HARVEST_COOLDOWN = 0.3f;

// Spawning
constexpr float ENEMY_SPAWN_INTERVAL = 5.0f;
constexpr int MAX_ENEMIES_BASE = 10;

// View culling
constexpr float VIEW_CULLING_MARGIN = 0.6f;

// Terrain
constexpr float WATER_THRESHOLD = 0.35f;
constexpr float SAND_THRESHOLD = 0.45f;
constexpr float GRASS_THRESHOLD = 0.75f;
constexpr float ROCK_THRESHOLD = 0.90f;
```

### 2. Error Handling
- Return error codes from loadTextures()
- Show error dialog if critical assets missing
- Fallback assets for textures

### 3. Memory Management
- Use std::unique_ptr consistently
- Avoid raw pointers where possible
- RAII for resources

---

## Technical Debt

### 1. Commented Out Code
Remove commented scale code in GameObject.cpp

### 2. Debug Output
Replace std::cout with proper logging system (levels: DEBUG, INFO, WARN, ERROR)

### 3. Thread Safety
Mutex usage in loading could be improved
Consider using atomic operations for flags

---

## Future Features (Low Priority)

1. **Crafting Recipes**: More complex items
2. **Building System**: Place walls, floors, furniture
3. **NPC Villagers**: Friendly NPCs, trading
4. **Biome Variety**: Desert, snow, forest biomes
5. **Underground Layers**: Multiple cave levels
6. **Boss Enemies**: Special night bosses
7. **Multiplayer**: Local co-op
8. **Seasons**: Visual changes, gameplay effects
9. **Weather**: Rain, snow, storms
10. **Quest System**: Simple objectives

---

## Performance Benchmarks to Track

- **FPS**: Target 60 stable
- **Frame Time**: <16.67ms per frame
- **Object Count**: Test with 10,000 objects
- **Enemy Count**: Test with 50+ enemies
- **Memory**: Monitor for leaks

---

## Testing Checklist

- [ ] Test with 250x250 map (current)
- [ ] Test with 500x500 map (performance test)
- [ ] Test with 100 enemies spawned
- [ ] Test cave entry/exit 20 times
- [ ] Test harvesting 500 objects
- [ ] Play for 30 continuous minutes
- [ ] Monitor memory usage over time
- [ ] Test all weapon types
- [ ] Test crafting all recipes
- [ ] Test death/respawn

---

## Build Configuration

### Debug
- Enable all assertions
- Verbose logging
- Memory leak detection
- Frame time overlay

### Release
- Disable logging (except errors)
- Enable optimizations (-O3)
- Strip debug symbols
- Enable LTO (Link Time Optimization)
