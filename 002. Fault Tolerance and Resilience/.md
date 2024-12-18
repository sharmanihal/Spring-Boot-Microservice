# Circuit Breaker Parameters and Configuration

## Key Parameters for Circuit Breakers

### 1. Last N Requests to Consider
When a failure occurs, the circuit breaker evaluates the last **N** requests to determine the failure rate. It does not act on a single failure to avoid reacting to isolated issues.

- **Example**: Evaluate the last 5 requests to decide if the circuit should break.
- **Sliding Window Types in Resilience4j**:
  - **Time-Based Sliding Window**: Considers all requests made in the last specified duration (e.g., 10 seconds).
  - **Count-Based Sliding Window**: Observes the outcomes of the last specified number of requests (e.g., 10 requests).

### 2. Failure Threshold
Defines the percentage of failed requests within the last **N** requests that triggers the circuit to break.

- **Example**: If 3 out of the last 5 requests fail, trip the circuit.

### 3. Timeout Duration
Specifies the maximum time to wait for a response before marking a request as failed.

- **Example**: If a request takes more than 2 seconds, consider it a failure.
- Can be set using `RestTemplate` or `WebClient`.

### 4. Sleep Window
Defines the duration the circuit breaker waits before trying to send requests again after being tripped.

- **Example**: Wait for 10 seconds before checking if the failing service has recovered.

---

## Configuring Circuit Breaker with Annotations
Annotate the method to be protected with the circuit breaker:
```java
@CircuitBreaker(name = "movie-catalog-api", fallbackMethod = "getCatalogFallback")
```

---

![image](https://github.com/user-attachments/assets/db36eb06-db14-4ef5-8dfd-80133ab50316)

## Configuration Properties : resilience4j.circuitbreaker.instances:movie-catalog-api:

| **Config Property**                | **Default Value** | **Description** |
|------------------------------------|-------------------|-----------------|
| **failureRateThreshold**           | 50                | Configures the failure rate threshold (percentage). When the failure rate equals or exceeds the threshold, the CircuitBreaker transitions to **OPEN** and starts short-circuiting calls. |
| **slowCallRateThreshold**          | 100               | Configures a threshold (percentage). Calls exceeding `slowCallDurationThreshold` are considered slow. When the percentage of slow calls equals or exceeds the threshold, the CircuitBreaker transitions to **OPEN**. |
| **slowCallDurationThreshold**      | 60000 ms          | Configures the duration (in ms) above which calls are considered slow. |
| **permittedNumberOfCallsInHalfOpenState** | 10                | Configures the number of calls permitted in the **HALF-OPEN** state. |
| **maxWaitDurationInHalfOpenState** | 0 ms              | Configures the maximum wait duration in **HALF-OPEN** state before transitioning back to **OPEN**. Value `0` means it waits indefinitely until all permitted calls complete. |
| **slidingWindowType**              | COUNT_BASED       | Configures the sliding window type:  - **COUNT_BASED**: Records the last `slidingWindowSize` calls. - **TIME_BASED**: Observes calls made in the last `slidingWindowSize` seconds. |
| **slidingWindowSize**              | 100               | Configures the size of the sliding window for recording outcomes in the **CLOSED** state. |
| **minimumNumberOfCalls**           | 100               | Sets the minimum number of calls required per sliding window period for calculating failure or slow call rates. |
| **waitDurationInOpenState**        | 60000 ms          | Specifies the duration the CircuitBreaker stays in the **OPEN** state before transitioning to **HALF-OPEN**. |
| **automaticTransitionFromOpenToHalfOpenEnabled** | false             | If `true`, automatically transitions from **OPEN** to **HALF-OPEN** after `waitDurationInOpenState`. Otherwise, requires a call to trigger the transition. |
| **recordExceptions**               | empty             | Lists exceptions that are recorded as failures and increase the failure rate. |
| **ignoreExceptions**               | empty             | Lists exceptions that are ignored and do not affect failure or success rates. |
| **recordFailurePredicate**         | `throwable -> true` | Custom predicate to evaluate if an exception should be recorded as a failure. |
| **ignoreExceptionPredicate**       | `throwable -> false` | Custom predicate to evaluate if an exception should be ignored. |

---

## Example Usage
### Annotated Method:
```java
@CircuitBreaker(name = "movie-catalog-api", fallbackMethod = "getCatalogFallback")
public String getCatalog() {
    // Method logic
}

public String getCatalogFallback(Throwable throwable) {
    return "Fallback response";
}
