## Performance Impact
<h4>1. Eliminates Whole-Component Tree Walks</h4>

With Zone.js, when any async event fires (click, HTTP response, setTimeout), Angular walks the entire component tree from top to bottom, checking every binding for changes. In a large app with hundreds of components, that's a lot of unnecessary work.

```text
Zone.js approach:     [Root] → [A] → [A1] → [A2] → [B] → [B1] → ... → [Z]
                      (every single component, every single time)

Signals approach:     [count changes] → [only button node patched]
                      (no tree walk at all)
```

<h4>2. Granular DOM Updates</h4>
Signals enable node-level change detection instead of component-level. This means Angular knows exactly which DOM node needs updating and can skip the entire component rendering pipeline for unrelated parts.

```typescript
// Template with mixed content
@Component({
  template: `
    <expensive-chart [data]="chartData()" />
    <user-list [users]="users()" />
    <small>Last updated: {{ lastUpdate() }}</small>
  `
})
export class DashboardComponent {
  chartData = signal<DataPoint[]>([]);
  users = signal<User[]>([]);
  lastUpdate = signal(new Date());

  refreshTimestamp() {
    // ✅ Only the <small> element re-renders
    // ❌ expensive-chart and user-list do NOT re-render
    this.lastUpdate.set(new Date());
  }
}
```

<h4>3. Fine-Grained Reactivity with computed</h4>
A computed Signal only recalculates when its direct dependencies change. This cascades efficiently.

```typescript
filteredUsers = computed(() => 
  this.users().filter(u => u.name.includes(this.searchTerm()))
);
// filteredUsers only recalculates when users OR searchTerm changes
// If neither changes, reading filteredUsers returns a cached value
```

<h4>4. Reduced Change Detection Cycles</h4>
With Zone.js, even a trivial click handler triggers a full change detection cycle. With Signals, a click that doesn't change any Signal value triggers zero re-rendering.

```typescript
@Component({...})
export class LoggingComponent {
  // Zone.js: every click triggers full tree check
  // Signals: this click causes zero DOM updates
  logClick() {
    console.log('clicked');
  }
}
```

## Quantified Benefits
For a typical enterprise Angular application with ChangeDetectionStrategy.OnPush already in use:

| Metric | Zone.js (Default) | Zone.js + OnPush | Signals
|---|---|---|---
| Change detection trigger | Any async event | @Input change + explicit markForCheck | Signal value change only
| Scope of check | Full component tree | One component + ancestors | Exact DOM node
| Unnecessary checks | High | Medium | Near zero
| First meaningful paint overhead | Baseline | Slightly better | Best (no tree walk)
| Runtime memory overhead | Low | Low | Moderate (tracking dependencies)

## The Real-World Impact
<b>For small apps (< 50 components): </b>You probably won't notice a difference. Zone.js is already fast enough.

<b>For medium-to-large apps (200+ components):</b>Signals start to shine. The application becomes more predictable — a state change in one part of the tree doesn't accidentally trigger work in unrelated branches. Scroll performance improves because background timers and non-critical Signals don't force full change detection.

<b>For complex dashboards, data-heavy UIs, or real-time apps:</b>This is where Signals truly matter. When you have live-updating data streams, Signals ensure that only the specific cells, rows, or chart elements that actually changed get re-rendered. This can be the difference between a janky 20 FPS experience and a smooth 60 FPS one.

## One Important Caveat
This granular rendering only works when templates read Signals directly — for example {{ mySignal() }} or [value]="mySignal()". If you pass a Signal's value to a child component via @Input(), the child component still updates via the normal input-settling mechanism, though Angular 18+ is optimizing this path as well.

```typescript
// ✅ Granular — only this binding updates
<span>{{ status() }}</span>

// ✅ Also granular
<my-component [value]="status()" />

// ⚠️ Caution: passing a computed Signal to many children
// still triggers input updates for each child, but without a tree walk
```


