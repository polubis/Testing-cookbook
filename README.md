## React testing library

### Avoid using container

```js
const { container } = render(<ChartWrapper isShadowAdded>Content</ChartWrapper>);

// Try to use screen[method] instead of container - but sometimes you can't
// With container usage we testing implementation details
expect(container.querySelector('.chart-shadow')).toBeInTheDocument();
```js

### Testing rendering

```js
const { rerender } = render(<ChartWrapper title="My chart">Content</ChartWrapper>);

expect(screen.getByText('My chart')).toBeInTheDocument();

rerender(<ChartWrapper title="">Content</ChartWrapper>);

// IMPORTANT: Use queryByText if you want test "not" cases.
expect(screen.queryByText('My chart')).not.toBeInTheDocument();
```

## Spying functions and mocking implementations

### Spy and mock function passsed as argument

```js
 const handleRetrySpy = jest.fn().mockImplementation(() => {});

render(
    <ChartWrapper error="Error" handleGetData={handleRetrySpy}>
        Content
    </ChartWrapper>
);

fireEvent.click(screen.getByText('Retry'));

expect(handleRetrySpy).toHaveBeenCalledTimes(1);
```

## Spying and mocking modules

### Mock module and mock implemenetation for all tests

```js
import { EventReportsRealm } from 'services/api/realms';

jest.mock('services/api/realms', () => ({
    EventReportsRealm: {
        GET: {
            dashboard: jest.fn().mockImplementation(() => {
                return new Promise(resolve => {
                    setTimeout(() => {
                        resolve();
                    }, 500);
                });
            })
        }
    }
}));
```

### Spy on fn call from module

```js
const dashboardSpy = jest.spyOn(EventReportsRealm.GET, 'dashboard');
// OR
// const dashboardSpy = jest.fn(EventReportsRealm.GET.dashboard);

waitFor(() => {
    expect(dashboardSpy).toHaveBeenCalledWith(0);
}).then(() => {});
```

### Spy and mock fn call from module

```js
jest.spyOn(EventReportsRealm.GET, 'dashboard').mockImplementation(() => {
    return Promise.reject(new Error('Error occured'));
});

waitFor(() => {
    expect(screen.getByText('Error occured')).toBeInTheDocument();
}).then(() => {});
```

## Clearing mocks

Always clean mocks after each test.

> In some cases, not using clearAllMocks may break other tests

```js
 afterEach(() => {
    jest.clearAllMocks();
});
```

## Test error throws

### Test error with builded in toThrow

```js
const _INVALID_VALUES_ = [null, undefined, '', NaN, 0, false];

_INVALID_VALUES_.forEach(value => {
    expect(() => AxiosFacade.onFailure(value)).toThrow(
        'Invalid error payload object in middleware'
    );
});
```
