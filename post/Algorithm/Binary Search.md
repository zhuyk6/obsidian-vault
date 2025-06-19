二分查找，广义来说，在一个区间`X = [left, right]`上，给定一个 `f: Fn(int) -> bool`， 如果 `f(X)` 是单调的，有如下两种 cases：
- Case 1：`[false, ..., false, true, ..., true]` 
- Case 2：`[true, ..., true, false, ..., false]`

那么我们可以在 $\log_2 (N)$ 的时间找到分界点：不妨定义为第一个为`true` 的下标（case 1）或者最后一个为 `true` 的下标（case 2）

我们把 case 1 的算法叫作 `lower_bound`，case 2 的叫作 `upper_bound`。

# `lower_bound`

`lower_bound` 处理 Case 1：`[false, ..., false, true, ..., true]`，要找到第一个 `true`。

很容易写出类似的代码：
```rust
pub fn lower_bound(left: i32, right: i32, check: impl Fn(i32) -> bool) -> i32 {
    let mut l = left;
    let mut r = right;
    while l < r {
        let mid = (l + r) / 2;
        if check(mid) {
            r = mid;
        } else {
            l = mid + 1;
        }
    }
    l
}
```

基本思路确实如此，但是有一些边界问题需要仔细思考，具体来说：
- 闭区间 `[l, r]` 如果没有 `f(i) = true` 怎么办？
- 怎么更新 `l, r`？
- 怎么确定 `mid` ？

第一个最好回答，对于 case 1，我们只需要检查 `f(right)`。

对于第二个问题，我们需要确定「循环不变量」，因为排除了「无解」的情况，**我们可以令「闭区间`[l, r]`始终包含解」作为循环不变量**，因此 `l == r` 时退出循环（找到解了）。
怎么更新 `l, r` 呢？
- `f(mid) = true`，解在 `[l, mid]` 中，令 `r = mid`
- `f(mid) = false`，解在 `[mid + 1, r]` 中，令 `l = mid + 1`

对于第三个问题，看上去我们只需要令 `mid` 为 `l + r / 2` 即可。但是整数除法是要取整的，这里需要认真思考。
问题的关键在于我们要避免「死循环」：
- `f(mid) = true`，`[l, r] -> [l, mid]`，当 `mid = r` 就会死循环，比如 `l = -2, r = -1, mid = (-2 + -1) / 2 = -1`；
- `f(mid) = false`， `[l, r] -> [mid + 1, r]`，只要 $\text{mid} \in [l, r]$ ，区间就一定变小。
所以我们发现，只需要 `mid = (l + r).div_floor(2)` 即可，数学上就是 $\lfloor \frac{l + r}{2} \rfloor$ 。
需要特别注意的是，几乎所有编程语言中的默认整数除法不是「向下取整」，而是「向0取整」，严格来说我们需要 `(l + r - 1) / 2`。

修改后的代码如下：
```rust
/// If the range `[left, right]` is `[false, ..., false, true, ..., true]` under `f`,
/// this method returns the first index `i` such that `f(i)` is true.
///
/// If `f(right)` is false, it returns `None`.
pub fn lower_bound(left: i32, right: i32, check: impl Fn(i32) -> bool) -> Option<i32> {
    if !check(right) {
        return None;
    }
    let mut l = left;
    let mut r = right;
    while l < r {
        let mid = (l + r - 1) / 2; // (l + r).div_floor(2)
        if check(mid) {
            r = mid;
        } else {
            l = mid + 1;
        }
    }
    Some(l)
}
```

# `upper_bound`

`upper_bound` 处理 Case 2：`[true, ..., true, false, ..., false]`，要找到最后一个 `true`。

类似的，我们考虑两种情况：
- `f(mid) = true`，解在`[mid, r]`中， `[l, r] -> [mid, r]`；
- `f(mid) = false`，解在 `[l, mid - 1]` 中，`[l, r] -> [l, mid - 1]`。
为了不死循环，我们需要$\text{mid} \in [l, r], \text{mid} > l$，只需要 $\text{mid} = \lceil \frac{l + r}{2} \rceil$，代码实现就是 `mid = (l + r + 1) / 2`。

```rust
/// If the range `[left, right]` is `[true, ..., true, false, ..., false]` under `f`,
/// this method returns the last index `i` such that `f(i)` is true.
///
/// If `f(left)` is false, it returns `None`.
pub fn upper_bound(left: i32, right: i32, check: impl Fn(i32) -> bool) -> Option<i32> {
    if !check(left) {
        return None;
    }

    let mut l = left;
    let mut r = right;
    while l < r {
        let mid = (l + r + 1) / 2; // (l + r).div_ceil(2)
        if check(mid) {
            l = mid;
        } else {
            r = mid - 1;
        }
    }
    Some(l)
}
```

