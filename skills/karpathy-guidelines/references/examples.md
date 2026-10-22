# Karpathy Guidelines - Examples & Anti-Patterns

Real-world code examples demonstrating the four principles. Each example contrasts what LLMs commonly do wrong with the expected surgical behavior.

---

## 1. Think Before Coding

### Example 1: Hidden Assumptions

**User Request:** "Add a feature to export user data"

#### ❌ What LLMs Do (Wrong Assumptions)

```python
def export_users(format='json'):
    """Export all users to a file."""
    users = User.query.all()
    
    if format == 'json':
        with open('users.json', 'w') as f:
            json.dump([u.to_dict() for u in users], f)
    elif format == 'csv':
        with open('users.csv', 'w') as f:
            writer = csv.DictWriter(f, fieldnames=['id', 'email', 'name'])
            writer.writeheader()
            writer.writerows([u.to_dict() for u in users])
    
    return f"Exported {len(users)} users"
```

**Problems:**
- Assumed it should export ALL users without pagination or filters (privacy / scale issues).
- Assumed local file write location without asking.
- Assumed which specific user fields to include.
- Assumed format defaults.

#### ✅ What Should Happen (Surface Assumptions First)

```text
Before implementing, I'd like to clarify a few details:

1. Scope: Should this export all users or a filtered subset?
2. Delivery format:
   - Direct download in browser?
   - Background export job with a signed download link?
   - API endpoint returning paginated JSON?
3. Sensitivity: Which fields should be exported? (e.g., exclude internal auth tokens or password hashes)
4. Volume: What is the typical user count?

Simplest approach: An authenticated endpoint returning paginated JSON.
Would you like to proceed with that, or did you have a specific file export in mind?
```

---

### Example 2: Multiple Interpretations

**User Request:** "Make the search faster"

#### ❌ What LLMs Do (Pick Silently)

```python
# Adds caching, database indexes, and async processing without asking which bottleneck matters
@lru_cache(maxsize=1000)
async def search(query: str) -> List[Result]:
    # ... 200 lines of complex caching and async queue logic
```

#### ✅ What Should Happen (Present Options)

```text
"Make the search faster" could target different bottlenecks:

1. Query latency (reducing response time from ~500ms to <100ms):
   - Add targeted database indexes on searched columns
   - In-memory cache for hot queries

2. Throughput (handling more concurrent search requests):
   - Connection pooling / async query worker

3. Perceived UX latency:
   - Return initial batch immediately and stream remaining results

What is the primary bottleneck you are experiencing right now?
```

---

## 2. Simplicity First

### Example 1: Over-abstraction

**User Request:** "Add a function to calculate discount"

#### ❌ What LLMs Do (Overengineered)

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, amount: float) -> float:
        pass

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percentage: float):
        self.percentage = percentage
    
    def calculate(self, amount: float) -> float:
        return amount * (self.percentage / 100)

class FixedDiscount(DiscountStrategy):
    def __init__(self, fixed_amount: float):
        self.fixed_amount = fixed_amount
    
    def calculate(self, amount: float) -> float:
        return min(self.fixed_amount, amount)

@dataclass
class DiscountConfig:
    strategy: DiscountStrategy
    min_purchase: float = 0.0
    max_discount: float = float('inf')

class DiscountCalculator:
    def __init__(self, config: DiscountConfig):
        self.config = config
    
    def apply_discount(self, amount: float) -> float:
        if amount < self.config.min_purchase:
            return 0
        discount = self.config.strategy.calculate(amount)
        return min(discount, self.config.max_discount)
```

#### ✅ What Should Happen (Simple & Direct)

```python
def calculate_discount(amount: float, percent: float) -> float:
    """Calculate discount amount. percent should be 0-100."""
    return amount * (percent / 100)
```

> **Rule:** Add strategy patterns only when multiple dynamic discount strategies are actually requested and required by current business logic.

---

### Example 2: Speculative Features

**User Request:** "Save user preferences to database"

#### ❌ What LLMs Do (Unnecessary Features & Hooks)

```python
class PreferenceManager:
    def __init__(self, db, cache=None, validator=None):
        self.db = db
        self.cache = cache or InMemoryCache()
        self.validator = validator or DefaultValidator()
    
    def save(self, user_id: int, prefs: dict, 
             merge: bool = True, 
             validate: bool = True,
             notify: bool = False) -> bool:
        # 40+ lines of caching, merge strategies, validation event dispatchers...
