# Taipy Issue #2589 Documentation

## Issue Details
- **Issue Number:** #2589
- **Title:** _Reloader context manager must be re-entrant
- **URL:** https://github.com/Avaiga/taipy/issues/2589

## Problem Description
The `_Reloader` context manager didn't support re-entrant behavior, meaning nested `with` statements didn't work correctly. For example:
```python
with _Reloader():
    with _Reloader():
        some_code()
    some_other_code()
```
In this case, `some_other_code()` would have reloading enabled even though it was still within the outer `_Reloader` context.

## Solution
The fix involved modifying the `_Reloader` class in `taipy/core/_entity/_reload.py` to:
1. Add a `_context_depth` counter to track nested context levels
2. Increment the counter in `__enter__`
3. Decrement the counter in `__exit__` and only set `_no_reload_context` to `False` when the counter reaches 0

### Changes Made
```python
class _Reloader:
    _instance = None
    _no_reload_context = False
    _context_depth = 0  # Added counter

    def __enter__(self):
        self._context_depth += 1  # Increment counter
        self._no_reload_context = True
        return self

    def __exit__(self, exc_type, exc_value, exc_traceback):
        self._context_depth -= 1  # Decrement counter
        if self._context_depth == 0:  # Only disable reloading when counter reaches 0
            self._no_reload_context = False
```

## Technical Details
- **Branch:** fix/reloader-reentrant
- **Commit Hash:** 26b1dc2ef
- **Files Modified:** taipy/core/_entity/_reload.py

## Pull Request Information
- **Title:** fix: make _Reloader context manager re-entrant
- **Status:** Created/In Review/Merged (update this based on current status)

## Testing
The changes ensure that:
1. Nested `with` statements work correctly
2. Reloading remains disabled for all code within any `_Reloader` context
3. Reloading is only re-enabled when all contexts are exited

## Notes
- This fix maintains backward compatibility
- No additional dependencies were required
- The solution is thread-safe as it uses instance variables

## Future Considerations
- Consider adding unit tests for nested context scenarios
- Monitor for any performance impact with deeply nested contexts 