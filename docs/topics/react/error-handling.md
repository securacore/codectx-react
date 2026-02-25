# Error Handling and Suspense

Error handling and loading states in React components. Both concerns share the same placement heuristic: isolation boundaries. For the reasoning behind these conventions, see [spec/README.md](spec/README.md).

## Error Boundaries

<rules>

- Use the `react-error-boundary` package. Do not write custom class-based Error Boundary components.
- Use a single `ErrorBoundary` component with a configurable `fallback` prop. Do not create a taxonomy of specialized boundary components (`ApiErrorBoundary`, `FeatureErrorBoundary`, etc.).
- Place Error Boundaries at isolation boundaries: points where an independent feature's failure is contained rather than propagating upward.
- Place Error Boundaries around third-party integration components (maps, charts, embeds).
- Place Error Boundaries around independently-loadable feature sections on a page.
- Do not wrap simple presentational components in Error Boundaries.
- Error Boundaries do not catch async errors in event handlers. Use try/catch in hooks for those cases (see the hook-as-adapter pattern in [hooks.md](hooks.md)).

</rules>

```typescript
import { ErrorBoundary } from "react-error-boundary";

// Correct: isolation boundary with contextual fallback
<ErrorBoundary fallback={<WidgetError />}>
  <DashboardWidget />
</ErrorBoundary>

// Correct: fallback as render function for retry capability
<ErrorBoundary fallbackRender={({ error, resetErrorBoundary }) => (
  <div>
    <p>Something went wrong.</p>
    <button onClick={resetErrorBoundary}>Retry</button>
  </div>
)}>
  <ThirdPartyMap />
</ErrorBoundary>

// Incorrect: wrapping a presentational component
<ErrorBoundary fallback={<p>Error</p>}>
  <Avatar name={user.name} />  // WRONG: Avatar cannot throw in a meaningful way
</ErrorBoundary>

// Incorrect: taxonomy of boundary components
export const ApiErrorBoundary: FC<Props> = ... // WRONG: use one ErrorBoundary with different fallbacks
```

## Suspense

<rules>

- Use `<Suspense>` boundaries at isolation boundaries for independently-loadable sections.
- Reach for `<Suspense>` when wrapping lazy-loaded components or handling async operations.
- Fallback UI is context-specific. Each boundary gets a fallback appropriate to the content it replaces (a skeleton matching the content shape when feasible, a simpler indicator when not). There is no single global spinner or skeleton component.

</rules>

## Pairing Error Boundaries with Suspense

A section that can fail independently can also load independently. Pair `<ErrorBoundary>` with `<Suspense>` at isolation boundaries. `ErrorBoundary` wraps the outside; `<Suspense>` wraps the inside.

<rules>

- When placing a manual `<Suspense>` boundary, wrap it in an `<ErrorBoundary>`.
- The `ErrorBoundary` catches errors. The `<Suspense>` handles loading. Both are colocated at the same isolation boundary.

</rules>

```typescript
// Correct: paired at an isolation boundary
<ErrorBoundary fallback={<WidgetError />}>
  <Suspense fallback={<WidgetSkeleton />}>
    <DashboardWidget />
  </Suspense>
</ErrorBoundary>

// Correct: multiple independent sections streaming in
<div className="dashboard">
  <ErrorBoundary fallback={<RevenueError />}>
    <Suspense fallback={<RevenueSkeleton />}>
      <RevenueChart />
    </Suspense>
  </ErrorBoundary>

  <ErrorBoundary fallback={<ActivityError />}>
    <Suspense fallback={<ActivitySkeleton />}>
      <ActivityFeed />
    </Suspense>
  </ErrorBoundary>
</div>

// Incorrect: Suspense without ErrorBoundary at an isolation boundary
<Suspense fallback={<WidgetSkeleton />}>
  <DashboardWidget />  // If this throws, nothing catches it at this level
</Suspense>
```
