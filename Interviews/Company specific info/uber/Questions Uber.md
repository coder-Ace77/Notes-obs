
---

### DSA

- [x] Count distinct navigation subseq - https://www.codinzhub.com/question/count-distinct-navigation-subsequences
- [x] Ride req - connected componenet - DSU - https://www.codinzhub.com/question/ride-request-queue-insertion-365
- [x]  Ride Request Queue Insertion  - Array - https://www.codinzhub.com/question/ride-request-queue-insertion
- [x] ### Earliest Reachable Intersection Before Closure - https://www.codinzhub.com/question/earliest-reachable-intersection-before-closure
- [x] Maximum Riders During Round Trip https://www.codinzhub.com/question/maximum-riders-during-round-trip
- [x] Minimum Travel Time with Mandatory Stops https://www.codinzhub.com/question/minimum-travel-time-with-mandatory-stops-545
- [x] Driver Hub Placement Optimization
- [x] Driver Console Autocomplete https://www.codinzhub.com/question/driver-console-autocomplete
- [x] Complete Signal Block Detection https://www.codinzhub.com/question/complete-signal-block-detection-239
- [x] Receipt optimization https://www.codinzhub.com/question/receipt-printing-optimization-617
- [x] Uber Green Zone Expansion https://www.codinzhub.com/question/uber-green-zone-expansion
- [x] ### Maximize Equal Portion Size https://www.codinzhub.com/question/maximize-equal-portion-size
- [x] https://leetcode.com/problems/minimum-edge-reversals-so-every-node-is-reachable/description/?envType=company&envId=microsoft&favoriteSlug=microsoft-six-months
- [ ] https://leetcode.com/problems/minimum-height-trees/description/
- [x] https://leetcode.com/problems/cheapest-flights-within-k-stops/description/
- [x] https://leetcode.com/problems/maximum-points-you-can-obtain-from-cards/description/
- [ ] Given two strings, s and t, consisting only of lowercase English characters. Determine the minimum number of times s must be appended to itself for t to be a subsequence of the resulting string.
- [ ] Given a n*m 2-D matrix with arrows in different directions, find out the minimum cost to travel from top-left grid to bottom-right grid if the cost to travel in the direction of the grid arrow costs you nothing but if you want to travel in any other direction, it would cost you 1 quantity.
- [ ] https://leetcode.com/problems/find-the-closest-palindrome/description/
- [ ] https://leetcode.com/problems/longest-continuous-subarray-with-absolute-diff-less-than-or-equal-to-limit/




---

### Coding problems

1 - Given a Terrain (2D - land and water) implement -
- addLand(x, y): sets (x, y) to land
- isLand(x, y): true if (x, y) is a land, false otherwise (water)
- countIslands(): returns the number of unique islands

Sol - Use union find to handle the island brdige case when land is added. 

2 - Problem:  
Implement file system APIs:

1. mkdir
2. pwd
3. cd
4. ls
5. addfileContent(path,data)
6. getFilecontent(path)

sol - By using hierarchical trie like data structure we can efficiently solve the problem. Node will hold childrens , parent pointers type of it and finally name of node. 

To handle  pwd and cd we maintain current dir pointer and upd it when cd is called. Finally to get path we traverse till root. 

`cd ` can also take `*` which is wildcard.

3 - Design and implement a high-performance **Frequency Counter** where each recorded element has a fixed **Time-To-Live (TTL)**. An element should contribute to the total and individual counts only if it was added within the last t seconds.

- put_element(key) - Adds an instance of `key` to the counter. The timestamp of the call marks the beginning of its t-second life.
- **`get_element_count(K key)`**: Returns the current number of non-expired instances of the specific `key`.
- **`get_total_elements()`**: Returns the total count of all non-expired elements across all keys currently in the system.

**Time Complexity:** * `put_element` should ideally be O(1).
- `get` methods should be highly optimized (avoid O(N) scans of all historical data).

Solution - Simplest idea is to use a queue and an hashmap to do this. Since time has to increase eveytime we can just use the queue to remove all the earlier entries and move forward. Then all the three operations are straigt forward. To get total elements we can just maintain a counter. 

