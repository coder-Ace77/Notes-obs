---
tags: [dsa, guide, geometry, convex-hull, cross-product]
chapter: 21
sheet-section: U
---

# Chapter 21 · Geometry (OA-lite)

> **Read this before you start the problems.** Each idea is introduced with a small example, so no prior familiarity with the problems is assumed.

Back to [[00 Guide Index]] · Sheet section **U** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

Geometry appears rarely in this kind of assessment, and when it does, it draws from a genuinely small set of primitives. The difficulty is not conceptual depth so much as precision: geometric code that relies on floating-point comparisons tends to fail on inputs that are almost, but not quite, degenerate, in ways that are extremely difficult to debug afterwards, since the failure is a matter of a comparison being off by a tiny fractional amount rather than an obviously wrong result. The practical response is to do essentially everything in integer arithmetic, comparing signs of expressions rather than magnitudes computed through division, and reserving floating-point work only for the small number of places where it is genuinely unavoidable.

It is also worth noting before starting that a fair number of problems that sound geometric on this sheet are not actually testing geometry at all. Several of the "hard" geometry-adjacent problems elsewhere on the sheet are sweeps, union-find applications, or segment tree problems that merely happen to be set in a plane, and the geometric flavour of the statement is scenery rather than substance.

---

## What these problems look like

The recurring questions are: whether one point lies to the left or right of a line through two others, or whether three points are collinear; whether two line segments intersect; the convex hull of a set of points, or occasionally the smallest shape enclosing them in some other sense; the closest pair among a set of points; whether a point lies inside a polygon; and the area of a polygon.

---

## Part 0 · Working in integers

Before any of the specific techniques, one general habit is worth adopting whenever the input coordinates are integers: avoid division, avoid square roots when only a comparison is needed, and avoid testing floating-point values for exact equality to zero.

Comparisons between quantities that would otherwise require dividing can almost always be restated by cross-multiplying instead, and distances can almost always be compared as their squares rather than requiring an actual square root. Coordinates as large as a billion produce cross products as large as roughly four times ten to the eighteenth power, which sits right at the edge of what a standard 64-bit integer can hold, so a computation involving the product of three such coordinates, rather than two, generally needs an extended-precision integer type.

---

## Part 1 · The cross product

This single computation underlies almost everything else in the chapter.

$$\text{cross}(O, A, B) = (A_x - O_x)(B_y - O_y) - (A_y - O_y)(B_x - O_x)$$

```cpp
using ll = long long;
struct P { ll x, y; };
ll cross(P o, P a, P b) {
    return (a.x - o.x) * (b.y - o.y) - (a.y - o.y) * (b.x - o.x);
}
```

**The sign of this expression tells you the orientation of the turn from `O` to `A` to `B`.** A positive value means the turn goes counter-clockwise, which is equivalent to saying `B` lies to the left of the ray from `O` through `A`. A negative value means the turn goes clockwise, meaning `B` lies to the right. A value of exactly zero means the three points are collinear.

**The magnitude of this same expression is twice the area of the triangle formed by the three points**, which is what makes the cross product useful for area calculations as well as for orientation.

CSES *Point Location Test* is a direct request for this function, and is worth writing without needing to think, since it recurs throughout the rest of the chapter.

**Everything else the cross product provides, listed together so the connections are visible:** the area of a polygon, via the shoelace formula, which sums `cross(origin, p_i, p_{i+1})` over consecutive vertex pairs and takes half the absolute value of the total; testing whether a point lies inside a convex polygon, by checking that it lies on the same side of every edge; the convex hull construction in the next part; and sorting a set of points by angle around a centre, which should always be done by first comparing which half-plane each point falls into and then comparing cross products, rather than by computing an angle directly with an inverse trigonometric function, since the latter introduces floating-point error where none is necessary.

---

## Part 2 · Segment intersection

Two segments, one from `A` to `B` and the other from `C` to `D`, properly cross each other exactly when `C` and `D` fall on opposite sides of the line through `A` and `B`, and simultaneously `A` and `B` fall on opposite sides of the line through `C` and `D`:

```cpp
int sgn(ll x) { return (x > 0) - (x < 0); }

bool intersect(P a, P b, P c, P d) {
    int d1 = sgn(cross(a, b, c)), d2 = sgn(cross(a, b, d));
    int d3 = sgn(cross(c, d, a)), d4 = sgn(cross(c, d, b));
    if (d1 * d2 < 0 && d3 * d4 < 0) return true;         // a proper crossing
    // collinear or touching cases:
    if (d1 == 0 && onSegment(a, b, c)) return true;
    if (d2 == 0 && onSegment(a, b, d)) return true;
    if (d3 == 0 && onSegment(c, d, a)) return true;
    if (d4 == 0 && onSegment(c, d, b)) return true;
    return false;
}
bool onSegment(P a, P b, P p) {                          // assumes p is already collinear with a and b
    return min(a.x, b.x) <= p.x && p.x <= max(a.x, b.x)
        && min(a.y, b.y) <= p.y && p.y <= max(a.y, b.y);
}
```

