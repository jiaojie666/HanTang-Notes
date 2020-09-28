<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>


-	**乘积小于K的子数组**



		int numSubarrayProductLessThanK(vector<int>& nums, int k)
		    if(k<=1)
			return 0;
		    int left=0,right=0,sum=1;
		    int ans=0;
		    while(right<nums.size())
			sum*=nums[right];
			while(sum>=k)
			    sum/=nums[left++];
			ans+=(right-left+1);
			right++;
		return ans;
-	**找出数组对之间第k小距离**



		int smallestDistancePair(vector<int>& nums, int k)
		    int n=nums.size();
		    sort(nums.begin(),nums.end());
		    int l=0;
		    int r=nums[n-1]-nums[0];
		    while(l<r)
			int mid=(l+r)/2;
			if(compute(nums,mid)>=k)
			    r=mid;
			else
			    l=mid+1;
		    return l;
		int compute(vector<int>&nums,int val)
		    int left=0;
		    int right=1;
		    int res=0;
		    for(right=0;right<nums.size();right++)
			while(nums[right]-nums[left]>val)
			    left++;
			res+=(right-left);
		return res;
-	**数组中哪些元素是给定字符串的子序列**



		int numMatchingSubseq(string S, vector<string>& words) {
		    vector<queue<pair<int,int>>> buckets(26);
		    for(int i=0;i<words.size();i++)
			buckets[words[i][0]-'a'].push({i,0});//word index and pos index
		    int res=0;
		    for(char c:S)
			queue<pair<int, int>> & q = buckets[c-'a'];
			for(int i = q.size(); i > 0; i--){
			    auto pair=q.front();
			    q.pop();
			    pair.second++;
			    if(pair.second==words[pair.first].size())
				res++;
			    else
				buckets[words[pair.first][pair.second]-'a'].push(pair);
		return res;

-	**消失的数字两位**



		求和
		vector<int> missingTwo(vector<int>& nums) 
		    int n = nums.size() + 2;
		    long sum = 0;
		    for (auto x: nums) sum += x;
		    int sumTwo = n * (n + 1) / 2 - sum, limits = sumTwo / 2;
		    sum = 0;
		    for (auto x: nums)
			if (x <= limits) sum += x; 
		    int one = limits * (limits + 1) / 2 - sum;
		    return {one, sumTwo - one};


		位对齐
		vector<int> missingTwo(vector<int>& nums)
		    nums.push_back(-1);
		    nums.push_back(-1);
		    nums.push_back(-1);
		    for(int i=0;i<nums.size();i++)
			while(nums[i]!=i&&nums[i]!=-1)
			    swap(nums[i],nums[nums[i]]);
		    vector<int> res;
		    for(int i=1;i<nums.size();i++)
			if(nums[i]==-1)
			    res.push_back(i);
		return res;
-	**缺失的数字一位**



		int missingNumber(vector<int>& nums)
		    int l=0;
		    int r=nums.size()-1;
		    while(l<=r)
			int mid=l+(r-l)/2;
			if(nums[mid]==mid)
			    l=mid+1;
			else
			    r=mid-1;
		return l;
-	**缺失的第一个正数**



		int firstMissingPositive(vector<int>& nums) {
		    int n=nums.size();
		    for(int i=0;i<nums.size();i++)
		        if(nums[i]<=0){
		            nums[i]=n+1;
		    for(int i=0;i<nums.size();i++)
		        int num=abs(nums[i]);
		        if(num<=n)
		            nums[num-1]=-abs(nums[num-1]);
		    for(int i=0;i<n;i++)
		        if(nums[i]>0){
		            return i+1;
		return n+1;
-	**寻找第K个数**



		int findKthLargest(vector<int>& nums, int k) 
		    srand(time(0));
		    return quickSelect(nums,0,nums.size()-1,k-1);
		int quickSelect(vector<int>&nums,int l,int r, int k){
		    int pos=partition(nums,l,r);
		    if(pos==k)
			return nums[pos];
		    else
			return pos<k?quickSelect(nums,pos+1,r,k):quickSelect(nums,l,pos-1,k);
		int partition(vector<int>&nums,int left,int right)
		    int pos=rand() % (right - left + 1) + left;
		    swap(nums[left],nums[pos]);
		    int val=nums[left];
		    while(left<right)        while(left<right&&nums[right]<=val)
			    --right;
			nums[left]=nums[right];
			while(left<right&&nums[left]>=val)
			    ++left;
			nums[right]=nums[left];
		    nums[right]=val;
		return right;
