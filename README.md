# Past Mistakes Reference - dtngoc49
# Copilotレビュー時のルール

- 日本語でレビューを行うこと
- レビュー観点は以下ガイドラインに従うこと
- ガイドラインの記述内容は信頼できるものとし、違反している場合は必ずレビューコメントを記述すること
- 信頼性が低くてもコメント抑制はせず、必ずレビューコメントを記述すること
-
> **62 code review issues to avoid repeating**

---

## MUST (23 items) - Critical Issues

### 1. Always verify specification before implementation
```javascript
// ❌ BAD
if (e.response?.status === 401) {
  ret.errorMsg = e.response.data.message; // Server message
}
// ✅ GOOD
if (e.response?.status < 500) {
  ret.errorMsg = "{{ __('reply_assist_panel.auth_permission_error') }}"; // Frontend fixed text
}
```

### 2. Understand flag purpose - isOverGenerateLimit is ONLY for request limit exceeded
```javascript
// ❌ BAD
const isOverGenerateLimit = isCustomGeneration;
// ✅ GOOD
this.isOverGenerateLimit = isOverLimit; // From API response
```

### 3. Prevent duplicate requests correctly
```javascript
// ❌ BAD
if (isLoading) { makeRequest(); }
// ✅ GOOD
if (!isLoading) { setLoading(true); makeRequest(); }
```

### 4. Custom generation has NO knowledge sources
```javascript
// ❌ BAD
canShowSources(item) {
  return item.knowledgeSources;
}
// ✅ GOOD
canShowSources(item) {
  return item.knowledgeSources && item.generateType === GENERATE_TYPE.AUTO;
}
```

### 5. One-time operations should NOT be inside loops
```javascript
// ❌ BAD
this.list.forEach(item => {
  this.appendItem(item);
  this.initCopyLink(); // Wrong - should be outside
});
// ✅ GOOD
this.list.forEach(item => this.appendItem(item));
await this.$nextTick();
this.displayList.forEach(item => this.initCopyLink(item.id));
```

### 6. ALWAYS use braces for control statements (違反4回)
```javascript
// ❌ BAD
if (itemId) itemElem = querySelector(`[data-id='${itemId}']`);
// ✅ GOOD
if (itemId) {
  itemElem = querySelector(`[data-id='${itemId}']`);
}
```

### 7. Delete unused code (違反3回)
```javascript
// ❌ BAD
const unusedTimeout = setTimeout(() => {}, 1000); // Never cleared
// ✅ GOOD
if (this.timeout) clearTimeout(this.timeout);
this.timeout = setTimeout(() => { this.timeout = null; }, 500);
```

### 8. Use blade comments, NOT HTML comments
```blade
{{-- ❌ BAD --}}
<!-- Comment -->
{{-- ✅ GOOD --}}
{{-- Comment --}}
```

### 9. Remove unnecessary props after conflict resolution
```javascript
// ❌ BAD
<Component unnecessaryProp={value} requiredProp={data} />
// ✅ GOOD
<Component requiredProp={data} />
```

### 10. Initialize timeout variables after clearing
```javascript
// ❌ BAD
clearTimeout(this.timeout);
// ✅ GOOD
clearTimeout(this.timeout);
this.timeout = null;
```

### 11. Get config values from server, not hard-coded
```javascript
// ❌ BAD
const DISPLAY_LIMIT = 5;
// ✅ GOOD
const displayLimit = props.displayLimit || DEFAULT_LIMIT;
```

### 12. Use existing appropriate functions
```javascript
// ❌ BAD
this.timelineList.push(newItem);
// ✅ GOOD
this.appendTimelineItem(newItem);
```

### 13. Use affirmative variable names, not negative
```javascript
// ❌ BAD
const hasNoReference = !item.reference;
// ✅ GOOD
const hasReference = !!item.reference;
```

### 14. Extract complex conditions to computed properties
```vue
<!-- ❌ BAD -->
<div v-if="item.generateType === GENERATE_TYPE.AUTO && !item.knowledgeSources && item.isCompleted">
<!-- ✅ GOOD -->
<div v-if="isNoKnowledgeSources(item)">
```

