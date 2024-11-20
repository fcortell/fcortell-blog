---
title: Do not use ToLower() method for string comparison
date: 2024-11-20 09:05:54 +0200
---

**Don’t use `.ToLower()` for string comparison. Why not?**  

1. **Performance Issues**:  
   The `.ToLower()` method creates a new string in memory every time it’s called, leading to unnecessary memory allocations, especially in high-frequency operations.  

2. **Culture Sensitivity**:  
   The `.ToLower()` method is culture-sensitive, which means results may vary depending on the current culture set for the executing thread.  

**What’s the Better Alternative?**  

Use the `.NET` enumeration **`StringComparison`** for more efficient and reliable string comparisons. ✅  

### `StringComparison` Options:  

- **`Ordinal`**:  
  Use for general-purpose comparisons where culture doesn’t matter. This is the fastest option.  

- **`OrdinalIgnoreCase`**:  
  Use for case-insensitive comparisons without cultural rules.  

- **`CurrentCulture` & `CurrentCultureIgnoreCase`**:  
  Best for comparing user-facing strings, as they adhere to the current culture's rules.  

- **`InvariantCulture` & `InvariantCultureIgnoreCase`**:  
  Ideal for consistent results across different cultures, often used in data storage and retrieval scenarios.  

**Important Note:**  
If you’re using Entity Framework Core, methods like `String.Equals(String, StringComparison)` cannot be translated into SQL and will throw an exception. Be cautious when applying `StringComparison` in LINQ queries.

Source 🔗 [https://thecodeman.net/](https://thecodeman.net/)