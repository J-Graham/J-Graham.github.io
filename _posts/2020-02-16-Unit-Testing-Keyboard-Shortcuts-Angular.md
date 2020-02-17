---
layout: post
title: Unit Testing Keyboard Shortcut Actions in Angular(9)
---

![settings-image](/assets/unit-test.jpg)

[Jump to the solution](#solution)

## The Use Case

Tonight I was doing some practice writing out a small personal project.  In this project I was trying out a few new development techniques.

First I wanted to try my hand at some Test Driven Development (TDD).  I'll follow-up with a future blog post on my experiences here.

I also wanted to experiment and get my feet wet with the recent release of Angular 9.

Sparing the details this application will be heavily using keyboard shortcuts.  I decided to use a pretty awesome open source package my company built called [ngx-keyboard-shortcuts](https://github.com/milestechnologies/ngx-keyboard-shortcuts).

With this package I setup a hotkey "Control + k" that would toggle the boolean value of a property on my component.  It ended up looking like this:

```typescript
export class ShortcutReaderComponent implements OnInit {
    startCapturingHotKey: IKeyboardShortcutListenerOptions = {
        description: 'Start Capture Keyboard',
        keyBinding: [KeyboardKeys.Ctrl, 'k'],
    };

    isCapturingHotKey = false;

    constructor() {}

    ngOnInit(): void {}

    startListening() {
        this.isCapturingHotKey = true;
    }
}
```
```html
<div class="jumbotron">
    <h3>Save a New Hotkey!</h3>
    <br />
    <div class="container">
        <div class="row">
            <div class="col-sm">
                <span class="lead">Press <span id="keyboardCallout" class="shadow p-3 bg-white rounded"
                        [keyboardShortcut]="startCapturingHotKey" (click)="startListening()">Ctrl + Enter</span> to
                    start tracking
                    your hotkey.</span>
            </div>
        </div>
    </div>
</div>
```

## Where I Started to have Trouble

I ran into a few problems that I thought would be valuable to share.  

1. I was using the `fixture.nativeElement.dispatchEvent` to trigger my keydown event.  I found out after some debugging that this was never being picked up by the package as a keypress.
1. I was trying to trigger 2 Keyboard events in order.  One for the "control" key and another for the "k" key.  This was being picked up as 2 separate keys not in succession.
1. I did not start the test with a `fixture.detectChanges()` to make sure that the DOM had rendered and my directive was in place.

## Solution

My solution turned out to be pretty simple like most things once you do some research.  As I stated above one of my first issues was that the event's weren't being detected.  I switched to `window.dispatchEvent()` and this solved that problem.

Next I tackled the problem of trying 2 keyboard events.  After reviewing the [KeyboardEvent docs](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/KeyboardEvent), I found a property that looked interesting for my case.  It was the `ctrlKey` constructor property.  I set that to true in my "k" key event, added the extra `fixture.detectChanges()` and ... success!

Here is the final solution I ended up with for the test:

```typescript
describe('::enableHotKey', () => {
      it('should set isCapturingHotKey property to be true', () => {
          fixture.detectChanges();
          window.dispatchEvent(new KeyboardEvent('keydown', { key: 'k', ctrlKey: true }));
          fixture.detectChanges();
          expect(component.isCapturingHotKey).toBeTrue();
      });
});
```