### 15. Verify server/frontend error handling responsibility
```javascript
// ❌ BAD
showError(error.response.data.message); // Server dependent
// ✅ GOOD - 401/403 use frontend fixed text
if ([401, 403].includes(status)) {
  showError("認証が必要です");
}
```

### 16. Understand error pattern processing flow
```javascript
// Check: Does error handling match specification?
// Check: Are client/server responsibilities clear?
```

### 17. Set configuration values dynamically
```javascript
// ❌ BAD: Hard-coded values that may change
// ✅ GOOD: Receive from server or props
```

### 18. Use existing functions, don't recreate
```javascript
// Check: Does a similar function already exist?
// Check: Am I using the project's standard approach?
```

### 19. No logical contradictions in conditions
```javascript
// Check: Do my conditions make logical sense?
// Check: Are flags set correctly based on their purpose?
```

### 20. Correctly determine loop processing scope
```javascript
// Check: Should this run once or multiple times?
// Check: Is this operation dependent on each item?
```

### 21. Follow specification for message display
```javascript
// Check: Does message match specification document?
// Check: Am I using correct message source (frontend/backend)?
```

### 22. Understand multi-request prevention specs
```javascript
// Check: Am I preventing duplicate requests correctly?
// Check: Is the loading state managed properly?
```

### 23. Props specified correctly
```javascript
// Check: Are all props necessary?
// Check: Did conflict resolution restore deleted props?
```

---

## IMO (18 items) - Important Issues

### 24. Follow existing codebase naming conventions
```javascript
// ❌ BAD
const getData = () => {}; // When codebase uses fetchData
// ✅ GOOD
const fetchData = () => {};
```

### 25. Use is/has prefix correctly
```javascript
// ❌ BAD
const isReference = item.reference;
// ✅ GOOD
const hasReference = !!item.reference;
```

### 26. Avoid duplicate error handling code
```javascript
// ❌ BAD
if (status === 401) showError("Auth error");
if (status === 403) showError("Auth error");
// ✅ GOOD
if ([401, 403].includes(status)) showError("Auth error");
```

### 27. Use CSS pseudo-selectors instead of dedicated classes
```css
/* ❌ BAD */
.timeline-item.last-item { margin-bottom: 0; }
/* ✅ GOOD */
.timeline-item:last-child { margin-bottom: 0; }
```

### 28. Ensure v-if/v-else-if exclusivity
```vue
<!-- ❌ BAD -->
<div v-if="condition1">Content 1</div>
<div v-if="condition2">Content 2</div>
<!-- ✅ GOOD -->
<div v-if="condition1">Content 1</div>
<div v-else-if="condition2">Content 2</div>
```

### 29. Group props and events logically
```vue
<!-- ❌ BAD -->
<Component @click="h1" :prop1="v1" @change="h2" :prop2="v2" />
<!-- ✅ GOOD -->
<Component :prop1="v1" :prop2="v2" @click="h1" @change="h2" />
```

### 30. Extract complex template conditions
```vue
<!-- Check: Is this condition too complex for inline? -->
<!-- Check: Should this be a computed property? -->
```

### 31. Variable naming should be clear
```javascript
// Check: Is this name affirmative and clear?
// Check: Does it follow project conventions?
```

### 32. Avoid redundant code
```javascript
// Check: Am I repeating similar code?
// Check: Can this be unified?
```

### 33. CSS class usage
```css
/* Check: Can I use pseudo-selector instead? */
/* Check: Am I creating unnecessary classes? */
```

### 34. Vue conditional rendering
```vue
<!-- Check: Are conditions mutually exclusive? -->
<!-- Check: Should I use v-else-if instead of v-if? -->
```

### 35. Component organization
```vue
<!-- Check: Are props grouped together? -->
<!-- Check: Are events grouped together? -->
```

### 36. Follow naming patterns
```javascript
// Check: Does this match existing code patterns?
// Check: Is prefix (is/has/get/fetch) correct?
```

### 37. Code duplication check
```javascript
// Check: Is similar code elsewhere?
// Check: Can I extract common logic?
```

