# Quick Reference: Using the Spatial Grid

## For Future Development

When adding new features that need to find objects, use the spatial grid instead of iterating!

---

## ? DON'T DO THIS (Slow O(n)):

```cpp
// BAD: Iterates ALL objects
for (auto& obj : m_mapGen.getObjects()) {
    if (isNear(obj, player)) {
        // do something
    }
}
```

---

## ? DO THIS INSTEAD (Fast O(1)):

### Find objects in a radius:
```cpp
std::vector<GameObject*> nearby;
m_mapGen.queryObjectsInRadius(playerPos, 100.0f, nearby);

for (GameObject* obj : nearby) {
    // only checking ~10 objects instead of 5000!
}
```

### Find objects in a rectangle:
```cpp
sf::FloatRect searchArea({ x, y }, { width, height });
std::vector<GameObject*> results;
m_mapGen.queryObjectsInRegion(searchArea, results);
```

---

## Common Use Cases

### 1. Weapon/Attack Range
```cpp
void checkAttackHits() {
    std::vector<GameObject*> nearby;
    float attackRange = 50.0f;
    m_mapGen.queryObjectsInRadius(playerPos, attackRange, nearby);
    
    for (GameObject* obj : nearby) {
        if (weaponHitbox.intersects(obj->getBounds())) {
            obj->takeDamage(damage);
        }
    }
}
```

### 2. NPC/Enemy AI Looking for Targets
```cpp
void Enemy::findNearbyTargets() {
    std::vector<GameObject*> nearby;
    float sightRange = 200.0f;
    map->queryObjectsInRadius(getPosition(), sightRange, nearby);
    
    for (GameObject* obj : nearby) {
        if (obj->getType() == ObjectType::PLAYER) {
            targetPlayer(obj);
            break;
        }
    }
}
```

### 3. Area of Effect (AOE) Abilities
```cpp
void castFireball(sf::Vector2f position) {
    std::vector<GameObject*> affected;
    float explosionRadius = 75.0f;
    m_mapGen.queryObjectsInRadius(position, explosionRadius, affected);
    
    for (GameObject* obj : affected) {
        obj->takeDamage(50);
        obj->setOnFire();
    }
}
```

### 4. Building Placement (Check for Obstacles)
```cpp
bool canPlaceBuilding(sf::Vector2f position, float size) {
    sf::FloatRect buildArea(position, { size, size });
    std::vector<GameObject*> obstacles;
    m_mapGen.queryObjectsInRegion(buildArea, obstacles);
    
    return obstacles.empty(); // Can only build if area is clear
}
```

### 5. Resource Gathering Range
```cpp
void gatherNearbyResources() {
    std::vector<GameObject*> nearby;
    float gatherRange = 150.0f;
    m_mapGen.queryObjectsInRadius(playerPos, gatherRange, nearby);
    
    for (GameObject* obj : nearby) {
        if (obj->getType() == ObjectType::TREE ||
            obj->getType() == ObjectType::ROCK) {
            showGatherPrompt(obj);
        }
    }
}
```

---

## Performance Guidelines

### Cell Size (Currently 64px)
- **Good for**: Medium-sized objects (trees, rocks, bushes)
- **Change if**: Objects are much smaller or larger
- **Modify in**: `Map.cpp` constructor `m_spatialGrid(64.0f)`

### Query Radius
- **Rule**: Query smallest radius possible
- **Good**: `queryRadius(playerPos, 50.0f)` ? checks ~4 cells
- **Bad**: `queryRadius(playerPos, 500.0f)` ? checks ~64 cells

### When NOT to Use Grid
- **Single object access**: Just use direct reference
- **Player position**: Already have the reference
- **UI elements**: Not in the grid

---

## Debug Tips

### Print Query Results:
```cpp
std::vector<GameObject*> results;
m_mapGen.queryObjectsInRadius(pos, radius, results);
std::cout << "Found " << results.size() << " objects" << std::endl;
```

### Visualize Query Area (Debug Mode):
```cpp
#ifdef _DEBUG
sf::CircleShape debugCircle(queryRadius);
debugCircle.setOrigin(queryRadius, queryRadius);
debugCircle.setPosition(queryCenter);
debugCircle.setFillColor(sf::Color(255, 0, 0, 50)); // Transparent red
debugCircle.setOutlineColor(sf::Color::Red);
debugCircle.setOutlineThickness(2.0f);
window.draw(debugCircle);
#endif
```

---

## Common Mistakes to Avoid

### ? Mistake 1: Querying every frame unnecessarily
```cpp
// BAD: Queries even when player isn't moving
void update() {
    queryObjectsInRadius(playerPos, range, results);
}

// GOOD: Only query when needed
void update() {
    if (playerMoved || needsUpdate) {
        queryObjectsInRadius(playerPos, range, results);
    }
}
```

### ? Mistake 2: Query radius too large
```cpp
// BAD: Checks entire screen
queryObjectsInRadius(pos, 1000.0f, results); // Way too big!

// GOOD: Only check what you need
queryObjectsInRadius(pos, weaponRange + 10.0f, results);
```

### ? Mistake 3: Not checking for null
```cpp
// BAD: Could crash
for (GameObject* obj : results) {
    obj->doSomething(); // What if obj is nullptr?
}

// GOOD: Always check
for (GameObject* obj : results) {
    if (obj && !obj->isDestroyed()) {
        obj->doSomething();
    }
}
```

---

## Integration Checklist

When adding a new system that needs object queries:

- [ ] Use `queryObjectsInRadius()` or `queryObjectsInRegion()`
- [ ] Choose smallest reasonable query radius
- [ ] Check for null/destroyed objects
- [ ] Only query when necessary (not every frame)
- [ ] Test with large object counts (5000+)
- [ ] Profile if performance-critical

---

## Further Reading

See `SPATIAL_PARTITIONING_IMPLEMENTATION.md` for:
- How the grid works internally
- Performance metrics
- Memory usage
- Advanced tuning

---

**Remember: Spatial grid = Fast queries. Use it whenever you need to find nearby objects!**
