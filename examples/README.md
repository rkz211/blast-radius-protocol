# Examples

Annotated examples showing what a Blast Radius codebase looks like in practice.

## Structure

Each example shows:
- A traditional "open hull" version of the same code
- The Blast Radius refactor — same behavior, constrained blast radius
- The contracts for each block

## Coming Soon

Examples are being added. PRs welcome.

---

### What a block looks like

```typescript
// Input: { items: Item[], onSelect: (id: string) => void }
// Output: renders a scrollable list of selectable items
// Must never: fetch data, manage selection state, or handle routing

import React from 'react';
import { Item } from '../types';

interface Props {
  items: Item[];
  onSelect: (id: string) => void;
}

export function ItemList({ items, onSelect }: Props) {
  return (
    <ul className="item-list">
      {items.map(item => (
        <li key={item.id} onClick={() => onSelect(item.id)}>
          {item.label}
        </li>
      ))}
    </ul>
  );
}
```

### What the assembly layer looks like

```typescript
// App.tsx — pure wiring. No logic lives here.
import { useItemData } from './hooks/useItemData';
import { useSelection } from './hooks/useSelection';
import { ItemList } from './components/ItemList';
import { DetailPanel } from './components/DetailPanel';

export default function App() {
  const { items } = useItemData();
  const { selectedId, select } = useSelection();

  return (
    <div className="app">
      <ItemList items={items} onSelect={select} />
      <DetailPanel selectedId={selectedId} />
    </div>
  );
}
```

Notice: no ternaries, no inline logic, no state beyond what's delegated to hooks. If you need to change how items are fetched, touch `useItemData`. If you need to change how items render, touch `ItemList`. These operations never collide.
