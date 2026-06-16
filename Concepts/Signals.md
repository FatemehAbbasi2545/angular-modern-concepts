
## What exactly is a Signal?
A Signal is a reactive container for a value. When its value changes, it automatically notifies and updates any part of the application that depends on it.

<ul>
  <li><code>signal()</code> for local reactive state</li>
  <li><code>computed()</code> for derived/dependent state</li>
  <li><code>effect()</code> for side effects</li>
  <li><code>input() / output()</code> as signal-based component APIs</li>
  <li><code>toObservable() / toSignal()</code> for bridging with RxJS</li>
</ul>

These are all stable APIs shipped in Angular 17+ and are considered the standard approach moving forward. The team has made it clear that Signals are the future of change detection in Angular, with Zone.js being optional from v18 onward.

## When to Use Angular Signals
Signals are reactive containers that notify consumers when their value changes. Unlike plain variables or traditional change detection, Signals provide precise, fine-grained reactivity — only the parts of the template or computed values that depend on a changed Signal are re-evaluated.

<h4>1. Local Component State</h4>
Replace simple component properties with Signals for better reactivity and clarity.

```typescript
@Component({...})
export class SearchComponent {
  searchTerm = signal('');
  isLoading = signal(false);
  results = signal<User[]>([]);

  search(query: string) {
    this.isLoading.set(true);
    this.service.search(query).subscribe(res => {
      this.results.set(res);
      this.isLoading.set(false);
    });
  }
}
```

Templates automatically track Signal dependencies — Angular only updates the DOM nodes affected by the change, bypassing full component tree checks.

when a Signal changes, only the parts of the template that actually read that Signal are re-rendered. This is the fundamental difference from the traditional Zone.js-based change detection.

When you call a Signal inside a template — for example {{ count() }} — Angular's template compiler automatically tracks that dependency. At runtime, Angular registers a reactive consumer for each template expression that reads a Signal.

```typescript
@Component({
  template: `
    <h1>{{ title() }}</h1>       <!-- depends on title Signal -->
    <p>{{ description() }}</p>   <!-- depends on description Signal -->
    <button (click)="increment()">{{ count() }}</button>  <!-- depends on count Signal -->
  `
})
export class MyComponent {
  title = signal('Hello');
  description = signal('Some text');
  count = signal(0);
}
```

If count changes, Angular only re-evaluates the {{ count() }} expression and patches the button's text node. The h1 and p elements are untouched — they never even get checked.