**The proper-crossing case, the first `if` statement, is the part almost everyone gets right immediately.** The four collinear-or-touching cases beneath it are where the actual difficulty of this function lives, covering situations where one segment's endpoint lands exactly on the other segment, or where the two segments overlap along a shared line. Writing out all four cases explicitly, rather than trying to find a shorter equivalent formulation, is the more reliable approach.

CSES *Line Segment Intersection* is a direct request for this function.

---

## Part 3 · Convex hull

The following implementation, known as Andrew's monotone chain, builds the convex hull by sorting the points and then sweeping through them twice.

```cpp
vector<P> hull(vector<P> p) {
    sort(p.begin(), p.end(), [](P a, P b){
        return a.x != b.x ? a.x < b.x : a.y < b.y;
    });
    p.erase(unique(p.begin(), p.end(), [](P a, P b){ return a.x==b.x && a.y==b.y; }), p.end());
    int n = p.size();
    if (n < 3) return p;
    vector<P> h(2 * n);
    int k = 0;
    for (int i = 0; i < n; i++) {                            // building the lower hull
        while (k >= 2 && cross(h[k-2], h[k-1], p[i]) <= 0) k--;
        h[k++] = p[i];
    }
    for (int i = n - 2, t = k + 1; i >= 0; i--) {            // building the upper hull
        while (k >= t && cross(h[k-2], h[k-1], p[i]) <= 0) k--;
        h[k++] = p[i];
    }
    h.resize(k - 1);
    return h;
}
```

**Reading the algorithm in words:** sort the points from left to right, then sweep through them once building the lower boundary of the hull, discarding the most recently added point whenever the last three points on the boundary make a turn that is not to the left, and then sweep back through them in reverse to build the upper boundary the same way. **This is structurally identical to the monotonic stack from chapter [[07 Monotonic Stacks and Deques]], with the cross product taking the place of the ordinary comparison used there**, and it shares the same linear-time argument, since each point is pushed onto the hull once and popped at most once.

**The choice between `<= 0` and `< 0` in the discard condition controls whether points lying exactly on the boundary of the hull, rather than at one of its corners, are kept or discarded.** Using `<= 0` produces the smallest possible hull, discarding such boundary points, while using `< 0` retains them. **This choice needs to match what the specific problem is asking for**, and reading the statement carefully for whether boundary points should be included is the single most common source of error in convex hull problems; CSES *Convex Hull* specifically wants such points retained, which means `< 0` is the version needed there.

---

## Part 4 · Closest pair of points

The textbook approach to this problem is a divide-and-conquer algorithm running in time proportional to the number of points times its own logarithm, splitting the points by an x-coordinate, recursing on each half, and then checking a narrow strip around the dividing line for pairs closer together than the best distance found so far, where checking each point in that strip only ever needs to compare against a small constant number of neighbouring points once sorted by y-coordinate.

**A sweep-line alternative achieves the same time complexity with considerably simpler code, and is the version worth actually using:**

```cpp
sort(p.begin(), p.end(), byX);
set<pair<ll,ll>> s;                     // the active points, ordered by (y, x)
ll best = LLONG_MAX;
int left = 0;
for (int i = 0; i < n; i++) {
    ll d = (ll)ceil(sqrt((long double)best));
    while (p[left].x < p[i].x - d) { s.erase({p[left].y, p[left].x}); left++; }
    auto lo = s.lower_bound({p[i].y - d, LLONG_MIN});
    auto hi = s.upper_bound({p[i].y + d, LLONG_MAX});
    for (auto it = lo; it != hi; ++it)
        best = min(best, dist2(p[i], {it->second, it->first}));
    s.insert({p[i].y, p[i].x});
}
```

**Why this remains fast**: the set of points currently active, meaning within horizontal distance `d` of the current point, together with the vertical band of height `2d` searched around the current point's y-coordinate, can only ever contain a small constant number of points at once, since any two points within that band and that close horizontally would already be closer together than the current best distance, which would have already reduced the search region further.

Keeping `best` as a squared distance throughout, and only taking a square root when computing the width `d` of the search window, keeps almost the entire computation in integers, with the one unavoidable use of a square root rounded generously upward so that no genuinely closer pair is missed.