However in the multithreaded environment we should be using lock which can be simple lock we need to do before each read and write request. The reason is that each read and write request is simply write first due to need to remove ttl. And also since removing stale entries req us to deal with three operations which need to be grouoped and made atomic. 

```java
private void remove_stale_entries(int t){
	while(!queue.isEmpty() && queue.peek()[1]<t){
		int key = queue.peek()[0];
		queue.poll();
		counts.put(key,counts.get(key)-1);
		if(counts.get(key)==0){
			counts.remove(key);
		}
		total-=1;
	}
}
```

Another method is to use concurrent datastructure however have a locked background scheduler running after 1s to evict keys. However we will be req to lock for each `put` as it is aggregated of three steps-

```
put in queue
put in hashmap 
update counter
```

Other than that read shoulds be very fast. 

However the production grade implementation is to use a ring of buffers. 

```java
package counter;

import java.util.HashMap;
import java.util.Map;

public class BucketCounter {
    // A bucket holds the aggregated data for a single second in time
    private static class Bucket {
        long timestamp = -1;
        int totalCount = 0;
        Map<Integer, Integer> keyCounts = new HashMap<>();

        public void reset(long newTimestamp) {
            this.timestamp = newTimestamp;
            this.totalCount = 0;
            this.keyCounts.clear();
        }
    }

    private final Bucket[] ring;
    private final int ttl;

    public BucketCounter(int ttl) {
        this.ttl = ttl;
        this.ring = new Bucket[ttl];
        for (int i = 0; i < ttl; i++) {
            ring[i] = new Bucket();
        }
    }

    public void put_element(int key) {
        long currentTime = System.currentTimeMillis() / 1000;
        int index = (int) (currentTime % ttl);
        Bucket bucket = ring[index];

        // If the bucket's timestamp is old, it means we wrapped around. Overwrite it!
        if (bucket.timestamp != currentTime) {
            bucket.reset(currentTime);
        }

        bucket.keyCounts.put(key, bucket.keyCounts.getOrDefault(key, 0) + 1);
        bucket.totalCount++;
    }

    public int get_element_count(int key) {
        long currentTime = System.currentTimeMillis() / 1000;
        int count = 0;

        // Sum up all valid buckets
        for (Bucket bucket : ring) {
            if (currentTime - bucket.timestamp < ttl) {
                count += bucket.keyCounts.getOrDefault(key, 0);
            }
        }
        return count;
    }

    public int get_total_count() {
        long currentTime = System.currentTimeMillis() / 1000;
        int total = 0;

        // Sum up all valid buckets
        for (Bucket bucket : ring) {
            if (currentTime - bucket.timestamp < ttl) {
                total += bucket.totalCount;
            }
        }
        return total;
    }
}
```

Well this is ideal only for use case of millions of operations per second. If ttl is large and we have less calls per second then we should stick to queue version instead. 
In case the ttl can vary we can use priority queue with expiration. 

4 - We were given logs in the format `UserA-Shared-UserB-timestamp`, meaning UserA and UserB share a cab.
- **Version 1:** For each log, return how many cabs are currently running.
- **Version 2:** For a given logs , return the timestamp when all users get into one cab.

Solution - Classic DSU


---

### LLD questions

1 - Design a **Train-Platform Management System** with functionalities:
    - Assign trains to platforms based on input.
    - Query which train is at a given platform at a specific time.
    - Query which platform a train is at, at a specific time.
- **Expectations:**
    - Write runnable code (in my preferred language – chose **Java**).
    - Explain design patterns used.
    - Walkthrough every step while coding.

Solution two hashmaps. 



---

### HLD questions

**Problem Statement: Real-Time & Historical Geospatial Heatmap Service**

Design a specialized sub-system for Uber's internal analytics team that visualizes driver density across a city via a heatmap. The system must transform raw GPS pings into meaningful geospatial aggregates to support both immediate operational monitoring and retrospective trend analysis.

