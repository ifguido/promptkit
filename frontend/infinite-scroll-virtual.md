# Infinite Scroll with Virtual Scrolling

## Objective
Implement a high-performance infinite scroll component with virtual scrolling (windowing) for React, capable of handling thousands of items efficiently.

## Technology Stack
- React 18+ with TypeScript
- react-virtual or @tanstack/react-virtual for virtualization
- Intersection Observer API for scroll detection
- React Query or SWR for data fetching
- Optional: react-window or react-virtuoso as alternatives

## Requirements

### 1. Core Features

**Infinite Scrolling:**
- Load more data when user scrolls near bottom
- Configurable threshold for triggering load
- Smooth loading transitions
- Support for bi-directional scrolling (load older items when scrolling up)

**Virtual Scrolling:**
- Render only visible items + buffer
- Dynamic item heights supported
- Smooth scroll performance with 10,000+ items
- Automatic height measurement
- Maintain scroll position when items change

**Data Fetching:**
- Paginated API requests
- Loading states
- Error handling with retry
- Cache management
- Optimistic updates

### 2. Components to Build

**VirtualInfiniteList Component:**
```typescript
interface VirtualInfiniteListProps<T> {
  fetchData: (page: number) => Promise<T[]>;
  itemHeight?: number | ((item: T) => number);
  overscan?: number; // Items to render outside viewport
  threshold?: number; // Distance from bottom to trigger load (px)
  renderItem: (item: T, index: number) => React.ReactNode;
  renderLoader?: () => React.ReactNode;
  renderError?: (error: Error, retry: () => void) => React.ReactNode;
  renderEmpty?: () => React.ReactNode;
  estimatedItemHeight?: number;
  hasMore?: boolean;
  className?: string;
  onScrollEnd?: () => void;
}
```

**InfiniteScrollGrid Component:**
- Grid layout with infinite scroll
- Configurable columns
- Responsive column count
- Masonry layout support

**SearchableInfiniteList Component:**
- Combines search with infinite scroll
- Debounced search
- Reset pagination on search
- Loading states for search

### 3. Hooks

**useInfiniteScroll:**
```typescript
const {
  data,
  isLoading,
  isError,
  error,
  hasMore,
  loadMore,
  retry
} = useInfiniteScroll({
  fetchData: async (page) => { /* fetch logic */ },
  pageSize: 20,
  initialPage: 1
});
```

**useVirtualizer:**
```typescript
const {
  virtualItems,
  totalHeight,
  scrollToIndex,
  scrollToOffset
} = useVirtualizer({
  count: items.length,
  getScrollElement: () => scrollRef.current,
  estimateSize: () => 50,
  overscan: 5
});
```

**useIntersectionObserver:**
```typescript
const { ref, inView } = useIntersectionObserver({
  threshold: 0.5,
  triggerOnce: false
});
```

### 4. File Structure
```
src/
├── components/
│   ├── VirtualInfiniteList/
│   │   ├── VirtualInfiniteList.tsx
│   │   ├── InfiniteScrollGrid.tsx
│   │   ├── SearchableInfiniteList.tsx
│   │   ├── ListItem.tsx
│   │   └── LoadingSpinner.tsx
│   └── ErrorBoundary/
│       └── InfiniteScrollError.tsx
├── hooks/
│   ├── useInfiniteScroll.ts
│   ├── useVirtualizer.ts
│   ├── useIntersectionObserver.ts
│   └── useScrollRestoration.ts
├── utils/
│   ├── scrollUtils.ts
│   └── measurementCache.ts
└── types/
    └── infiniteScroll.types.ts
```

### 5. Advanced Features

**Performance Optimizations:**
- Memoize rendered items
- Debounce scroll events
- RequestAnimationFrame for smooth rendering
- Batch DOM measurements
- Cache item heights

**Scroll Restoration:**
- Save scroll position in session storage
- Restore position on navigation back
- Maintain position when items update

**Smart Loading:**
- Prefetch next page before user reaches end
- Cancel pending requests when scrolling fast
- Exponential backoff for retries
- Stale-while-revalidate strategy

**Accessibility:**
- ARIA live regions for loading announcements
- Keyboard navigation support
- Focus management
- Screen reader friendly

### 6. State Management Pattern
```typescript
interface InfiniteScrollState<T> {
  pages: T[][];
  currentPage: number;
  hasMore: boolean;
  isLoading: boolean;
  error: Error | null;
  totalCount?: number;
}
```

### 7. API Integration Example
```typescript
// Backend should support pagination
GET /api/items?page=1&limit=20
{
  data: [...],
  pagination: {
    page: 1,
    limit: 20,
    totalPages: 50,
    totalItems: 1000,
    hasMore: true
  }
}
```

### 8. Features for Different Use Cases

**For Social Media Feeds:**
- Pull to refresh
- New items notification
- Auto-scroll to top
- Gap detection (missing items)

**For Search Results:**
- Faceted search with infinite scroll
- Scroll position preservation on filter change
- Total count display

**For Chat/Messages:**
- Reverse infinite scroll (load older)
- Auto-scroll to bottom on new message
- Stick to bottom behavior
- Jump to specific message

**For Tables:**
- Virtual scrolling with fixed headers
- Column sorting with scroll reset
- Row selection across pages
- Bulk actions

### 9. Error Handling
- Network error recovery
- Empty state when no data
- Rate limit handling
- Timeout handling
- Offline support with cached data

### 10. Performance Metrics
Target performance:
- 60 FPS while scrolling
- < 100ms to render new items
- Support 10,000+ items without lag
- < 50MB memory for large lists
- Smooth on mobile devices

## Deliverables
- VirtualInfiniteList component with all features
- Data fetching integration
- Multiple layout examples (list, grid, masonry)
- Comprehensive error handling
- Loading skeletons
- TypeScript types
- Performance monitoring utilities
- Storybook stories or examples

## Testing Checklist
- [ ] Loads initial data on mount
- [ ] Triggers load more when scrolling near bottom
- [ ] Shows loading indicator while fetching
- [ ] Handles errors gracefully with retry option
- [ ] Shows empty state when no data
- [ ] Virtual scrolling renders only visible items
- [ ] Scroll performance is smooth with 1000+ items
- [ ] Dynamic heights are measured correctly
- [ ] Scroll position is maintained on updates
- [ ] Works on mobile devices
- [ ] Keyboard navigation functions
- [ ] Screen readers announce loading states
- [ ] Memory usage stays reasonable
- [ ] Handles rapid scrolling without issues
- [ ] Cancels requests when unmounting
- [ ] Search resets pagination correctly
