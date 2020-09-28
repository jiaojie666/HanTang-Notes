<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

1.	**寻找两个正序数组的中位数**



		double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2)
		    int sum=nums1.size()+nums2.size();
		    return sum%2==0?(getKthValue(nums1,nums2,sum/2)+getKthValue(nums1,nums2,sum/2+1))/2:getKthValue(nums1,nums2,sum/2+1);
		double getKthValue(vector<int> &nums1,vector<int>&nums2,int k)
		    int len1=nums1.size(),len2=nums2.size();
		    int i1=0,i2=0;
		    while(i1<=len1||i2<=len2)
			if(i1==len1)
			    return nums2[i2+k-1];
			if(i2==len2)
			    return nums1[i1+k-1];
			if(k==1)
			    return min(nums1[i1],nums2[i2]);
			int half=k/2;
			int new_i1=min(i1+half,len1)-1;
			int new_i2=min(i2+half,len2)-1;
			if(nums1[new_i1]<nums2[new_i2])
			    k-=(new_i1-i1+1);
			    i1=new_i1+1;
			else
			    k-=(new_i2-i2+1);
			    i2=new_i2+1;
		return 0;
2.	**搜索旋转排序数组**



		int search(vector<int>& nums, int target)
		    int left=0;
		    int right=nums.size()-1;
		    while(left<=right)
			int mid=left+(right-left)/2;
			if(nums[mid]==target)
			    return mid;
			else if(nums[0]<=nums[mid])
			    if(target>=nums[0]&&target<=nums[mid])
				right=mid-1;
			    else
				left=mid+1;
			else
			    if(target>nums[mid]&&target<=nums[right])
				left=mid+1;
			    else
				right=mid-1;
3.	**排序数组查找元素第一次出现和最后一次出现的位置**



		int extremeInsertionIndex(int[] nums, int target, boolean left) {
		    int lo = 0;
		    int hi = nums.length;
		    while (lo < hi) 
			int mid = (lo + hi) / 2;
			if (nums[mid] > target || (left && target == nums[mid])) 
			    hi = mid;
			else 
			    lo = mid+1;
		    return lo;
		int[] searchRange(int[] nums, int target) {
		    int[] targetRange = {-1, -1};
		    int leftIdx = extremeInsertionIndex(nums, target, true);
		    if (leftIdx == nums.length || nums[leftIdx] != target) {
			return targetRange;
		    targetRange[0] = leftIdx;
		    targetRange[1] = extremeInsertionIndex(nums, target, false)-1;
4.	**旋转排序数组最小值**



		int findMin(vector<int>& nums)
		    int left=0;
		    int right=nums.size()-1;
		    while(left<right)
			int mid=left+(right-left)/2;
			if(nums[mid]>nums[right])
			    left=mid+1;
			else if(nums[mid]<nums[right])
			    right=mid;
			else
			    right--;
		return nums[left];
5.	**寻找峰值**



		public int findPeakElement(int[] nums)
		    int l = 0, r = nums.length - 1;
		    while (l < r) {
			int mid = (l + r) / 2;
			if (nums[mid] > nums[mid + 1])
			    r = mid;
			else
			    l = mid + 1;
		return l;
6.	**有序矩阵第K小的元素**



		int kthSmallest(vector<vector<int>>& matrix, int k)
		    int m=matrix.size();
		    int n=matrix[0].size();
		    int left=matrix[0][0];
		    int right=matrix[m-1][n-1];
		    while(left<right)
			int mid=left+(right-left)/2;
			if(check(matrix,k,mid))
			    right=mid;
			else
			    left=mid+1;
		    return left;
		int check(vector<vector<int>>& matrix, int k,int mid)
		    int i=matrix.size()-1;
		    int j=0;
		    int res=0;
		    while(i>=0&&j<matrix[0].size())
			if(matrix[i][j]<=mid)
			    res+=(i+1);j++;
			else
			    i--;
		return res>=k;
7.	**有序数组中K个最接近的元素**



		vector<int> findClosestElements(vector<int>& arr, int k, int x)
		    int lo = 0, hi = arr.size() - k;
		    while (lo < hi) {
			int mid = lo + (hi - lo >> 1);
			if (x - arr[mid] > arr[mid + k] - x ) 
			    lo = mid + 1;
			else 
			    hi = mid;
		return vector<int>(arr.begin() + lo, arr.begin() + lo + k);

