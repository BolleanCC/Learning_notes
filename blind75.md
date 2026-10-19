#6.3模板二：收缩到边界（找最小值时常用）
def lower_bound(nums, target):

    lo, hi = 0, len(nums) - 1

    while lo < hi:

        mid = (lo + hi) // 2

        if nums[mid] >= target:
            hi = mid

        else:
            lo = mid + 1

    return lo
