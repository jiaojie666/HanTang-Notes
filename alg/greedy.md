<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

-	**跳跃游戏**



		bool canJump(vector<int>& nums) 
		    int n = nums.size();
		    int rightmost = 0;
		    for (int i = 0; i < n; ++i) 
			if (i <= rightmost) 
			    rightmost = max(rightmost, i + nums[i]);
			    if (rightmost >= n - 1) 
				return true;
		return false;
-	**跳跃游戏恰好到最后一步**



		int dp(vector<int>&nums,int p)/ 
		    int n=nums.size();
		    if(p>=n-1)
			return 0;
		    if(memo[p]!=n)
			return memo[p];
		    int steps=nums[p];
		    for(int i=1;i<=steps;i++)
			int subProblem=dp(nums,p+i);
			memo[p]=min(memo[p],subProblem+1);
		    return memo[p];
		int jump(vector<int>& nums) 
		    int n=nums.size();
