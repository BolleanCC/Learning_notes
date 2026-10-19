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
#双指针，用于有序数组
#7.1对撞指针
def two_sum_sorted(nums, target):
    l, r = 0, len(nums) - 1
    while l < r:
        s = nums[l] + nums[r]
        if s == target:
            return[l,r]
        elif s < target:
            l+=1
        else:
            r-=1
    return []
