<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

- **组合问题：每个元素可以重复选取，但应该整体无重**



		void DFS(int start, int target)
		    if (target == 0) {
			res.push_back(path); return;
		    for (int i = start; i < candidates.size() && target - candidates[i] >= 0; i++) 
			if (i > start && candidates[i] == candidates[i - 1])
			    continue;
			path.push_back(candidates[i]);
			DFS(i, target - candidates[i]);
			path.pop_back();
		vector<vector<int>> combinationSum(vector<int> &candidates, int target)
		    sort(candidates.begin(), candidates.end());
		    this->candidates = candidates;
		    DFS(0, target);
		return res;
-	**组合问题只能使用一次**



		void DFS(int start, int target)
		    if (target == 0) 
			res.push_back(path);return;
		    for (int i = start; i < candidates.size() && target - candidates[i] >= 0; i++)
			if (i > start && candidates[i] == candidates[i - 1])
			    continue;
			path.push_back(candidates[i]);
			DFS(i + 1, target - candidates[i]);
			path.pop_back();
		vector<vector<int>> combinationSum2(vector<int> &candidates, int target)
		    sort(candidates.begin(), candidates.end());
		    this->candidates = candidates;
		DFS(0, target); return res;
	-	**全排列：无重复数字**



			vector<vector<int>> permute(vector<int>& nums)
			    if(nums.empty())
				return {};
			    permute(nums,0);return res;
			void permute(vector<int>&nums,int start)
			    if(start>=nums.size())
				res.push_back(nums);
				return;
			    for(int i=start;i<nums.size();i++)
				swap(nums[start],nums[i]);
				permute(nums,start+1);
				swap(nums[start],nums[i]);
-	**全排列：有重复**



			相比上面swap之前要加一个judge
			bool judge(vector<int>&nums,int start,int end)
			    for(int i=start;i<end;i++)
				if(nums[i]==nums[end])
				    return true;
			return false;




-	**子集问题：无重复**


		vector<vector<int>> subsets(vector<int>& nums) {
		    vector<int> temp={};
		    helper(nums,0,temp);return res;
		void helper(vector<int>&nums,int start,vector<int>&temp)
		    res.push_back(temp);
		    for(int i=start;i<nums.size();i++)
			temp.push_back(nums[i]);
			helper(nums,i+1,temp);
			temp.pop_back();
-	**子集问题有重复**



		vector<vector<int>> subsetsWithDup(vector<int>& nums)
		    vector<int> temp={};
		    sort(nums.begin(),nums.end());
		    dfs(temp,0,nums);return res;
		void dfs(vector<int>&temp,int cur,vector<int>&nums)
		    res.push_back(temp);
		    if(cur==nums.size())
			return;
		    for(int i=cur;i<nums.size();i++)
			if(i>cur&&nums[i]==nums[i-1])
			    continue;
			temp.push_back(nums[i]);
			dfs(temp,i+1,nums);
			temp.pop_back();
-	**N皇后问题**



		vector<vector<string>> solveNQueens(int n)
		    vector<string> data;
		    string line;
		    for(int i=0;i<n;i++)
			line.push_back('.');
		    for(int i=0;i<n;i++)
			data.push_back(line);
		    solveNQueens(data,0,n);return res;
		void solveNQueens(vector<string>&data,int cur,int n)
		    if(cur>=n){
			res.push_back(data);
			return;
		    for(int i=0;i<n;i++)
			if(isValid(data,cur,i)){
			    data[cur][i]='Q';
			    solveNQueens(data,cur+1,n);
			    data[cur][i]='.';
-	**分割回文串**



		vector<vector<string>> partition(string s)
		    if(s.empty())
			return {};
		    vector<string> temp;
		    compute(s,0,temp);
		    return res;
		void compute(string&s,int start,vector<string>&temp)
		    if(start>=s.size())
			res.push_back(temp);
			return;
		    for(int i=start;i<s.size();i++){
			if(valid(s,start,i)){
			    temp.push_back(s.substr(start,i-start+1));
			    compute(s,i+1,temp);
			    temp.pop_back();
-	**字母大小写全排列**



		vector<string> letterCasePermutation(string S) {
		    if(S.empty())
			return {};
		    dfs(S,0);return res;
		void dfs(string&s, int cur)
		    if(cur>=s.size())
			res.push_back(s);
			return;
		    dfs(s,cur+1);
		    if(isalpha(s[cur]))
			s[cur]^=32;
			dfs(s,cur+1);