CSES *Minimum Euclidean Distance* is a direct request for this computation.

---

## The ideas worth carrying forward

1. **Work in integers wherever the input allows it.** Avoid division, avoid square roots except where a width or an actual distance is genuinely required, and avoid testing floating-point values for exact equality to zero.

2. **The cross product is the foundation of the entire chapter.** Its sign gives orientation and collinearity, and its magnitude gives twice a triangle's area.

3. **Sort points by angle using half-plane comparison followed by the cross product**, never with an inverse trigonometric function, which introduces unnecessary floating-point error.

4. **The proper-crossing case in segment intersection is easy; the collinear and touching cases are where the real difficulty lies.** Writing all four out explicitly is more reliable than looking for a shortcut.

5. **Convex hull construction is a monotonic stack with the cross product as its comparison**, sharing the same linear-time argument as the stacks in chapter [[07 Monotonic Stacks and Deques]].

6. **The choice between `<= 0` and `< 0` in the hull construction decides whether boundary points survive**, and this has to match what the specific problem asks for rather than being chosen by habit.

7. **The sweep-line approach to closest pair is considerably simpler to write correctly than the classical divide-and-conquer version**, and achieves the same complexity because the active search region can only ever hold a small constant number of candidates.

8. **Coordinates as large as a billion push cross products close to the limit of a 64-bit integer**, and a computation multiplying three such coordinates together generally needs extended precision.

9. **Many sweep-line and structural problems on this sheet look geometric and are not.** Checking chapter [[02 Intervals and Sweep Line]] before reaching for geometric machinery is often the faster route.

---

## Where people lose these problems

**Using floating-point arithmetic where integer arithmetic was available.** This is the single largest source of wrong answers in this chapter.

**Sorting points by angle with an inverse trigonometric function.** Slow, and a source of precision error that half-plane comparison combined with the cross product avoids entirely.

**Overflowing the cross product**, particularly when a computation multiplies three coordinates together rather than two.

**Missing one of the four collinear or touching cases in segment intersection.**

**Choosing the wrong strictness in the convex hull construction**, producing a hull that is technically correct in shape but does not match what the problem asked for regarding boundary points.

**Failing to remove duplicate points before constructing a convex hull.** Duplicates produce degenerate cross products partway through the construction and can cause certain implementations to loop indefinitely.

**Forgetting to handle fewer than three points as a special case in the hull construction.**

**Comparing distances using an actual square root rather than comparing their squares directly.**

---

## Working through the problem list

This chapter has four problems on the sheet, which is proportionate to how rarely geometry of this kind actually appears in assessments.

- **CSES Point Location Test** — *given three points, determine the orientation of the turn between them.* The cross product itself. A ten-minute problem, and worth saving the function afterwards.
- **CSES Line Segment Intersection** — *determine whether two given segments intersect.* All four collinear and touching cases need genuine care here.
- **CSES Convex Hull** — *compute the convex hull of a set of points.* The monotone chain, noting that CSES specifically wants boundary points included.
- **CSES Minimum Euclidean Distance** — *find the closest pair among a set of points.* The sweep-line approach.

**A few additional primitives are worth knowing even though they are not on the sheet**, in case a slightly wider assessment expects them: the shoelace formula for polygon area, testing whether a point lies inside a polygon using ray casting, and rotating calipers for finding the diameter of a point set. Each is short, on the order of ten lines, and each closes a small gap that occasionally appears.

---

**Completion within a single sitting, with all four functions saved in a personal template file, is a more meaningful goal here than a numerical accuracy target.**

Spending substantially more time than that on geometry specifically is unlikely to be a good use of preparation time relative to spending the same time in chapter [[08 Segment Trees]] or elsewhere, given how infrequently this chapter's material actually appears.

---

## Check yourself

1. Write the cross product. What does its sign tell you, and what does its magnitude tell you?
2. Name three things the cross product computes, beyond orientation.
3. Why is an inverse trigonometric function avoided when sorting points by angle, and what replaces it?
4. Beyond a proper crossing, what four additional cases must segment intersection handle?
5. What does the choice between `<= 0` and `< 0` control in the monotone chain, and how should that choice be decided?
6. Why is convex hull construction structurally the same as a monotonic stack?
7. In the closest-pair sweep, why can the active search region only ever contain a small constant number of candidate points?
8. Coordinates reach up to a billion. How large can a cross product become, and does it fit in a standard 64-bit integer?
9. Name three problems elsewhere on this sheet that sound geometric but are not.
