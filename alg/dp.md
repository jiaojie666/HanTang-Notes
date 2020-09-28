<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

-	<b>最长回文子串：动态规划算法</b>

		string longestPalindrome(string s) 
		    int n = s.size();
		    vector<vector<int>> dp(n, vector<int>(n));string ans;
		    for (int l = 0; l < n; ++l)
			for (int i = 0; i + l < n; ++i)
			    int j = i + l;
			    if (l == 0) 
				dp[i][j] = 1;
			    else if (l == 1) 
				dp[i][j] = (s[i] == s[j]);
			    else 
				dp[i][j] = (s[i] == s[j] && dp[i + 1][j - 1]);
			    if (dp[i][j] && l + 1 > ans.size()) {
				ans = s.substr(i, l + 1);




-	**最长回文子串:中心扩展法**


		string longestPalindrome(string s)
		    string m="";
		    for(int i=0;i<s.size();i++)
			string s1=palindrome(s,i,i);
			string s2=palindrome(s,i,i+1);
			string max=s1.size()>s2.size()?s1:s2;
			m=max.size()>m.size()?max:m;
		    return m;
		string palindrome(string s,int left,int right)
		    while(left>=0&&right<s.size()&&s[left]==s[right])
			left--;right++;
		    return s.substr(left+1,right-left-1);
-	**正则表达式：递归**



		bool match(char* str, char* pattern)
		if(!pattern||!*pattern)
		    return (!str||!*str);
		bool first_match=false;
		if(str&&*str)
		    first_match=str[0]==pattern[0]||pattern[0]=='.';
		if(strlen(pattern)>=2&&pattern[1]=='*')
		    returnfirst_match&&match(str+1, pattern)||match(str,pattern+2);
		else
		    return first_match&&match(str+1,pattern+1);
-	**正则表达式动态规划**



		dp[0][0]=true;//means [0,i], [0,j] match case
		for(int i=1;i<=s.size();i++)
		    dp[i][0]=false;
		for(int j=1;j<=p.size();j++)
		    dp[0][j]=p[j-1]=='*'?dp[0][j-2]:false;
		for(int i=1;i<=s.size();i++){
		    for(int j=1;j<=p.size();j++)
		    if(p[j-1]=='*'&&j>=2){
			if(p[j-2]==s[i-1]||p[j-2]=='.')
			    dp[i][j]=dp[i-1][j]||dp[i][j-2];
			else
			    dp[i][j]=dp[i][j-2];
		    else
			if(p[j-1]==s[i-1]||p[j-1]=='.')
			    dp[i][j]=dp[i-1][j-1];
        
-	**通配符匹配 **



		bool isMatch(string s, string p) 
		    int m = s.size();
		    int n = p.size();
		    vector<vector<int>> dp(m + 1, vector<int>(n + 1));
		    dp[0][0] = true;
		    for (int i = 1; i <= n; ++i) 
			if (p[i - 1] == '*')
			    dp[0][i] = true;
			else break;
		    for (int i = 1; i <= m; ++i) 
			for (int j = 1; j <= n; ++j) 
			    if (p[j - 1] == '*') 
				dp[i][j] = dp[i][j - 1] | dp[i - 1][j];
			    else if (p[j - 1] == '?' || s[i - 1] == p[j - 1]) 
				dp[i][j] = dp[i - 1][j - 1];
-	**最大子序和**


		int maxSubArray(vector<int>& nums) 
		    int pre = 0, maxAns = nums[0];
		    for (const auto &x: nums) 
			pre = max(pre + x, x);
			maxAns = max(maxAns, pre);
		return maxAns;
-	**爬楼梯**


		int climbStairs(int n) 
		    int p = 0, q = 0, r = 1;
		    for (int i = 1; i <= n; ++i) 
			p = q; q = r; 
			r = p + q;
		return r;
		int climbStairs(int n) 
		    double sqrt5 = sqrt(5);
		    double fibn = pow((1 + sqrt5) / 2, n + 1) 
		    - Math.pow((1 - sqrt5) / 2, n + 1);
		return (int)(fibn / sqrt5);
-	**编码方法**


		int pre = 1, curr = 1;//dp[-1] = dp[0] = 1
		for (int i = 1; i < s.size(); i++) {
		    int tmp = curr;
		    if (s[i] == '0')
		        if (s[i - 1] == '1' || s[i - 1] == '2') curr = pre;
		        else return 0;
		    else if (s[i - 1] == '1' || (s[i - 1] == '2' && s[i] >= '1' && s[i] <= '6'))
		        curr = curr + pre;
		    pre = tmp;
-	**1-n能组成的所有二叉树打印**



		vector<TreeNode*> generateTrees(int left,int right)
		    if(left>right)
			return {nullptr};
		    vector<TreeNode*> allTrees;
		    for(int i=left;i<=right;i++)
			vector<TreeNode*> leftTrees=generateTrees(left,i-1);
			vector<TreeNode*> rightTrees=generateTrees(i+1,right);
			for(auto left:leftTrees)
			    for(auto right:rightTrees)
				TreeNode *node=new TreeNode(i);
				node->left=left;
				node->right=right;
				allTrees.push_back(node);
		return allTrees;

-	**买卖股票最佳时机**



		base case：
		dp[-1][k][0] = dp[i][0][0] = 0
		dp[-1][k][1] = dp[i][0][1] = -infinity
		状态转移方程：
		dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])
		dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
		k=1:
			dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
		dp[i][1] = max(dp[i-1][1], -prices[i])
		k=int_max
			dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
		dp[i][1] = max(dp[i-1][1], dp[i-1][0] - prices[i])
		k=int_max && cooldown
			dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
		dp[i][1] = max(dp[i-1][1], dp[i-2][0] - prices[i])
		k=int_max && fee
			dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
		dp[i][1] = max(dp[i-1][1], dp[i-1][0] - prices[i] - fee)

-	**单词拆分问题**



		for(int i=1;i<=s.size();i++)
		    for(int j=i-1;j>=0;j--)
			if(dp[j]&&keys.count(s.substr(j,i-j))>0)
			    dp[i]=true; beak;

-	**打印所有单词可能的拆分序列**



		vector<string> wordBreak(string&s,int start)
		    if(memo.count(start)>0)
			return memo[start];
		    vector<string> ans;
		    if(start>=s.size())
			ans.push_back("");
		    for(int end=start+1;end<=s.size();end++){
			if(words.count(s.substr(start,end-start))>0)
			    vector<string> temp=wordBreak(s,end);
			    for(string &t:temp){
				    ans.push_back(s.substr(start,end-start)+(t==""?"":" ")+t);
		    memo[start]=ans;
		return ans;

-	**乘积最大子数组**



		if(nums.empty())
		    return 0;
		int res=nums[0],minV=nums[0],maxV=nums[0];
		for(int i=1;i<nums.size();i++)
		    int mx=maxV,mn=minV;
		    minV=min(mn*nums[i],min(nums[i],mx*nums[i]));
		    maxV=max(mx*nums[i],max(nums[i],mn*nums[i]));
		    res=max(res,maxV);
		return res;
-	**打家劫舍问题**



		int rob(int[] nums) {
		    int n = nums.length;
		    int dp_i_1 = 0, dp_i_2 = 0;
		    int dp_i = 0; 
		    for (int i = n - 1; i >= 0; i--)
			dp_i = Math.max(dp_i_1, nums[i] + dp_i_2);
			dp_i_2 = dp_i_1;
			dp_i_1 = dp_i;
		return dp_i;

		首尾相连的情况：
			max(robRange(nums, 0, n - 2), 
				    robRange(nums, 1, n - 1));
			int robRange(int[] nums, int start, int end) 
		    int n = nums.length;
		    int dp_i_1 = 0, dp_i_2 = 0;
		    int dp_i = 0;
		    for (int i = end; i >= start; i--) 
			dp_i = Math.max(dp_i_1, nums[i] + dp_i_2);
			dp_i_2 = dp_i_1;
			dp_i_1 = dp_i;
-	**二维矩阵中最大正方形**



		int maximalSquare(vector<vector<char>>& matrix) 
		    if (matrix.size() == 0 || matrix[0].size() == 0) 
			return 0;
		    int maxSide = 0;
		    int rows = matrix.size(), columns = matrix[0].size();
		    vector<vector<int>> dp(rows, vector<int>(columns));
		    for (int i = 0; i < rows; i++) {
			for (int j = 0; j < columns; j++) 
			    if (matrix[i][j] == '1') 
				if (i == 0 || j == 0) 
				    dp[i][j] = 1;
				else 
				    dp[i][j] = min(min(dp[i - 1][j], dp[i][j - 1]), dp[i - 1][j - 1]) + 1;
				maxSide = max(maxSide, dp[i][j]);
		    int maxSquare = maxSide * maxSide;
-	**完全平方数**



		int numSquares(int n) {
		    for(int i=1;i<=sqrt(n);i++)
			nums.push_back(i*i);
		    vector<int> dp(n+1,INT_MAX);
		    dp[0]=0;
		    for(int i=1;i<=n;i++)
			for(int k:nums)
			    if(i<k)
				break;
			    dp[i]=min(dp[i],dp[i-k]+1);
		return dp[n];
-	**最长上升子序列**



		if(nums.empty())
		    return 0;
		int res=0; vector<int> dp(nums.size()+1,1);
		for(int i=0;i<nums.size();i++)
		    int val=1;
		    for(int j=i-1;j>=0;j--)
			if(nums[i]>nums[j])
			    val=max(val,dp[j]+1);
		    dp[i]=val;
		    res=max(res,dp[i]);
		return res;
-	**零钱兑换问题**



		币种无穷求最小
		int coinChange(vector<int>& coins, int amount) 
		    vector<int> dp(amount+1,amount+1);
		    dp[0]=0;
		    for(int i=0;i<=amount;i++) 
			for(int& coin:coins){
			    if(i<coin){
				continue;
			    dp[i]=min(dp[i],dp[i-coin]+1);
		return dp[amount]==amount+1?-1:dp[amount];

		凑价值n的币种问题
		dp[i][j]代表前i种钱凑出j价值的方案数
		dp[i][0]=1 dp[0][i]=1
		for(int i=1;i<=len1;i++)
		  for(int j=1;j<=n+1;j++)
			if(j>coins[i])
			  dp[i][j]=dp[i-1][j]+dp[i-1][j-coins[i]]
			else dp[i][j]=dp[i-1][j]
-	**整数拆分问题**



		vector<int> dp(n+1,0);
		for(int i=2;i<=n;i++)
		    int cur=0;
		    for(int j=i-1;j>=0;j--)
			cur=max(cur,max(j*(i-j),j*dp[i-j]));
		    dp[i]=cur;
		return dp[n];
-	**划分为k个子集**



		bool canPartitionKSubsets(vector<int>& nums, int k) {
			int sum=accumulate(nums.begin(),nums.end(),0);
			if(sum%k!=0)
			    return false;
			int target=sum/k;
			vector<bool> visited(nums.size(),false);
			sort(nums.begin(),nums.end(),greater<int>());
			return helper(0,0,k,nums,visited,target);
		bool helper(int start, int sum, int group, vector<int>&nums, vector<bool>&visited, int target)
		    if(group==1)
			return true;
		    bool flag=false;
		    for(int i=start;i<nums.size();i++)
			if((!visited[i])&&(sum+nums[i]<=target))
			    visited[i]=true;
			    if(sum+nums[i]==target)
				flag=helper(0,0,group-1,nums,visited,target);
			    else
				flag=helper(i+1,sum+nums[i],group,nums,visited,target);
			    if(flag)
				return true;
			    visited[i]=false;
		return flag;
-	**鸡蛋掉落问题**



		int superEggDrop(int K, int N)
		    vector<vector<int>> dp(N+1,vector<int>(K+1,0));
		    int m=0;
		    while(dp[m][K]<N)
			m++;
			for(int k=1;k<=K;k++)
			    dp[m][k]=dp[m-1][k-1]+dp[m-1][k]+1;
		return m;
-	**连续子数组模K**



		bool checkSubarraySum(int[] nums, int k) {
		    int sum = 0;
		    HashMap < Integer, Integer > map = new HashMap < > ();
		    map.put(0, -1);
		    for (int i = 0; i < nums.length; i++)
		        sum += nums[i];
		        if (k != 0)
		            sum = sum % k;
		        if (map.containsKey(sum))
		            if (i - map.get(sum) > 1)
		                return true;
		        else
		            map.put(sum, i);
-	**二叉搜索树数目**


		
		int numTrees(int n) 
		    vector<int> G(n + 1, 0);
		    G[0] = 1;
		    G[1] = 1;
		    for (int i = 2; i <= n; ++i)
		        for (int j = 1; j <= i; ++j) 
		            G[i] += G[j - 1] * G[i - j];
		return G[n];
-	**最长公共子数组**



		int findLength(vector<int>& A, vector<int>& B) {
		    int n = A.size(), m = B.size();
		    vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
		    int ans = 0;
		    for (int i = n - 1; i >= 0; i--) 
			for (int j = m - 1; j >= 0; j--) 
			    dp[i][j] = A[i] == B[j] ? dp[i + 1][j + 1] + 1 : 0;
			    ans = max(ans, dp[i][j]);
		return ans;
-	**最长递增序列数量**



		int findNumberOfLIS(vector<int>& nums) {
		    if (nums.empty()) return 0;
		    int N = nums.size();
		    // pair<int, int> 分别为最长递增长度与对应的数目
		    vector<pair<int, int> > dp(N, {1, 1});
		    int mx = 1;
		    for (int i = 1; i < N; ++i) 
			for (int j = 0; j < i; ++j) 
			    if (nums[i] > nums[j]) {
				if (dp[i].first < dp[j].first + 1) 
				    dp[i] = {dp[j].first + 1, dp[j].second};
				else if (dp[i].first == dp[j].first + 1) 
				    dp[i].second += dp[j].second;
			mx = max(mx, dp[i].first);
		    int res = 0;
		    for (int i = 0; i < N; ++i) 
			if (dp[i].first == mx) {
			    res += dp[i].second;
		return res;
	-**青蛙跳台阶 1~n**



		return pow(2,number-1)
-	**背包问题**



		0-1背包问题：初始化为0
		dp[i][j] = max(dp[i−1][j], dp[i−1][j−w[i]]+v[i]) 逆向枚举
		优化：i in [1,n] j[W,w[i]] dp[j]=max{dp[j],dp[j-w[i]]+v[i]}
		完全背包问题：初始化为0
		dp[i][j] = max(dp[i−1][j], dp[i][j−w[i]]+v[i]) 优化，正方向枚举i in [1,n] j[w[i],W] dp[j]=max{dp[j],dp[j-w[i]]+v[i]}
		最多k个
		k <= min(n[i], j/w[i])
		dp[i][j] = max{(dp[i-1][j − k*w[i]] + k*v[i]) for every k}
		恰好装满
		定义dp[i][j]代表前i件物品能否装满j容量，若优化则倒序 dp[j]=dp[j]||dp[j-nums[i]]
		正常写法
		vector<vector<bool>> dp(n + 1, vector<bool>(sum + 1, false));
		for (int i = 0; i <= n; i++) dp[i][0] = true;
		for (int i = 1; i <= n; i++) {
		    for (int j = 1; j <= sum; j++) {
			if (j - nums[i - 1] < 0) {
			    dp[i][j] = dp[i - 1][j]; 
			else 
			    dp[i][j] = dp[i - 1][j] || dp[i - 1][j-nums[i-1]];
		return dp[n][sum];
