
## What exactly is a Signal?
A Signal is a reactive container for a value. When its value changes, it automatically notifies and updates any part of the application that depends on it.

<h4>. signal() for local reactive state</h4>  
<h4>. computed() for derived/dependent state</h4>
<h4>. effect() for side effects</h4>
<h4>. input() / output() as signal-based component APIs</h4>
<h4>. toObservable() / toSignal() for bridging with RxJS</h4>

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





