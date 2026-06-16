
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

typescript
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

Templates automatically track Signal dependencies — Angular only updates the DOM nodes affected by the change, bypassing full component tree checks.