```

#### ✅ What Should Happen (Just What Was Asked)

```python
def save_preferences(db, user_id: int, preferences: dict):
    """Save user preferences to database."""
    db.execute(
        "UPDATE users SET preferences = ? WHERE id = ?",
        (json.dumps(preferences), user_id)
    )
```

---

## 3. Surgical Changes

### Example 1: Drive-by Refactoring

**User Request:** "Fix the bug where empty emails crash the validator"

#### ❌ What LLMs Do (Too Much Unsolicited Editing)

```diff
  def validate_user(user_data):
-     # Check email format
-     if not user_data.get('email'):
+     """Validate user data."""
+     email = user_data.get('email', '').strip()
+     
+     # Validate email
+     if not email:
          raise ValueError("Email required")
-     
-     # Basic email validation
-     if '@' not in user_data['email']:
+     if '@' not in email or '.' not in email.split('@')[1]:
+         raise ValueError("Invalid email")
      
-     # Check username
-     if not user_data.get('username'):
+     # Validate username
+     username = user_data.get('username', '').strip()
+     if not username:
          raise ValueError("Username required")
+     if len(username) < 3:
+         raise ValueError("Username too short")
```

#### ✅ What Should Happen (Surgical Fix Only)

```diff
  def validate_user(user_data):
      # Check email format
-     if not user_data.get('email'):
+     email = user_data.get('email', '')
+     if not email or not email.strip():
          raise ValueError("Email required")
      
      # Basic email validation
      if '@' not in user_data['email']:
          raise ValueError("Invalid email")
      
      # Check username
      if not user_data.get('username'):
          raise ValueError("Username required")
      
      return True
```

---

### Example 2: Style Drift

**User Request:** "Add logging to the upload function"

#### ❌ What LLMs Do (Reformat Quotes, Types, Spacing)

```diff
- def upload_file(file_path, destination):
+ def upload_file(file_path: str, destination: str) -> bool:
+     """Upload file to destination with logging."""
+     logger.info(f"Uploading {file_path} to {destination}")
      try:
-         with open(file_path, 'rb') as f:
-             data = f.read()
-         response = requests.post(destination, files={'file': data})
+         with open(file_path, "rb") as f:
+             data = f.read()
+         response = requests.post(destination, files={"file": data})
```

#### ✅ What Should Happen (Match Existing Style)

```diff
+ import logging
+ 
+ logger = logging.getLogger(__name__)
+ 
  def upload_file(file_path, destination):
+     logger.info(f'Starting upload: {file_path}')
      try:
          with open(file_path, 'rb') as f:
              data = f.read()
          
          response = requests.post(destination, files={'file': data})
          
          if response.status_code == 200:
+             logger.info(f'Upload successful: {file_path}')
              return True
          else:
+             logger.error(f'Upload failed: {file_path}, status={response.status_code}')
              return False
      except Exception as e:
-         print(f"Error: {e}")
+         logger.exception(f'Upload error: {file_path}')
          return False
```

---

## 4. Goal-Driven Execution

### Example 1: Vague Approach vs. Verifiable Plan

**User Request:** "Fix the authentication session bug"

#### ❌ What LLMs Do

```text
I will fix the auth bug:
1. Review the code
2. Find the problem
3. Update functions
4. Test it
[Proceeds to touch 5 files without clear success criteria]
```

#### ✅ What Should Happen

```text
Goal: Invalidate stale user sessions when a password is changed.

Verification plan:
1. Step 1 (Reproduction): Write a test where a user changes password and makes a request with old session token.
   → Verify: Test fails with expected bug behavior.
2. Step 2 (Fix): Add session token invalidation hook on password update.
   → Verify: Reproduction test passes.
3. Step 3 (Regression): Run full auth test suite.
   → Verify: All existing auth tests pass with 0 errors.
```

---

## Anti-Patterns Summary Table

| Principle | Anti-Pattern | Recommended Practice |
| :--- | :--- | :--- |
| **Think Before Coding** | Silently assuming format, scope, or architecture | State assumptions clearly; ask if multiple interpretations exist |
| **Simplicity First** | Overengineered design patterns for single-use logic | Plain functions and direct logic until complexity is demanded |
| **Surgical Changes** | Reformatted unrelated lines, added unrequested type hints or docstrings | Only modify lines directly responsible for fulfilling the prompt |
| **Goal-Driven** | "I will improve and test the code" | Define reproducible failure → apply fix → verify green tests |
