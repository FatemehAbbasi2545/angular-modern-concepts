
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

<h4>2. Dependent Calculations (Derived State with computed)</h4>
Use computed to define state that depends on other Signals. It only recalculates when its dependencies change.

```typescript
items = signal<CartItem[]>([]);
totalPrice = computed(() => 
  this.items().reduce((sum, item) => sum + item.price * item.quantity, 0)
);
itemCount = computed(() => this.items().length);
```

<h4>3. Shared State Between Components (signal + inject)</h4>
For simple cross-component state, Signals can replace RxJS BehaviorSubject without the subscription overhead.

```
typescript
@Injectable({ providedIn: 'root' })
export class AuthService {
  private user = signal<User | null>(null);
  readonly user$ = this.user.asReadonly();

  login(username: string, password: string) {
    this.http.post<User>('/login', { username, password })
      .subscribe(user => this.user.set(user));
  }
}
```

<h4>4. Reactive Component Inputs</h4>
Replace @Input() with input() to get a reactive Signal that can be used in computed or effect.

```typescript
@Component({...})
export class UserCardComponent {
  user = input<User>();
  role = input<'admin' | 'user'>('user');
  userId = input.required<string>();
  
  displayName = computed(() => 
    this.user()?.name ?? `User #${this.userId()}`
  );
}
```

<h4>5. Side Effects with effect</h4>
Run imperative code automatically when Signal dependencies change. Useful for logging, syncing with localStorage, or triggering non-template updates.

```typescript
effect(() => {
  const count = this.notifications().length;
  if (count > 0) {
    console.log(`${count} new notification(s)`);
  }
});
```

<h4>6. Simple Form State & Validation</h4>
For forms that don't require FormsModule or ReactiveFormsModule.

```typescript
@Component({...})
export class LoginFormComponent {
  username = signal('');
  password = signal('');
  
  isFormValid = computed(() => 
    this.username().length > 0 && this.password().length > 3
  );
}
```

## When NOT to Use Signals
Use Case | Reason
|---|---
| Complex async streams | RxJS operators (switchMap, combineLatest, debounceTime, catchError) are still superior for multi-step async pipelines
| Debouncing / Throttling | Signals don't provide these natively; RxJS remains the right tool
| Error recovery chains | 	retry, retryWhen, and catchError are much more ergonomic in RxJS
| Large-scale state management | NgRx, Elf, or similar libraries still offer more structure for complex app-wide state
| Interop with existing RxJS-heavy codebases | Mixing Signals and Observables is fine, but a full migration may not be justified (Combine a stream with several operations)

In modern Angular (17+), the recommended architecture is:

<ul>
  <li><h6>Local state → </h6> Signals + computed</li>
  <li><h6>Async data pipelines → </h6> RxJS</li>
  <li><h6>Connecting the two → </h6> toSignal() and toObservable()</li>
</ul>

Signals and RxJS are complementary, not competitors. Signals handle synchronous reactive state elegantly, while RxJS continues to excel at async event streams and complex transformations.