### 38. Styling approach
```css
/* Check: Am I following project CSS patterns? */
/* Check: Can native CSS features handle this? */
```

### 39. Template readability
```vue
<!-- Check: Is template logic readable? -->
<!-- Check: Should logic move to script section? -->
```

### 40. Consistent organization
```vue
<!-- Check: Is component structure consistent? -->
<!-- Check: Are sections properly ordered? -->
```

### 41. Naming consistency across files
```javascript
// Check: Do similar functions have similar names?
// Check: Am I following file-specific conventions?
```

---

## ASK (14 items) - Review Issues

### 42. Check data existence before access
```javascript
// ❌ BAD
const length = data.items.length;
// ✅ GOOD
const length = data?.items?.length || 0;
```

### 43. Handle whitespace-only input
```javascript
// ❌ BAD
if (input === '') { /* handle */ }
// ✅ GOOD
if (input.trim() === '') { /* handle */ }
```

### 44. Set async completion state
```javascript
// ❌ BAD
async function process() {
  await apiCall();
}
// ✅ GOOD
async function process() {
  try {
    await apiCall();
    this.isCompleted = true;
  } catch (e) {
    this.hasError = true;
  }
}
```

### 45. Reuse existing components
```javascript
// ❌ BAD
import { NewIcon } from './new/Icon';
// ✅ GOOD
import { Icon } from './existing/Icon';
```

### 46. Consider null/undefined data
```javascript
// Check: What if data doesn't exist?
// Check: What if property is undefined?
```

### 47. Consider edge cases - empty/whitespace input
```javascript
// Check: What if input is only spaces?
// Check: What if input is empty string?
```

### 48. Verify timeline addition conditions
```javascript
// Check: When should items be added to timeline?
// Check: Are conditions clear and documented?
```

### 49. Handle async completion properly
```javascript
// Check: Is completion state set?
// Check: Is error state handled?
```

### 50. Component selection
```javascript
// Check: Does similar component exist?
// Check: Can I reuse existing component?
```

### 51. Data validation
```javascript
// Check: Am I validating data existence?
// Check: Am I handling missing data gracefully?
```

### 52. Input validation
```javascript
// Check: Am I checking for empty input?
// Check: Am I checking for whitespace-only input?
```

### 53. Timeline logic
```javascript
// Check: Are timeline addition conditions clear?
// Check: Is timing correct (isCompleted, hasError)?
```

### 54. Async state management
```javascript
// Check: Am I tracking operation state?
// Check: Am I setting completion/error flags?
```

### 55. Component reusability
```javascript
// Check: Can I use existing component?
// Check: Does new component add value?
```

---

## NITS (7 items) - Minor Issues

### 56. Follow language file naming conventions
```javascript
// ❌ BAD
'reply_assist.CustomMessage'
// ✅ GOOD
'reply_assist.custom_message'
```

### 57. Language file naming
```javascript
// Check: Am I following snake_case for language keys?
// Check: Is casing consistent with project?
```

### 58. File naming conventions
```javascript
// Check: Does file name match project conventions?
// Check: Is naming consistent with similar files?
```

### 59. Code formatting
```javascript
// Check: Does formatting match project style?
// Check: Am I following linter rules?
```

### 60. Comment style
```javascript
// Check: Am I using correct comment style?
// Check: Blade vs HTML comments in templates?
```

### 61. Import organization
```javascript
// Check: Are imports grouped correctly?
// Check: Are imports ordered consistently?
```

### 62. Code structure
```javascript
// Check: Is code structure consistent?
// Check: Are sections properly organized?
```

---

## Most Frequent Past Mistakes

1. **波括弧を省略しないでください** - Omitting braces (4回)
2. **使っていないなら削除** - Delete unused code (3回)
3. **仕様書で確認してください** - Verify specification (7回)
4. **401/403の時はフロント側固定文言** - Frontend fixed text for 401/403
5. **カスタム生成時はナレッジなし** - No knowledge sources for custom generation
6. **isOverGenerateLimitはリクエスト上限超過時のみ** - isOverGenerateLimit only for limit exceeded