**Functional Requirements**
- **Data Ingestion:** Process a massive stream of incoming GPS coordinates (latitude, longitude) from the driver app.
- **Geospatial Mapping:** Convert raw coordinates into indexed geographic "cells" (e.g., Geohashes or H3 indices) at a configurable precision.
- **Dual-Window Analytics:**
    - **Near Real-Time (NRT):** Provide an aggregate heatmap for the **past 20 minutes**. This view must be updated continuously.
    - **Historical:** Provide a historical heatmap for the **past 24 hours**, with data bucketized into 1-hour intervals.
- **Query Interface:** Provide a way for an internal dashboard to fetch density counts for a specific city or region for both NRT and historical timeframes.

**Non-Functional Requirements**
- **Extreme Throughput:** The system must handle a peak load of **500,000 Transactions Per Second (TPS)** without dropping events.
- **Low Latency (NRT):** The 20-minute window should reflect the current state of the city with sub-minute lag.
- **Scalability:** The architecture must be horizontally scalable to support multiple cities and growing driver counts.
- **Fault Tolerance:** In the event of a processing node failure, the system must use checkpoints to ensure counts remain accurate and "exactly once" or "at least once" semantics are maintained.
- **Storage Efficiency:** Optimize storage to handle high-frequency writes for the NRT use case and structured, queryable storage for historical data.
- **Input Stream:** ≈500,000 events/sec.




### Twitter 

Design Twitter like system. Implement below functionality -
1. Post a tweet
2. Like a tweet
3. Dislike a tweet
4. Get Top liked tweets for a user
5. Follow a user
6. Unfollow a user
7. Generate news feed for a user ( consisting of user’s tweets + followee’s tweets in reverse time order)

#### **Problem Statement: Popularity-Driven Product Discovery Engine**

Design the backend for an e-commerce browsing platform that identifies and highlights "Trending" or "Top-K" merchandise in real-time. The system must process high-velocity user engagement signals to dynamically update product rankings without impacting the core browsing experience.

### Uber eats

Design the backend architecture for the Uber Eats homepage and restaurant discovery service. The system must provide a highly personalized, location-aware browsing experience while simultaneously tracking complex business metrics for operational insights.

**Functional Requirements**

- **Location-Based Discovery:** Users must see a curated list of nearby restaurants based on their current GPS coordinates or a provided address.    
- **Search Functionality:** Support searching for specific restaurants or dishes with low latency.
- **Homepage Composition:** Serve a dynamic homepage featuring:
    - "Restaurants Near You" (Geospatial search).
    - "Top Rated" or "Trending" (Aggregated data).
    - Active order status (Real-time tracking).
- **Ordering Flow:** Manage the lifecycle of an order, including linking users to specific dishes and restaurants.
- **Analytics & Metrics:** Capture and aggregate real-time data to identify:
    - The most ordered restaurant and dish in a specific region.
    - Total orders processed per restaurant for payout and ranking purposes.

**Non-Functional Requirements**

- **Low Latency:** The homepage must load in under 200ms to prevent user bounce.
- **High Availability:** The discovery service must be highly available; a failure in search should not bring down the entire app.
- **Scalability:** The system must handle massive spikes during peak meal times (lunch/dinner).
- **Data Consistency:** * **Eventual Consistency** is acceptable for restaurant ratings and "top-ordered" lists.
    - **Strong Consistency** is required for order placement and payment status.

**Technical Focus Areas**

1. **Geospatial Indexing:** Justify the use of **GeoHashing** (e.g., Redis Geo, S2, or H3) to partition restaurant data by location.
2. **Schema Design:** Define the relationship between `Restaurants`, `Dishes`, and `Orders`.
    - _Constraint:_ Determine if a single Database (SQL) or a Polyglot approach (SQL for Orders, NoSQL for Menu/Catalog) is superior.
3. **API Design:** Propose REST or GraphQL endpoints for `GET /homepage` and `GET /search`.
4. **Caching Strategy:** Implement a multi-level cache (CDN for static assets, Redis for popular menus and session data) to minimize DB hits.


