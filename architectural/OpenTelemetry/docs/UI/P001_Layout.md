```css
::deep.layout {
    height: 100vh;
    width: 100vw;
    display: grid;
    grid-template-columns: auto 1fr;
    grid-template-rows: auto auto 1fr;
    grid-template-areas:
    "icon head"
    "nav messagebar"
    "nav main";
    background-color: var(--fill-color);
    color: var(--neutral-foreground-rest);
}
```

---

```
┌───────┬───────────────────────────────┐
│ icon  │             head              │
├───────┼───────────────────────────────┤
│ nav   │             main              │
│       │                               │
│       │                               │
│       │                               │
└───────┴───────────────────────────────┘
```
