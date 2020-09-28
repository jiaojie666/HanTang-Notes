<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

-	**栈的压入弹出序列**


	
		int i=0;int j=0;int len=pushV.size();
		while(i<len){
		    s.push(pushV[i++]);
		    while(j<len&&s.top()==popV[j]){
			s.pop();
			j++;
-	**括号匹配问题**



		stack<char> st;
		bool isValid(string s) {
		    for(char c:s){
			if(typeCheck(c))
			    st.push(c);
			else
			    if(!st.empty()&&leftOf(c)==st.top())
				st.pop();
			    else return false;
		    return st.empty();
		bool typeCheck(char c){
		    if(c=='('||c=='{'||c=='['){
			return true;
		    return false;
		char leftOf(char c)
		    if(c=='}')    return '{';
		    else if(c==']') return '[';
		    else return '(';
-	**接雨水问题**



		int trap(vector<int>& height)
		    if(height.empty()) return 0;
		    int left=0, right=height.size()-1;
		    int l_max=height[left];
		    int r_max=height[right];
		    while(left<=right)
			l_max=max(l_max,height[left]);
			r_max=max(r_max,height[right]);
			if(l_max<r_max)
			    res+=(l_max-height[left]);
			    left++;
			else
			    res+=(r_max-height[right]);
			    right--;
-	**柱状图中的最大矩形**



		int largestRectangleArea(vector<int>& heights)
		    if(heights.empty())
			return 0;
		    vector<int> left(heights.size(),0);
		    vector<int> right(heights.size(),0);
		    left[0]=-1;
		    right[heights.size()-1]=heights.size();
		    for(int i=1;i<heights.size();i++){
			int p=i-1;
			while(p>=0&&heights[p]>=heights[i])
			    p=left[p];
			left[i]=p;
		    for(int i=heights.size()-2;i>=0;i--)
			int p=i+1;
			while(p<heights.size()&&heights[p]>=heights[i])
			    p=right[p];
			right[i]=p;
		    int area=0;
		    for(int i=0;i<heights.size();i++){
			area=max(area,heights[i]*(right[i]-left[i]-1));
-	**最小栈**



		class MinStack {
		public:
		    void push(int x)
			normal.push(x);
			if(min.empty())
			    min.push(x);
			    return;
			int m=min.top()<x?min.top():x;
			min.push(m);
		    void pop()
			min.pop();
			normal.pop();
		    int top() 
			return normal.top();
		    int getMin() 
			return min.top();
		private:
		    stack<int> normal;
			stack<int> min;
-	**单调栈：下一个更大元素**



		vector<int> nextGreaterElements(vector<int>& nums)
		    int n=nums.size();
		    vector<int> ans(n);
		    stack<int> s;
		    for(int i=2*n-1;i>=0;i--)
			while(!s.empty()&&s.top()>=nums[i%n])
			    s.pop();
			ans[i%n]=s.empty()?-1:s.top();
			s.push(nums[i%n]);
		return ans;
